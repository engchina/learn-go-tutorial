# 第01步：输入和输出

在本课中，你将学习如何：

- 从文件中读取
- 打印到标准输出
- 处理多个返回值
- 处理错误
- 创建并添加一个元素到一个切片
- 在一个切片上进行范围循环
- 延迟一个函数的调用
- 记录错误

## 概述

我们已经掌握了基础知识，现在是时候开始这个游戏了！

首先，我们要读取迷宫的数据。我们有一个叫做`maze01.txt`的文件，基本上是迷宫的ASCII表示（如果你愿意，可以用文本编辑器打开它）。你可以假设：

```
- # 代表一堵墙
- . 代表一个点
- P 代表玩家
- G 代表鬼魂（敌人）
- X 代表力量增强的药丸
```

我们的第一个任务包括将这个迷宫的ASCII表示法加载到一个字符串片断中，然后将其打印到屏幕上。看起来很简单，对吗？的确如此！

## 任务01：加载迷宫

让我们从读取`maze01.txt`文件开始。

我们将使用`os`包中的函数`Open`来打开它，并使用缓冲IO包（`bufio`）中的扫描器对象将其读入内存（到一个名为`maze`的全局变量）。最后我们需要通过调用`os.Close`来释放文件处理程序。

所有这些都是下面的代码。

```go
var maze []string

func loadMaze(file string) error {
    f, err := os.Open(file)
    if err != nil {
        return err
    }
    defer f.Close()

    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }

    return nil
}
```

现在让我们把它分解，看看发生了什么事。

请注意，你需要导入`bufio`和`os`包，如下所示。

```go
import "bufio"
import "os"
```

另外，由于你已经有一个导入（`fmt`），你可以把它作为一个列表添加。

```go
import (
    "bufio"
    "fmt"
    "os"
)
```

`os.Open()`函数返回一对值：一个文件和一个错误。从一个函数中返回多个值是Go中的一个常见模式，特别是在返回错误时。

```go
f, err := os.Open(file)
```

`:=`运算符是一个赋值运算符，但它有一个特殊的属性，即根据右边的值自动推断变量的类型。

请记住，Go是一种强类型语言，但是这个很好的特性使我们在可以推断类型的时候省去了指定类型的麻烦。

在上面的例子中，Go自动推断了`f`和`err`两个变量的类型。

当一个函数返回一个错误时，紧接着检查这个错误是一个常见的模式。

```go
    f, err := os.Open(file)
    if err != nil {
        // do something with err
        log.Print("...")
        return
    }
```

注意：保持"快乐路径(happy path)"向左对齐，"悲伤路径(sad path)"向右对齐（即提前终止函数）是一个好的做法。

`nil`在Go中意味着没有给一个变量赋值。

如果条件为真，`if`语句会执行一个语句。它可以像`for`语句一样有一个初始化子句，还有一个`else`子句，在条件为假时运行。请记住，创建的变量的范围将只是if语句的主体。比如说：

```go
// optional initialization clause
if foo := rand.Intn(2); foo == 0 {
    fmt.Print(foo) // foo is valid here
} else {
    fmt.Print(foo) // and here
}
// but you can't use foo here!
```

`loadMaze`代码的另一个有趣的方面是`defer`关键字的使用。它基本上是说在当前函数的末尾调用`defer`之后的函数。它对清理工作非常有用，在本例中，我们用它来关闭我们刚刚打开的文件。

```go
func loadMaze(file) error {
    f, err := os.Open(file)
    // omitted error handling
    defer f.Close() // puts f.Close() in the call stack

    // rest of the code

    return nil
    // f.Close is called implicitly
}
```

代码的下一部分只是逐行读取文件并将其追加到`maze`切片中。

```go
    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        line := scanner.Text()
        maze = append(maze, line)
    }
```

`scanner`是读取文件的一种非常方便的方式。`scanner.Scan()`会在有东西要从文件中读取时返回true，`scanner.Text()`会返回下一行的输入。

`append`内置函数负责向`maze`切片添加新元素。

## 任务02：打印到屏幕上

一旦我们将迷宫(maze)文件加载到内存中，我们需要将其打印到屏幕上。

一种方法是遍历`maze`切片中的每一个条目并打印它。这可以通过`range`操作符方便地完成。

```go
func printScreen() {
    for _, line := range maze {
        fmt.Println(line)
    }
}
```

请注意，我们正在使用`:=`赋值运算符来初始化两个值：下划线（_）和`line`变量。下划线只是一个占位符，用来表示编译器对变量名称的期望。使用下划线意味着我们要忽略这个值。

在`range`操作符的情况下，第一个返回值是元素的索引，从0开始。第二个返回值是值本身。

如果我们不写下划线字符来忽略第一个值，range操作符将只返回索引（而不是值）。比如说：

```go
for idx := range maze {
    fmt.Println(idx)
}
```

因为在这种情况下，我们只关心内容而不关心索引，所以我们可以安全地忽略索引，把它赋值给下划线。

## 任务03：更新游戏循环

现在我们有了`loadMaze'和`printScreen'两个函数，我们应该更新`main'函数来初始化迷宫并在游戏循环中打印它。请看下面的代码。

```go
func main() {
    // initialise game

    // load resources
    err := loadMaze("maze01.txt")
    if err != nil {
        log.Println("failed to load maze:", err)
        return
    }

    // game loop
    for {
        // update screen
        printScreen()

        // process input

        // process movement

        // process collisions

        // check game over

        // Temp: break infinite loop
        break

        // repeat
    }
}
```

像往常一样，我们保持快乐的路径，所以如果`loadMaze`函数失败，我们使用`log.Println`来记录它，然后`return`来终止程序执行。由于我们使用了一个新的包，`log`，请确保它被添加到导入部分。

```go
import (
    "bufio"
    "fmt"
    "log"
    "os"
)
```

注意：也可以使用`log.Fatalln`达到同样的效果，但我们需要确保任何延迟调用在退出`main`函数之前执行，`log.Fatal`系列的函数通过内部调用`os.Exit(1)`跳过延迟函数调用。我们在main函数中还没有任何延迟调用，但我们将在下一章添加一个。

现在我们已经完成了游戏循环的修改，我们可以用`go run`来运行程序，或者用`go build`来编译，然后作为一个独立的程序运行。

```sh
go run main.go
```

你应该可以看到打印到终端的迷宫。

[带我到第02步！](../step02/README.md)
