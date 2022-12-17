# 第02步：处理玩家的输入

在本课中，你将学习如何：

- 在不同的终端模式下工作
- 从Go代码中调用外部命令
- 向终端发送转义序列
- 从标准输入读取
- 创建一个可以返回多个值的函数

## 概述

在上一步中，我们已经学会了如何向标准输出端打印东西。现在是时候学习如何从标准输入读取数据了。

在这个游戏中，我们将处理一组有限的动作：上、下、左和右。除了这些，我们唯一要使用的键是`Esc`键，以便使玩家能够优雅地退出游戏。这些动作将被映射到方向键上。

这一步将处理`Esc`键，我们将在第03步看到如何处理方向键。

但是，在进入实现之前，我们需要了解一下终端模式。

## 终端模式的介绍

终端可以在三种可能的[modes](https://en.wikipedia.org/wiki/Terminal_mode)下运行：

1. Cooked模式
2. Cbreak模式
3. Raw模式

Cooked模式是我们习惯使用的模式。在这种模式下，终端收到的每一个输入都是经过预处理的，也就是说，系统会拦截特殊字符，赋予它们特殊的含义。

注意：特殊字符包括退格、删除、Ctrl+D、Ctrl+C、方向键等等

Raw模式则相反：数据按原样传递，不做任何形式的预处理。

Cbreak模式是一个中间地带。有些字符被预处理了，有些则没有。例如，Ctrl+C仍然会导致程序中断，但方向键会原封不动地传递给程序。

我们将使用Cbreak模式来允许我们处理与`Esc`和方向键相对应的转义序列。

## 任务01：启用Cbreak模式

为了启用Cbreak模式，我们将调用一个控制终端行为的外部命令，即`stty`命令。我们还将禁用终端输出(terminal echo)，这样我们就不会用按键的输出来污染屏幕了。

下面是我们的 "init "的定义：

```go
func initialise() {
    cbTerm := exec.Command("stty", "cbreak", "-echo")
    cbTerm.Stdin = os.Stdin

    err := cbTerm.Run()
    if err != nil {
        log.Fatalln("unable to activate cbreak mode:", err)
    }
}
```

如果你的IDE没有配置为自动添加，你需要添加导入 "os/exec"。

`log.Fatalln`函数将在出现错误时，在打印日志后终止程序。这在这里很重要，因为如果没有Cbreak模式，游戏就无法进行。由于这是我们在程序中要调用的第一个函数，我们并不担心跳过任何延迟调用的问题

## 任务02：恢复Cooked模式

恢复Cooked模式是一个相当简单的过程。它与启用Cbreak模式相同，但标志是相反的。

```go
func cleanup() {
    cookedTerm := exec.Command("stty", "-cbreak", "echo")
    cookedTerm.Stdin = os.Stdin

    err := cookedTerm.Run()
    if err != nil {
        log.Fatalln("unable to restore cooked mode:", err)
    }
}
```

现在我们需要在`main`函数中调用这两个函数。

```go
func main() {
    // initialise game
    initialise()
    defer cleanup()

    // load resources
    // ...
```

## 任务03：从标准输入读取数据

从标准输入口读取数据的过程包括调用函数`os.stdin.Read`，并给定一个读取缓冲区。

`os.Stdin.Read`返回两个值：读取的字节数和一个错误值。请看下面的`readInput`函数的代码。

```go
func readInput() (string, error) {
    buffer := make([]byte, 100)

    cnt, err := os.Stdin.Read(buffer)
    if err != nil {
        return "", err
    }

    if cnt == 1 && buffer[0] == 0x1b {
        return "ESC", nil
    }

    return "", nil
}
```

The `make` function is a [built-in function](https://golang.org/pkg/builtin/#make) that allocates and initialises objects. It is only used for slices, maps and channels. In this case we are creating an array of bytes with size 100 and returning a slice that points to it.
`make`函数是一个[内置函数](https://golang.org/pkg/builtin/#make)，用于分配和初始化对象。它只用于分片、映射和通道(slices, maps and channels)。在这个例子中，我们要创建一个大小为100的字节数组，并返回一个指向它的切片。

在通常的错误处理之后（我们只是在调用堆栈上传递错误），我们要测试我们是否只读了一个字节，以及该字节是否是转义键。(0x1b是代表Esc的十六进制代码）。

如果Esc键被按下，我们返回 "ESC"，否则返回一个空字符串。

现在你可能想知道为什么要分配一个100字节的缓冲区，或者为什么要测试一个确切的字节的计数...

如果缓冲区突然有5个元素，其中一个是Esc键怎么办？我们难道不应该关心一下这个问题吗？那个键会不会丢失？

简短的回答是我们不应该关心。请记住，这是一个游戏。根据处理速度和你的键盘缓冲区的长度，如果我们按顺序处理事件，我们可能会引入移动滞后(movement lag)，也就是说，有一个尚未处理的箭头键队列。

由于我们是在一个循环中读取输入的，把所有的按键放在一个队列中，只关注最后一个按键，这并没有什么损失。这将使游戏的反应比我们关注每一个按键的情况要好。

## 任务04：更新游戏的循环程序

现在是时候更新游戏循环了，让`readInput'函数在每个迭代中被调用。请注意，如果发生错误，我们也需要中断循环。

```go
// process input
input, err := readInput()
if err != nil {
    log.Print("error reading input:", err)
    break
}
```

最后，我们可以摆脱那个永久性的`break`语句，开始测试 "ESC "键的按下情况。

```go
if input == "ESC" {
    break
}
```

## 任务05：清空屏幕

由于我们现在有了一个适当的游戏循环，我们需要在每次循环后清除屏幕，这样我们就有了一个空白的屏幕，以便在下一次迭代中绘图。为了做到这一点，我们将使用一些特殊的转义序列(Escape sequences)。

[Escape sequences](https://en.wikipedia.org/wiki/ANSI_escape_code#Escape_sequences) 之所以被称为转义序列，是因为它们以ESC字符（0x1b）开头，后面跟着一个或多个字符。这些字符作为终端仿真器的命令发挥作用。

实际上你不需要担心我们将要使用的序列，因为我们将导入另一个叫做`simpleansi`的包，它为我们做这些工作。

```go
import "github.com/danicat/simpleansi"
```

---
### 关于外部软件包的说明

这一次我们不是从标准库中导入一个包，而是导入一个外部包。如果你看一下`simpleansi`的[实现](https://github.com/danicat/simpleansi)，你会注意到每个函数都以大写字母开头，比如`ClearScreen`或`MoveCursor`。

这在Go中很重要，因为一个词的大写字母定义了该函数或变量是否有**公共的**或**私人的**范围。

以小写字母开头的单词对定义它的包来说是私有的，而以大写字母开头的单词是公共的。这可能会让来自其他语言（如java）的人感到困惑，但如果你遵循 "类（结构）总是以大写字母开头 "这样的命名惯例，你最终可能会无意中使你的代码中的每个类型都变成公共的，这可能不是你想要的。
---

我们将更新printScreen函数，在打印前调用`simpleansi.ClearScreen`，这样我们就能确保每一帧都使用空白屏幕。

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        fmt.Println(line)
    }
}
```

现在再次运行游戏并尝试按下`ESC`键。

请注意，如果你偶然按了Ctrl+C，程序将在不调用清理函数的情况下终止，所以你将无法看到你在终端中输入的内容（因为有`-echo`标志）。

如果你遇到这种情况，要么关闭终端并重新打开它，要么就再次运行游戏并使用`ESC`键优雅地退出。

[带我到第03步！](../step03/README.md)