# Cobra

![](../img/3.png)
Cobra既是创建功能强大的现代CLI应用程序的库，也是生成应用程序和命令文件的程序。
许多使用最广泛的Go项目都是使用Cobra构建的，其中包括：
- [Kubernetes](http://kubernetes.io/)
- [Hugo](http://gohugo.io/)
- [rkt](https://github.com/coreos/rkt)
- [etcd](https://github.com/coreos/etcd)
- [Moby (former Docker)](https://github.com/moby/moby)
- [Docker (distribution)](https://github.com/docker/distribution)
- [OpenShift](https://www.openshift.com/)
- [Delve](https://github.com/derekparker/delve)
- [GopherJS](http://www.gopherjs.org/)
- [CockroachDB](http://www.cockroachlabs.com/)
- [Bleve](http://www.blevesearch.com/)
- [ProjectAtomic (enterprise)](http://www.projectatomic.io/)
- [GiantSwarm's swarm](https://github.com/giantswarm/cli)
- [Nanobox](https://github.com/nanobox-io/nanobox)/[Nanopack](https://github.com/nanopack)
- [rclone](http://rclone.org/)
- [nehm](https://github.com/bogem/nehm)
- [Pouch](https://github.com/alibaba/pouch)

依赖:

```xml
[[constraint]]
  name = "github.com/spf13/cobra"
  revision = "7b2c5ac9fc04fc5efafb60700713d4fa609b777b"
```

## 概述

Cobra是一个库，提供了一个简单的接口来创建功能强大的现代CLI 接口，类似于git＆go工具。
Cobra也是一个应用程序，将生成您的应用程序脚手架，以便迅速开发基于Cobra的应用程序。
Cobra提供以下功能:

- 简单的基于子命令的CLI：app server，app fetch等.
- 完全符合POSIX的标志（包括短版本和长版本）
- 嵌套的子命令
- Global, local 和 cascading(级联)标志
- 使用cobra init appname＆cobra add cmdname轻松生成应用程序和命令
- 智能建议(app srver... did you mean app server?)
- 自动帮助生成命令和标志
- 自动帮助标志识别-h，--help等
- 为您的应用程序自动生成bash自动完成
- 为您的应用程序自动生成手册
- 命令别名，所以你可以改变命令，而不会打破它们
- 灵活定义自己的帮助，使用等
- 与可选的12个应用程序的viper紧密集成

## 概念

Cobra建立在command，arguments和flags的结构上。
命令表示动作，参数是事物，标志是这些动作的修饰符。
最好的应用程序将在使用时阅读起来像句子。 用户将知道如何使用该应用程序，方便他们将在本地理解如何使用它。
遵循的模式是APPNAME VERB NOUN --ADJECTIVE。 或APPNAME COMMAND ARG --FLAG
一些很好的现实世界的例子可能会更好地说明这一点。
在下面的例子中，'server'是一个命令，'port'是一个标志:

```shell
hugo server --port=1313
```

类似于下面这个命令告诉我们git clone的url:

```shell
git clone URL --bare
```

### 命令

Command是应用程序的中心点。 应用程序支持的每个交互都将包含在Command中。 一个命令可以有子命令并可以选择运行一个动作。
在上面的例子中，“server”是命令。

[更多](https://godoc.org/github.com/spf13/cobra#Command)关于Cora Command内容.

### 标志

Flag是修改命令行为的一种方法。 Cobra支持完全符合POSIX的标志以及Go标志包。 Cobra命令可以定义持续到子命令的标志，以及只对该命令可用的标志。

在上面的例子中，“port”是标志。

标志功能是由[pflag库](https://github.com/spf13/pflag)提供的，它是标志标准库的一个分支，它在保持接口相同的同时增加了POSIX合规性。

## 安装

使用Cobra很容易。 首先，使用go get来安装最新版本的库。 该命令将安装Cobra生成器可执行文件以及库及其依赖项：

```shell
$ go get -u github.com/spf13/cobra/cobra
$ import "github.com/spf13/cobra"
```


下一步,添加Cobra到你的应用中.

## 入门

通常基于眼镜蛇的应用程序将遵循以下组织结构：

```xml
▾ appName/
    ▾ cmd/
        add.go
        your.go
        commands.go
        here.go
      main.go
```

在Cobra应用程序中，通常main.go文件非常空。 它只有有一个目的：初始化Cobra。

```go
package main

import (
  "fmt"
  "os"

  "{pathToYourApp}/cmd"
)

func main() {
  cmd.Execute()
}
```

### 使用Cobra生成应用

Cobra提供创建你应用程序的程序,并且添加你想要的命令。用Cobra生成你的应用是非常容易的.
[这里](https://github.com/spf13/cobra/blob/master/cobra/README.md)你可以找到更多关于它的信息.

### 使用Cobra类库

想要要手动实现Cobra，您需要创建一个空的main.go文件和一个rootCmd文件。 您可以根据需要提供其他命令。

#### rootCmd

Cobra不需要任何特殊的构造函数。只需创建你的命令。

理想情况下，你把它放在app/cmd/root.go：

```go
var rootCmd = &cobra.Command{
  Use:   "hugo",
  Short: "Hugo is a very fast static site generator",
  Long: `A Fast and Flexible Static Site Generator built with
                love by spf13 and friends in Go.
                Complete documentation is available at http://hugo.spf13.com`,
  Run: func(cmd *cobra.Command, args []string) {
    // Do Stuff Here
  },
}
```

您将另外定义flag并处理init()函数中的配置。
比如cmd/root.go:

```go
import (
  "fmt"
  "os"

  homedir "github.com/mitchellh/go-homedir"
  "github.com/spf13/cobra"
  "github.com/spf13/viper"
)

func init() {
  cobra.OnInitialize(initConfig)
  rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra.yaml)")
  rootCmd.PersistentFlags().StringVarP(&projectBase, "projectbase", "b", "", "base project directory eg. github.com/spf13/")
  rootCmd.PersistentFlags().StringP("author", "a", "YOUR NAME", "Author name for copyright attribution")
  rootCmd.PersistentFlags().StringVarP(&userLicense, "license", "l", "", "Name of license for the project (can provide `licensetext` in config)")
  rootCmd.PersistentFlags().Bool("viper", true, "Use Viper for configuration")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
  viper.BindPFlag("projectbase", rootCmd.PersistentFlags().Lookup("projectbase"))
  viper.BindPFlag("useViper", rootCmd.PersistentFlags().Lookup("viper"))
  viper.SetDefault("author", "NAME HERE <EMAIL ADDRESS>")
  viper.SetDefault("license", "apache")
}

func initConfig() {
  // Don't forget to read config either from cfgFile or from home directory!
  if cfgFile != "" {
    // Use config file from the flag.
    viper.SetConfigFile(cfgFile)
  } else {
    // Find home directory.
    home, err := homedir.Dir()
    if err != nil {
      fmt.Println(err)
      os.Exit(1)
    }

    // Search config in home directory with name ".cobra" (without extension).
    viper.AddConfigPath(home)
    viper.SetConfigName(".cobra")
  }

  if err := viper.ReadInConfig(); err != nil {
    fmt.Println("Can't read config:", err)
    os.Exit(1)
  }
}
```

#### 创建你的main.go

要使用root命令，你需要让main函数执行它。尽管可以在任何命令上调用Execute，但为了清晰起见，应该在root上运行Execute。

在Cobra应用程序中，通常main.go文件非常空。 它只有有一个目的，初始化Cobra。

```go
package main

import (
  "fmt"
  "os"

  "{pathToYourApp}/cmd"
)

func main() {
  cmd.Execute()
}
```

#### 创建其他命令

可以定义其他命令，并且通常在cmd/目录中分别指定它们自己的文件。

如果您想创建一个版本命令，您可以创建cmd/version.go并使用以下命令填充它：

```go
package cmd

import (
  "fmt"

  "github.com/spf13/cobra"
)

func init() {
  rootCmd.AddCommand(versionCmd)
}

var versionCmd = &cobra.Command{
  Use:   "version",
  Short: "Print the version number of Hugo",
  Long:  `All software has versions. This is Hugo's`,
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hugo Static Site Generator v0.9 -- HEAD")
  },
}
```

### 使用Flag

Flag提供修饰符控制command操作如何行动.

#### 将标志分配给一个命令

由于标志在不同的位置有不同的定义和使用，我们需要在正确的范围外定义一个变量来分配标志。

```go
var Verbose bool
var Source string
```

有两种不同的方法来分配一个标志。

#### 持久性标志

标志可以是"持久的",这意味着该标志被分配的命令,以及该命令下的每个命令都可用。在根上分配一个标志作为持久标志,就是全局标志。

```go
rootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```

#### 本地标志

标志也可以是本地标志,只适用于被分配的命令.

```go
rootCmd.Flags().StringVarP(&Source, "source", "s", "", "Source directory to read from")
```

#### 父命令上的本地标志

默认情况下，Cobra只解析目标命令的本地标志，父命令的任何本地标志都被忽略。 通过启用Command.TraverseChildren Cobra将在执行目标命令之前分析每个命令上的本地标志。

```go
command := cobra.Command{
  Use: "print [OPTIONS] [COMMANDS]",
  TraverseChildren: true,
}
```

#### 用config绑定标志

可以使用[viper](https://github.com/spf13/viper)绑定你的标志

```go
var author string

func init() {
  rootCmd.PersistentFlags().StringVar(&author, "author", "YOUR NAME", "Author name for copyright attribution")
  viper.BindPFlag("author", rootCmd.PersistentFlags().Lookup("author"))
}
```

在这个例子中，持久标志author绑定了Cobra。 请注意，当用户未提供--author标志时，变量author将不会被设置为来自config的值。

### 位置和自定义参数

位置参数的验证可以使用Command的Args字段指定。
下面是些内置的验证器:

- NoArgs: 如果有任何位置参数，该命令将报告错误。
- ArbitraryArgs: 该命令接受任何标志
- OnlyValidArgs: 如果有任何不在Command的ValidArgs字段中的位置参数，该命令将报告错误。
- MinimumNArgs(int): 至少有N个位置参数，否则该命令将报告错误。
- MaximumNArgs(int)： 最多只能有N个位置参数,否则该命令将报错.
- ExactArgs(int): 正好有N个位置参数,否则该命令将报错.
- RangeArgs(min, max): 位置参数的个数>=min并且<=max，否则报错

设置自定义验证器的示例：

```go
var cmd = &cobra.Command{
  Short: "hello",
  Args: func(cmd *cobra.Command, args []string) error {
    if len(args) < 1 {
      return errors.New("requires at least one arg")
    }
    if myapp.IsValidColor(args[0]) {
      return nil
    }
    return fmt.Errorf("invalid color specified: %s", args[0])
  },
  Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("Hello, World!")
  },
}
```

### 例子

在下面的例子中，我们定义了三个命令。 两个在最高层，一个（cmdTimes）是最高命令之一的子命令。 在这种情况下，根是不可执行的，这意味着子命令是必需的。 这是通过不为“rootCmd”提供“run”来实现的。

我们只为一个命令定义了一个标志。

```go
package main

import (
  "fmt"
  "strings"

  "github.com/spf13/cobra"
)

func main() {
  var echoTimes int

  var cmdPrint = &cobra.Command{
    Use:   "print [string to print]",
    Short: "Print anything to the screen",
    Long: `print is for printing anything back to the screen.
For many years people have printed back to the screen.`,
    Args: cobra.MinimumNArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Println("Print: " + strings.Join(args, " "))
    },
  }

  var cmdEcho = &cobra.Command{
    Use:   "echo [string to echo]",
    Short: "Echo anything to the screen",
    Long: `echo is for echoing anything back.
Echo works a lot like print, except it has a child command.`,
    Args: cobra.MinimumNArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Println("Print: " + strings.Join(args, " "))
    },
  }

  var cmdTimes = &cobra.Command{
    Use:   "times [# times] [string to echo]",
    Short: "Echo anything to the screen more times",
    Long: `echo things multiple times back to the user by providing
a count and a string.`,
    Args: cobra.MinimumNArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
      for i := 0; i < echoTimes; i++ {
        fmt.Println("Echo: " + strings.Join(args, " "))
      }
    },
  }

  cmdTimes.Flags().IntVarP(&echoTimes, "times", "t", 1, "times to echo the input")

  var rootCmd = &cobra.Command{Use: "app"}
  rootCmd.AddCommand(cmdPrint, cmdEcho)
  cmdEcho.AddCommand(cmdTimes)
  rootCmd.Execute()
}
```

### Help Command

当您有子命令时，Cobra会自动向您的应用程序添加一个帮助命令。 这将在用户运行“app help”时被调用。 此外，help也将支持所有其他命令作为输入。 比如说，你有一个叫做“create”的命令，没有任何额外的配置; 当“app help create”被调用时，Cobra将工作。 每个命令都会自动添加“--help”标志。

#### 例子

以下输出由Cobra自动生成。 除了命令和标志定义之外，什么都不需要。

```console
$ cobra help

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobra [command]

Available Commands:
  add         Add a command to a Cobra Application
  help        Help about any command
  init        Initialize a Cobra Application

Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -h, --help             help for cobra
  -l, --license string   name of license for the project
      --viper            use Viper for configuration (default true)

Use "cobra [command] --help" for more information about a command.
```

help就像任何其他的命令一样。 周围没有特殊的逻辑或行为。 事实上，如果你愿意的话，你可以自己提供help命令。

#### 自定义你的help

使用以下功能的默认命令，您可以提供您自己的help命令或您自己的模板：

```go
cmd.SetHelpCommand(cmd *Command)
cmd.SetHelpFunc(f func(*Command, []string))
cmd.SetHelpTemplate(s string)
```

后两者也将适用于任何子命令。

### Usage消息

当用户提供无效标志或无效命令时，Cobra通过向用户显示“usage”做出响应。

#### 例子

你可以从上面的help中认识到这一点。 这是因为help默认嵌入usage作为其输出的一部分。

```console
$ cobra --invalid
Error: unknown flag: --invalid
Usage:
  cobra [command]

Available Commands:
  add         Add a command to a Cobra Application
  help        Help about any command
  init        Initialize a Cobra Application

Flags:
  -a, --author string    author name for copyright attribution (default "YOUR NAME")
      --config string    config file (default is $HOME/.cobra.yaml)
  -h, --help             help for cobra
  -l, --license string   name of license for the project
      --viper            use Viper for configuration (default true)

Use "cobra [command] --help" for more information about a command.
```

#### 自定义你的usage

像help一样,你也可以提供自定义的usage功能或者模板供Cobra使用.函数和模板可以通过公共方法重写:

```go
cmd.SetUsageFunc(f func(*Command) error)
cmd.SetUsageTemplate(s string)
```

### Version标志

如果在root命令上设置了version字段，Cobra将添加顶级的“--version”标志。 使用“--version”标志运行应用程序将使用version模板将version打印到标准输出。 该模板可以使用cmd.SetVersionTemplate（s string）函数进行定制。

### PreRun和PostRun钩子

可以在命令的主要的Run函数运行之前或之后运行。 PersistentPreRun和PreRun函数将在Run之前执行。 PersistentPostRun和PostRun将在Run后执行。 如果Persistent * Run函数没有声明自己，那么它们将被子代继承。 这些功能按以下顺序运行：

1. PersistentPreRun
1. PreRun
1. Run
1. PostRun
1. PersistentPostRun

以下是使用所有这些功能的两个命令的示例。 子命令执行时，将运行root命令的PersistentPreRun，但不运行root命令的PersistentPostRun：

```go
package main

import (
  "fmt"

  "github.com/spf13/cobra"
)

func main() {

  var rootCmd = &cobra.Command{
    Use:   "root [sub]",
    Short: "My root command",
    PersistentPreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PersistentPreRun with args: %v\n", args)
    },
    PreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PreRun with args: %v\n", args)
    },
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd Run with args: %v\n", args)
    },
    PostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PostRun with args: %v\n", args)
    },
    PersistentPostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside rootCmd PersistentPostRun with args: %v\n", args)
    },
  }

  var subCmd = &cobra.Command{
    Use:   "sub [no options!]",
    Short: "My subcommand",
    PreRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PreRun with args: %v\n", args)
    },
    Run: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd Run with args: %v\n", args)
    },
    PostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PostRun with args: %v\n", args)
    },
    PersistentPostRun: func(cmd *cobra.Command, args []string) {
      fmt.Printf("Inside subCmd PersistentPostRun with args: %v\n", args)
    },
  }

  rootCmd.AddCommand(subCmd)

  rootCmd.SetArgs([]string{""})
  rootCmd.Execute()
  fmt.Println()
  rootCmd.SetArgs([]string{"sub", "arg1", "arg2"})
  rootCmd.Execute()
}
```

输出:

```console
Inside rootCmd PersistentPreRun with args: []
Inside rootCmd PreRun with args: []
Inside rootCmd Run with args: []
Inside rootCmd PostRun with args: []
Inside rootCmd PersistentPostRun with args: []

Inside rootCmd PersistentPreRun with args: [arg1 arg2]
Inside subCmd PreRun with args: [arg1 arg2]
Inside subCmd Run with args: [arg1 arg2]
Inside subCmd PostRun with args: [arg1 arg2]
Inside subCmd PersistentPostRun with args: [arg1 arg2]
```

### 当发生'unknown command'时的建议

当'unknow comman'错误发生时，Cobra将自动打印建议。 当错字发生时，这允许Cobra的行为类似于git命令。 例如：

```console
$ hugo srever
Error: unknown command 'srever' for "hugo"

Did you mean this?
        server

Run 'hugo --help' for usage.
```

建议是基于注册的每个子命令自动进行的，并使用[Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance)实现。 每个注册的命令匹配的最小distance为2（忽略大小写）将显示为一个建议。

如果您需要禁用建议或在命令中调整字符串distance，请使用：

```go
command.DisableSuggestions = true
```

或者

```go
command.SuggestionsMinimumDistance = 1
```

您还可以使用SuggestFor属性显式设置给定命令的名称。 这允许对字符串distance不接近的字符串提出建议，对于您的一组命令以及对于某些您不需要别名的字符串来说是很有意义的。 例：

```shell
$ kubectl remove
Error: unknown command "remove" for "kubectl"

Did you mean this?
        delete

Run 'kubectl help' for usage.
```