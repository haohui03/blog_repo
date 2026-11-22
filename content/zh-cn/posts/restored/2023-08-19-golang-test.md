---
title: golang test
date: '2023-08-19T08:01:23+08:00'
lastmod: '2023-08-19T09:27:30+08:00'
draft: false
tags:
- golang
- test
categories:
- code
url: /zh-cn/2023/08/19/category/golang-test/
authors:
- yehh
---

## 标准库 {#标准库}

testing

标准测试使用go test 来编译运行测试\*\_test.go文件，这些文件不是go build命令编译的对象。

\*\_test.go 中，有三种测试

### 功能测试 {#功能测试}

每一个测试文件必须导入testing包，并且函数的签名为：

```go
//以Test开头，后面的名称首字母大写
func TestName(t *testing.T){

}
```

#### 命令 {#命令}

go test

- -v 打印详情
- -run=“xxx” xxx是正则表达式可以匹配函数名
- -cover 允许代码分析覆盖率
- -covermode 代码分析模式（set：是否执行；count：执行次数；atomic：次数，并发执行）
- -coverprofile 输出结果文件
- -race 测试竞态

go tool cover -func=cover.out 可以查看具体函数的测试覆盖率

#### Table-Driven Test {#table-driven-test}

通常情况下我们会希望待测函数针对一系列输入都能给出正确的输出，需要验证各个场景，而不是简单的测试单个case就结束。这个时候我们就可以把测试的 input 和 output 成对组装起来，给出一个数组。然后用一次 for 循环依次执行，校验输出是否符合预期。

```go
func TestIntMinTableDriven(t *testing.T) {
    var tests = []struct {
        a, b int // 输入
        want int // 输出
    }{
        {0, 1, 0},
        {1, 0, 0},
        {2, -2, -2},
        {0, -1, -1},
        {-1, 0, -1},
    }

    for _, tt := range tests {
        testname := fmt.Sprintf("%d,%d", tt.a, tt.b) // 测试case名称
        t.Run(testname, func(t *testing.T) {
            ans := IntMin(tt.a, tt.b)
            if ans != tt.want {
                t.Errorf("got %d, want %d", ans, tt.want)
            }
        })
    }
}
```

如果测试文件中包含函数 TestMain，那么生成的测试将调用 TestMain(m)，而不是直接运行测试。

### 基准测试 {#基准测试}

函数签名为

调用时: `go test -bench=.` 运行所有的benchmarks

### 示例函数 {#示例函数}
