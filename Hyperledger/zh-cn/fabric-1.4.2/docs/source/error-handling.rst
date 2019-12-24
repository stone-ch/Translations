错误处理
<<<<<<< HEAD
============================

概述
----------------

Hyperledger Fabric代码应该使用供应商提供的软件包 **github.com/pkg/errors** 来代替Go提供的标准错误类型。该软件包允许生成和显示带有错误信息的堆栈跟踪。

## 运用指南

应该使用**github.com/pkg/errors**来代替对 `fmt.Errorf()` 或`errors.New()`的所有调用。使用此程序包将生成一个调用堆栈，该调用堆栈会被附加到错误信息上。

使用该程序包很简单，只需对你的代码做轻微调整。

首先，你需要引入 **github.com/pkg/errors**

然后，更新您的代码生成的所有错误，以使用以下错误创建函数中的一个 (errors.New(), errors.Errorf(), errors.WithMessage(), errors.Wrap(), errors.Wrapf().
=======
==================================

概览
----------------
Hyperledger Fabric 使用 **github.com/pkg/errors** 包来替换 Go 的标准错误类型。这个包可以通过错误信息来简化堆栈追踪的生成和展示。

使用说明
------------------

应该使用 **github.com/pkg/errors** 来替换所有 ``fmt.Errorf()`` 或 ``errors.New()`` 的调用。使用该包可以生成一个调用栈，并把错误信息追加到栈中。

这个包使用简单并且只需要简单的调整你的代码。

首先，你需要导入 **github.com/pkg/errors**。

然后，更新所有使用了错误创建方法（比如，errors.New()、errors.Errorf()、errors.WithMessage()、errors.Wrap()、errors.Wrapf()）的错误。
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

.. note:: 在 https://godoc.org/github.com/pkg/errors 查看所有支持的错误创建方法。也可以参考下边 Fabric 代码使用指南的章节。

<<<<<<< HEAD
最后，将所有记录器或fmt.Printf()调用的格式指令从 ``%s`` 更改为 ``%+v`` ，以打印调用堆栈和错误信息。

Hyperledger Fabric中错误处理的通用准则
-----------------------------------------------------------

- 若要处理用户请求，应记录并返回错误。
- 若错误来自外部来源（如Go文库或vendored软件包），则用errors.Wrap()打包该错误，为其生成一个调用堆栈。
- 若错误来自另一个Fabric函数，当有需要时，在不影响调用堆栈的情况下使用errors.WithMessage()在错误信息中添加更多context。
- Panic不应被传播给其他软件包。 
=======
最后，将所有记录器的格式或 fmt.Printf() 的调用的 ``%s`` 改为 ``%+v``，以便通过错误信息来打印调用栈。

Hyperledger Fabric 中错误处理指南
-----------------------------------------------------------

- 如果你为用户提供服务，你应该记录并返回错误。
- 如果错误来自外部，比如 Go 的依赖，使用 errors.Wrap() 来包装错误以便生成该错误的调用堆栈。
- 如果错误来自 Fabric 的其他方法，如果可以的话就使用 errors.WithMessage() 在离开调用堆栈时，在错误信息中增加更多的上下文信息。
- 致命的错误不应该传播到其他包。
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

示例程序
---------------

<<<<<<< HEAD
以下案列程序清楚地展示了如何使用软件包：
=======
下边的示例程序清晰的展示了包的用法：
>>>>>>> 3915c06381b1ecf6f1e7420afdf0ec132c8e954e

.. code:: go

  package main

  import (
    "fmt"

    "github.com/pkg/errors"
  )

  func wrapWithStack() error {
    err := createError()
    // do this when error comes from external source (go lib or vendor)
    return errors.Wrap(err, "wrapping an error with stack")
  }
  func wrapWithoutStack() error {
    err := createError()
    // do this when error comes from internal Fabric since it already has stack trace
    return errors.WithMessage(err, "wrapping an error without stack")
  }
  func createError() error {
    return errors.New("original error")
  }

  func main() {
    err := createError()
    fmt.Printf("print error without stack: %s\n\n", err)
    fmt.Printf("print error with stack: %+v\n\n", err)
    err = wrapWithoutStack()
    fmt.Printf("%+v\n\n", err)
    err = wrapWithStack()
    fmt.Printf("%+v\n\n", err)
  }

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/