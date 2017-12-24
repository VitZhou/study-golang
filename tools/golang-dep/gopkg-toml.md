# Gopkg.toml

## required

required 列表是Gopkg.lock(不是project)中必须包含的。该列表与当前项目导入的一组包合并。

他将被用于:linters,generator或者其他开发工具。

- 需要你的项目
- 不是由你的项目直接或者间接导入
- 你不想放在你的$GOPATH中,你想锁定版本

请注意，它只关注这些依赖关系的来源。 它不会安装或编译它们。 所以，如果你需要安装这个工具，你应该在每个dep ensure之后运行下面的命令（手工或者从Makefile）：

```shell
cd vendor/pkg/to/install
go install .
```

如果安装这些可执行文件的项目只有一个的话，那么它是可以安全的工作的。 但是如果你想在不同项目中运行不同版本的相同可执行文件的话，上面的命令是不够的。 在这种情况下，您必须对每个项目使用不同的GOBIN，在执行上述命令之前先执行下面的操作：

```shell
export GOBIN=$PWD/bin
export PATH=$GOBIN:$PATH
```

您也可以尝试使用[virtualgo](https://github.com/GetStream/vg)，它会自动在项目特定的GOBIN中安装required列表中的依赖关系。

## ignored

ignored列表是静态分析源码时被忽略的一组包。ignored的包可以在这个项目或者依赖项中.
它被用于：防止安装程序包和该程序包的任何独特依赖项。

```xml
ignored = ["github.com/user/project/badpkg"]
```

使用通配符（*）来定义要忽略的包前缀。 使用它来忽略任何包及其子包。

```xml
ignored = ["github.com/user/project/badpkg*"]
```

## metadata

metadata可以存在于跟或者constraint和overrid声明之下.
metadata声明被dep忽略，意味着被其他独立系统使用。
根metadata声明定义了关于项目本身的metadata。 在constraint或overrid下的metadata声明定义了关于该constraint或ovrreid的medata会覆盖根metadata。

```xml
[metadata]
key1 = "value that convey data to other systems"
system1-data = "value that is used by a system"
system2-data = "value that is used by another system"
```

## constraint

constraint提供了如何将[直接依赖](https://github.com/golang/dep/blob/master/docs/FAQ.md#what-is-a-direct-or-transitive-dependency)关系并入到依赖图中的规则。 无论是来自当前项目的Gopkg.toml还是依赖项，他们都会被关注。

```xml
[[constraint]]
  # Required: the root import path of the project being constrained.
  name = "github.com/user/project"
  # Recommended: the version constraint to enforce for the project.
  # Only one of "branch", "version" or "revision" can be specified.
  version = "1.0.0"
  branch = "master"
  revision = "abc123"

  # Optional: an alternate location (URL or import path) for the project's source.
  source = "https://github.com/myfork/package.git"

  # Optional: metadata about the constraint or override that could be used by other independent systems
  [metadata]
  key1 = "value that convey data to other systems"
  system1-data = "value that is used by a system"
  system2-data = "value that is used by another system"
```

## override

override与constraint声明具有相同的结构，但取代所有项目的所有contraint声明。 只应用于来当前项目的Gopkg.toml的ovrried声明。

```xml
[[override]]
  # Required: the root import path of the project being constrained.
  name = "github.com/user/project"
  # Optional: specifying a version constraint override will cause all other constraints on this project to be ignored; only the overridden constraint needs to be satisfied. Again, only one of "branch", "version" or "revision" can be specified.
  version = "1.0.0"
  branch = "master"
  revision = "abc123"
  # Optional: specifying an alternate source location as an override will enforce that the alternate location is used for that project, regardless of what source location any dependent projects specify.
  source = "https://github.com/myfork/package.git"

  # Optional: metadata about the constraint or override that could be used by other independent systems
  [metadata]
  key1 = "value that convey data to other systems"
  system1-data = "value that is used by a system"
  system2-data = "value that is used by another system"
```

它被用于:很多东西具有一个相同的constraint,但是要传递依赖，并且限制传递依赖项的版本. 请参阅[如何限制传递依赖项的版本](https://github.com/golang/dep/blob/master/docs/FAQ.md#how-do-i-constrain-a-transitive-dependencys-version)


