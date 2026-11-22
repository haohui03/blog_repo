---
title: protobuf-intro
date: '2023-11-19T09:01:23+08:00'
lastmod: '2023-11-19T09:20:30+08:00'
draft: false
tags:
- golang
- protobuf
categories:
- code
url: /zh-cn/2023/11/19/category/protobuf-intro/
authors:
- yehh
---

# 本篇文章记录学习protobuf的过程 {#本篇文章记录学习protobuf的过程}

## 定义消息 {#定义消息}

```proto
syntax = "proto3";

message name {
    body
}
```

```proto
syntax = "proto3";
 
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

> 通常把message定义在一个单独的文件中，文件名以.proto结尾。

- 第一行指定版本，不指定默认是proto2.
- message是消息的关键字，后面跟着消息的名称，名称必须是唯一的。

### 字段类型，字段标识和字段标签（Field Type & Field Numbers & Field labels） {#字段类型字段标识和字段标签field-type--field-numbers--field-labels}

#### 字段类型Field Type {#字段类型field-type}

##### 数字类型 {#数字类型}

int64,int32,uint32,uint64,sint32,sin64 都是可变长的,然后uint表示无符号,sint在表示负数的时候比较高效

fixed32,fixed64,sfix32,sfix64 都是固定长度的

##### 其他常见类型 {#其他常见类型}

bool

string 大小小于2\^32

bytes 大小可能大于2\^32

##### enum {#enum}

```proto
enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
  Corpus corpus = 4;
}
```

##### map {#map}

```proto
map<key_type, value_type> map_field = N;
```

> …where the key_type can be any integral or string type (so, any scalar type except for floating point types and bytes). Note that enum is not a valid key_type. The value_type can be any type except another map.

key_type可以是:标量(除bytes和bool),value_type不能是map,不能嵌套map

##### any {#any}

Any 消息类型允许您将消息作为嵌入类型使用，而不需要它们的.proto 定义。

Any 以字节的形式包含任意序列化的消息，以及作为该消息类型的全局唯一标识符并解析为该消息类型的 URL。要使用 Any 类型，您需要 import google/protobuf/any.proto

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

##### oneof {#oneof}

如果需要一条包含多个字段的消息，并且最多同时设置一个字段，可以强制执行此行为并使用 oneof 功能节省内存。

oneof 字段与常规字段一样，在 oneof 共享内存中的所有字段，最多可以同时设置一个字段。设置 oneof 的任何成员会自动清除所有其他成员。

如果设置了多个值，则由 proto 中的 order 确定的最后一个设置的值将覆盖所有以前的设置值。

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

#### Field Numbers {#field-numbers}

每个字段都要有一个标识，这里还是比较复杂的。

先说基本要求：

1.  field numbers 必须唯一（指在那个message内）
2.  19,000-19,999是被pb内部使用的，不能使用
3.  不能使用reserved的和已经分配给extension的
4.  Field number不能被reuse，这其实是在说delete的时候要把这个field number和对应的name加到reserved里面去，之后都不能用

然后是一些tricks：尽量把经常用到的字段分配小的field number因为他们的占用相对小，而且在编码的时候也会更加高效。

#### Field labels {#field-labels}

- optional
  - field is set, 也就是有一个确切值
  - field is unset,那么会返回的是default value
- repeated 其实就是说这个字段是一个数组
- map
- implicit field message(如果没有label)

### package {#package}

给一个.proto文件指定package, 是为了避免和其他的.proto文件的message名称冲突.

option go_package = “github.com/TripleCGame/apis/api;api”;—前一个参数用于指定生成文件的位置，后一个参数指定生成的 .go 文件的 package 。具体语法如下：

```proto
option go_package = "{out_path};out_go_package";
```

注意：这里指定的 out_path 并不是绝对路径，只是相对路径或者说只是路径的一部分，和 protoc 的 –go_out 拼接后才是完整的路径。

## 定义service {#定义service}

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

## 命令 {#命令}

```sh
protoc -I . -I /usr/local/include -I $(GOPATH)/src --go_out=. simple.proto
```

-I 或 –proto_path: 指定import搜索的文件夹路径

–go_out : 编译产生的 go代码 的生成位置

### 代码生成规则 {#代码生成规则}

每个xxx.proto文件, 生成一个xxx.pb.go文件.

Go文件的package文件的生成规则:

- case1: .proto文件不包含package

  - 如果一个.proto文件中不包含package声明，生成的源代码将会使用.proto文件的文件名(去掉扩展名)作为Go包名，
  - 其中, 生成的Go的包名中会自动把.转换为\_。
  - 举例来说一个名为high.score.proto不包含package声明的proto文件, 将会生成文件high.score.pb.go，他的Go包名是high_score。

- case2: .proto文件包含package

  - 经过编译的Go代码的包名默认使用proto文件的package名称, 除非在.proto文件中使用option go_package显示进行指定Go文件的import path.
  - 其中, 生成的Go的包名中会自动把.转换为\_。
  - 举例来说proto包名example.high_score将会生成Go包名example_high_score。

- case3: .proto文件包含package, 同时包含option go_package

  - option go_package 的作用是去指定生成的go文件的package的完整导入路径(import path)的, 如:

  `option go_package = "example.com/foo/bar"; // 生成的foo.pb.go文件的package的import path就是 "example.com/foobar"`

## 参考 {#参考}

1.  [Protocol Buffers Documentation](https://protobuf.dev/programming-guides/proto3/)
2.  [这篇文章讲的特别好](https://juejin.cn/post/6865126893063471112)
