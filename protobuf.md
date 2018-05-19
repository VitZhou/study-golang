# Go Protobuf

这个包和它生成的代码至少需要Go 1.4

该软件为protocol buffers的go实现.有关porotocol buffer的信息,请参考
https://developers.google.com/protocol-buffers

## 安装

要使用该软件,你必须:

- 从https://developers.google.com/protocol-buffers 安装protocol buffer的标准C ++实现(MacOs: brew install porobuf)
- 当然也要安装go环境
- 从repository中获取并安装proto package.最简单的方法就是运行go get -u github.com/golang/protobuf/protoc-gen-go.编译器插件protoc-gen-go将安装在$GOBIN中，默认为$GOPATH/bin。 需要将 protocol compiler,protoc添加到$PATH中

这个软件有两个部分：一个“protocol compiler plugin”，用于生成Go源文件，一旦编译，就可以访问和管理protocol buffers; 以及一个实现对编码（编组），解码（解组）和访问protocol buffer的运行时支持的库。
在Go中使用protocol buffer支持gRPC。 有关详细信息，请参阅此文件底部的注释。
插件中没有插入点。

## 在Go中使用protocol buffer

一旦软件安装完毕，有两个步骤来使用它。 首先，您必须编译protocol buffer定义，然后将其与支持库一起导入到您的程序中。
要编译protocol buffer定义，运行protoc，并将--go_out参数设置为要输出Go代码的目录。

```xml
protoc --go_out=. *.proto
```

生成的文件将以.pb.go作为后缀。 有关使用此类文件的示例，请参阅下面的测试代码。
proto库的包注释包含描述Go中为protocol buffer提供的接口的文本。 这是一个编辑的版本。

======

proto软件包将数据结构转换为protocol buffer的有线格式， 它与protocol compiler为.proto文件生成的Go源代码协同工作。
protocol buffer变量v的协议缓冲区接口的属性摘要：

- 消息中的name从camel_case转换为CamelCase用于导出。
- v上没有设置字段的方法;只是把他们当作stucture字段。
- 如果有设置字段的值，getter返回该字段的值，如果未设置,返回该字段的默认值。即使接收者是nil消息，getter也能工作。
- struct的初始化状态是nil.编译前必须设置所有需要的字段.
- Reset（）方法会将protobuf struct恢复到nil状态。
- 非repeated字段是指向值的指针;nil表示未设置。也就是说，可选或必填字段int32 f变成F * int32。
- repeated的字段是切片。
- Helper功能可用于帮助设置字段。用于获取值的helpers将被GetFoo方法所取代，并且不推荐使用它们。 msg.Foo = proto.String（“hello”）//设置字段
- 常量被定义为保存所有字段的默认值。它们的格式为Default_StructName_FieldName。由于getter方法处理默认值，因此直接使用这些常量应该很少。
- 枚举类型名称和从名称到值的映射。枚举值的前缀是枚举的类型名称。枚举类型有一个String方法和一个Enum方法来协助消息构造。
- 嵌套组和枚举具有以周围消息类型的名称为前缀的类型名称。
- 扩展名被赋予以E_开头的描述符名称，随后是包含它的嵌套消息的下划线分隔列表（如果有），后面是扩展字段本身的CamelCased名称。 HasExtension，ClearExtension，GetExtension和SetExtension是用于处理扩展的函数。
- 一个字段集在其消息中被赋予单个字段，对于每个可能的字段值具有可区分的包装类型。
Marshal和Unmarshal是对线格式进行编码和解码的函数。

当.proto文件指定syntax=“proto3”时，会有一些差异：

非消息类型的非重复字段是值而不是指针。
枚举类型不会得到一个枚举方法。

例如文件test.proto,包含如下:

```xml
syntax = "proto2";
package example;

enum FOO { X = 17; };

message Test {
  required string label = 1;
  optional int32 type = 2 [default=77];
  repeated int64 reps = 3;
  optional group OptionalGroup = 4 {
    required string RequiredField = 5;
  }
}
```

要创建和使用示例包中的Test对象，

```go
package main

import (
    "log"

    "github.com/golang/protobuf/proto"
    "path/to/example"
)

func main() {
    test := &example.Test {
        Label: proto.String("hello"),
        Type:  proto.Int32(17),
        Reps:  []int64{1, 2, 3},
        Optionalgroup: &example.Test_OptionalGroup {
            RequiredField: proto.String("good bye"),
        },
    }
    data, err := proto.Marshal(test)
    if err != nil {
        log.Fatal("marshaling error: ", err)
    }
    newTest := &example.Test{}
    err = proto.Unmarshal(data, newTest)
    if err != nil {
        log.Fatal("unmarshaling error: ", err)
    }
    // Now test and newTest contain the same data.
    if test.GetLabel() != newTest.GetLabel() {
        log.Fatalf("data mismatch %q != %q", test.GetLabel(), newTest.GetLabel())
    }
    // etc.
}
```

## 参数

要将额外参数传递给插件，请使用以冒号分隔的输出目录，逗号分隔参数列表：

```shell
protoc --go_out=plugins=grpc,import_path=mypackage:. *.proto
```

- import_prefix=xxx: 添加到所有import开头的前缀。 对于在子目录中生成原型或在vendored中生成protobuf等有用。
- import_path = foo/bar: 如果没有输入文件声明go_package，则用作包。 如果它包含斜线，则忽略最右边的斜线。
- plugins = plugin1 + plugin2: 指定要加载的子插件列表。 这个repo的唯一插件是grpc。
- Mfoo/bar.proto = quux/shme: 声明foo/bar.proto与Go包quux/shme关联。 这受限于import_prefix参数。

## gRPC支持

如果原型文件指定RPC服务，则可以指示protoc-gen-go生成与gRPC（http://www.grpc.io/）兼容的代码。 为此，将plugins参数传递给protoc-gen-go; 通常的方法是将其插入到protoc的--go_out参数中：

```shell
protoc --go_out=plugins=grpc:. *.proto
```
