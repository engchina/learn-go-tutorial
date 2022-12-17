# 第03步：添加移动

在本课中，你将学习如何：

- 创建一个结构体
- 使用开关语句
- 处理方向键
- 使用命名的返回值

## 概述

我们有一个迷宫，我们可以优雅地退出游戏......但没有什么非常令人兴奋的事情发生，对吗？因此，让我们给这个东西加点料，增加一些动作。

在这一步中，我们将添加玩家角色，并通过方向键使其移动。

## 任务01：跟踪玩家的位置

我们旅程的第一步是创建一个变量来保存玩家的数据。由于我们将跟踪2D坐标（行和列），我们将定义一个结构来保存这些信息。

```go
type sprite struct {
    row int
    col int
}

var player sprite
```

为了简单起见，我们也将玩家定义为一个全局变量。

接下来我们需要在加载迷宫时，在`loadMaze`函数中捕捉玩家的位置。

```go
// traverse each character of the maze and create a new player when it locates a `P`
for row, line := range maze {
    for col, char := range line {
        switch char {
        case 'P':
            player = sprite{row, col}
        }
    }
}
```

注意，这次我们使用的是`range`运算符的完整形式，因为我们对找到玩家的行和列感兴趣。

下面是完整的`loadMaze`。

```go
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

    for row, line := range maze {
        for col, char := range line {
            switch char {
            case 'P':
                player = sprite{row, col}
            }
        }
    }

    return nil
}
```

---

### 可选。关于可见性的说明

我们在这里保持简单，只是为了本教程的目的。因为所有的东西都是一个文件，所以我们没有考虑到变量的可见性，例如，没有考虑它们是公共的或私有的。

尽管如此，Go在定义可见性方面有一个有趣的机制。它没有一个公共的关键字，而是将每一个名字以大写字母开头的符号视为公共的。另一方面，如果名字以小写字母开头，它就是一个私有符号。

这就是为什么到目前为止我们所使用的每个库函数名称都是以大写字母开头的。这也是为什么如果你定义任何变量、函数或类型时以大写字母开头，你的IDE会抱怨缺少注释。在Go的习语中，公共符号应该总是被注释，因为这些符号以后会被提取出来成为包的文档。

在这个特殊的例子中，我们对所有的变量、类型和函数都使用小写的符号，因为从包`main`中导出一个符号没有任何意义。

---

## 任务02：处理方向键

接下来，我们需要修改`readInput`来处理方向键。

```go
if cnt == 1 && buffer[0] == 0x1b {
    return "ESC", nil
} else if cnt >= 3 {
    if buffer[0] == 0x1b && buffer[1] == '[' {
        switch buffer[2] {
        case 'A':
            return "UP", nil
        case 'B':
            return "DOWN", nil
        case 'C':
            return "RIGHT", nil
        case 'D':
            return "LEFT", nil
        }
    }
}
```

方向键的转义序列有3个字节，以`ESC+[`开始，然后是A到D的一个字母。

我们现在需要一个函数来处理这个动作。

```go
func makeMove(oldRow, oldCol int, dir string) (newRow, newCol int) {
    newRow, newCol = oldRow, oldCol

    switch dir {
    case "UP":
        newRow = newRow - 1
        if newRow < 0 {
            newRow = len(maze) - 1
        }
    case "DOWN":
        newRow = newRow + 1
        if newRow == len(maze) {
            newRow = 0
        }
    case "RIGHT":
        newCol = newCol + 1
        if newCol == len(maze[0]) {
            newCol = 0
        }
    case "LEFT":
        newCol = newCol - 1
        if newCol < 0 {
            newCol = len(maze[0]) - 1
        }
    }

    if maze[newRow][newCol] == '#' {
        newRow = oldRow
        newCol = oldCol
    }

    return
}
```

注意：如果你习惯了其他语言中的switch语句，请注意在Go中，每个case条件后都有一个隐含的`break`。所以我们不需要在每个块之后明确地中断。如果我们想跳过下一个`case`块，我们可以使用`fallthrough`关键字。


上面的函数利用了`命名的返回值(named return values)`来返回移动后的新位置（`newRow`和`newCol`）。基本上，该函数首先 "尝试 "移动，如果新的位置碰壁（`#`），移动就被取消了。

它还处理了这样一个属性：如果角色移动到迷宫范围之外，它就会出现在对面的位置。

移动拼图的最后一块是定义一个函数来移动玩家。

```go
func movePlayer(dir string) {
    player.row, player.col = makeMove(player.row, player.col, dir)
}
```

## 任务03：更新迷宫

我们已经有了所有的运动逻辑，但是我们需要让屏幕反映出来。我们将重构`printScreen`函数，只打印我们想要打印的东西，而不是整个地图。

这将给我们更多的控制权，使我们能够用`moveCursor`函数在任意位置打印玩家。请看下面的代码。

```go
func printScreen() {
    simpleansi.ClearScreen()
    for _, line := range maze {
        for _, chr := range line {
            switch chr {
            case '#':
                fmt.Printf("%c", chr)
            default:
                fmt.Print(" ")
            }
        }
        fmt.Println()
    }

    simpleansi.MoveCursor(player.row, player.col)
    fmt.Print("P")


    // Move cursor outside of maze drawing area
    simpleansi.MoveCursor(len(maze)+1, 0)
}
```

目前，我们忽略了任何不是墙或玩家的东西。

## 任务04：动画！

最后，我们需要在游戏循环中调用`movePlayer`。

```go
// game loop
for {
    // update screen
    printScreen()

    // process input
    input, err := readInput()
    if err != nil {
        log.Println("error reading input:", err)
        break
    }

    // process movement
    movePlayer(input)

    // process collisions

    // check game over
    if input == "ESC" {
        break
    }

    // repeat
}
```

我们可以开始了！

[带我到第04步！](../step04/README.md)
