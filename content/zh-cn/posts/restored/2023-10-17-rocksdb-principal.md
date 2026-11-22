---
title: rocksdb原理
date: '2023-10-17T13:01:23+08:00'
lastmod: '2023-10-17T13:27:30+08:00'
draft: false
tags:
- infra
- Middleware
- rocksdb
categories:
- code
url: /zh-cn/2023/10/17/category/rocksdb-principal/
authors:
- yehh
---

#### 基本概念 {#基本概念}

- 版本(Version)：这个是RocksDB内部概念。一个版本包含某个时间点的所有存活SST文件。一旦一个落盘或者压缩完成，由于存活SST文件发生了变化，一个新的“版本”会被创建。一个旧的“版本”还会被仍在进行的读请求或者压缩工作使用。旧的版本最终会被回收。

- 超级版本(super version)：RocksDB的内部概念。一个超级版本包含一个特定时间的 的 一个SST文件列表（一个“版本”）以及一个存活memtable的列表。不管是压缩还是落盘，抑或是一个memtable切换，都会生成一个新的“超级版本”。一个旧的“超级版本”会被继续用于正在进行的读请求。旧的超级版本最终会在不再需要的时候被回收掉。

- 对two_write_queues的解释 When we have two write queues (two_write_queues=true) then the primary write queue can write to both WAL and memtable and the 2nd one can write only to the WAL, which will be used for writing the commit marker in WritePrepared transactions. In this case the primary queue (and its PreReleaseCallback callback) is always used for prepare entires and the 2nd queue (and its PreReleaseCallback callback) is always used only for commits. This i) avoids race condition between the two queues, ii) maintains the in-order addition to PreparedHeap, and iii) simplifies the code by avoiding concurrent insertion to CommitCache (and the code that is called upon each eviction from it).

#### transaction {#transaction}

暂时没有总结什么，看[2PC](https://whoiami.github.io/ROCKSDB_TRANSACTION_2PC)和[transaction](https://whoiami.github.io/ROCKSDB_TRANSACTION)就差不多了。

#### pipeline {#pipeline}

pipeline write

【默认情况】

单一的write thread队列，队首writer成为leader，并负责写WAL以及memtable

【pipeline write】

只有一个writer的情况下，要先写WAL，再写memtbale

如果有多个writer，默认情况就需要先写完WAL，在写memtable

启用pipeline之后，前一个writer写完WAL就可以写memtbale，而后一个writer开始写他的WAL

开启方式：Options.enable_pipeline_write=true

提升：20%性能提升

#### Merge {#merge}

Merge 本质上就是一个 read-modify-write的原子操作。

code is all you need

```c++
    // A 'model' merge operator with uint64 addition semantics
    class UInt64AddOperator : public AssociativeMergeOperator {
     public:
      virtual bool Merge(
        const Slice& key,
        const Slice* existing_value,
        const Slice& value,
        std::string* new_value,
        Logger* logger) const override {

        // assuming 0 if no existing value
        uint64_t existing = 0;
        if (existing_value) {
          if (!Deserialize(*existing_value, &existing)) {
            // if existing_value is corrupted, treat it as 0
            Log(logger, "existing value corruption");
            existing = 0;
          }
        }

        uint64_t oper;
        if (!Deserialize(value, &oper)) {
          // if operand is corrupted, treat it as 0
          Log(logger, "operand value corruption");
          oper = 0;
        }

        auto new = existing + oper;
        *new_value = Serialize(new);
        return true;        // always return true for this, since we treat all errors as "zero".
      }

      virtual const char* Name() const override {
        return "UInt64AddOperator";
       }
    };

    // Implement 'add' directly with the new Merge operation
    class MergeBasedCounters : public RocksCounters {
     public:
      MergeBasedCounters(std::shared_ptr<DB> db);

      // mapped to a leveldb Merge operation
      virtual void Add(const string& key, uint64_t value) override {
        string serialized = Serialize(value);
        db_->Merge(merge_option_, key, serialized);
      }
    };

    // How to use it
    DB* dbp;
    Options options;
    options.merge_operator.reset(new UInt64AddOperator);
    DB::Open(options, "/tmp/db", &dbp);
    std::shared_ptr<DB> db(dbp);
    MergeBasedCounters counters(db);
    counters.Add("a", 1);
    ...
    uint64_t v;
    counters.Get("a", &v);
```

AssociativeMergeOperator适用于比较简单的情景，也就是说，如果你的merge操作是满足结合律的，那么就可以使用AssociativeMergeOperator。

但是如果是复杂一点的，比如说data是json字符串，但是merge的时候只是对其中的一个子结构进行修改，那么就需要使用MergeOperator。

值得注意的是，put和merge并不会在invoke的时候就立刻执行，而是会stack，在get的时候才执行，而在stack的时候就是PartialMerge的时候了。

接口：

```c++
    // The Merge Operator
    //
    // Essentially, a MergeOperator specifies the SEMANTICS of a merge, which only
    // client knows. It could be numeric addition, list append, string
    // concatenation, edit data structure, ... , anything.
    // The library, on the other hand, is concerned with the exercise of this
    // interface, at the right time (during get, iteration, compaction...)
    class MergeOperator {
     public:
      virtual ~MergeOperator() {}

      // Gives the client a way to express the read -> modify -> write semantics
      // key:         (IN) The key that's associated with this merge operation.
      // existing:    (IN) null indicates that the key does not exist before this op
      // operand_list:(IN) the sequence of merge operations to apply, front() first.
      // new_value:  (OUT) Client is responsible for filling the merge result here
      // logger:      (IN) Client could use this to log errors during merge.
      //
      // Return true on success. Return false failure / error / corruption.
      virtual bool FullMerge(const Slice& key,
                             const Slice* existing_value,
                             const std::deque<std::string>& operand_list,
                             std::string* new_value,
                             Logger* logger) const = 0;

      struct MergeOperationInput { ... };
      struct MergeOperationOutput { ... };
      virtual bool FullMergeV2(const MergeOperationInput& merge_in,
                               MergeOperationOutput* merge_out) const;

      // This function performs merge(left_op, right_op)
      // when both the operands are themselves merge operation types.
      // Save the result in *new_value and return true. If it is impossible
      // or infeasible to combine the two operations, return false instead.
      virtual bool PartialMerge(const Slice& key,
                                const Slice& left_operand,
                                const Slice& right_operand,
                                std::string* new_value,
                                Logger* logger) const = 0;

      // The name of the MergeOperator. Used to check for MergeOperator
      // mismatches (i.e., a DB created with one MergeOperator is
      // accessed using a different MergeOperator)
      virtual const char* Name() const = 0;

      // Determines whether the MergeOperator can be called with just a single
      // merge operand.
      // Override and return true for allowing a single operand. FullMergeV2 and
      // PartialMerge/PartialMergeMulti should be implemented accordingly to handle
      // a single operand.
      virtual bool AllowSingleOperand() const { return false; }
    };
```

示例：

```c++

    // A 'model' pseudo-code merge operator with json update semantics
    // We pretend we have some in-memory data-structure (called JsonDataStructure) for
    // parsing and serializing json strings.
    class JsonMergeOperator : public MergeOperator {          // not associative
     public:
      virtual bool FullMerge(const Slice& key,
                             const Slice* existing_value,
                             const std::deque<std::string>& operand_list,
                             std::string* new_value,
                             Logger* logger) const override {
        JsonDataStructure obj;
        if (existing_value) {
          obj.ParseFrom(existing_value->ToString());
        }

        if (obj.IsInvalid()) {
          Log(logger, "Invalid json string after parsing: %s", existing_value->ToString().c_str());
          return false;
        }

        for (const auto& value : operand_list) {
          auto split_vector = Split(value, " = ");      // "xyz[0] = 5" might return ["xyz[0]", 5] as an std::vector, etc.
          obj.SelectFromHierarchy(split_vector[0]) = split_vector[1];
          if (obj.IsInvalid()) {
            Log(logger, "Invalid json after parsing operand: %s", value.c_str());
            return false;
          }
        }

        obj.SerializeTo(new_value);
        return true;
      }


      // Partial-merge two operands if and only if the two operands
      // both update the same value. If so, take the "later" operand.
      virtual bool PartialMerge(const Slice& key,
                                const Slice& left_operand,
                                const Slice& right_operand,
                                std::string* new_value,
                                Logger* logger) const override {
        auto split_vector1 = Split(left_operand, " = ");   // "xyz[0] = 5" might return ["xyz[0]", 5] as an std::vector, etc.
        auto split_vector2 = Split(right_operand, " = ");

        // If the two operations update the same value, just take the later one.
        if (split_vector1[0] == split_vector2[0]) {
          new_value->assign(right_operand.data(), right_operand.size());
          return true;
        } else {
          return false;
        }
      }

      virtual const char* Name() const override {
        return "JsonMergeOperator";
       }
    };

    ...

    // How to use it
    DB* dbp;
    Options options;
    options.merge_operator.reset(new JsonMergeOperator);
    DB::Open(options, "/tmp/db", &dbp);
    std::shared_ptr<DB> db_(dbp);
    ...
    // Put/store the json string into to the database
    db_->Put(put_option_, "json_obj_key",
             "{ employees: [ {first_name: john, last_name: doe}, {first_name: adam, last_name: smith}] }");

    ...

    // Use the "merge operator" to incrementally update the value of the json string
    db_->Merge(merge_option_, "json_obj_key", "employees[1].first_name = lucy");
    db_->Merge(merge_option_, "json_obj_key", "employees[0].last_name = dow");
```

#### Compaction Filter {#compaction-filter}

Compaction Filter是用来在compaction的时候执行一些自定义操作比如：根据ttl删除值，删除特定范围的键值对等等。

接口可以在rocksdb/compaction_filter.h文件里面找到.

有两种方式可以使用compaction filter:

```c++
options.compaction_filter = new CustomCompactionFilter();
// or
options.compaction_filter_factory.reset(new CustomCompactionFilterFactory());
```

这两种方式对并发有不同的要求：

1.  compaction_filter: 一个compaction filter实例会被多个线程共享，所以需要保证线程安全。
2.  compaction_filter_factory: 每个线程会有一个compaction filter实例，所以不需要保证线程安全。

> Compaction filter will not be invoked during flush, despite arguably flush is a special type of compaction. When merge is being used, compaction filter is invoked per merge operand. The result of compaction filter is applied to the merge operand before merge operator is invoked.

有两套API来实现compaction filter:

1.  Filter/FilterMergeOperand 作用有限
2.  FilterV2 changing the value, or dropping a range of keys starting from the current key.

#### Reference {#reference}

[简单介绍](https://segmentfault.com/a/1190000041198407) [rocksdb文档的翻译](https://wanghenshui.github.io/rocksdb-doc-cn/)
