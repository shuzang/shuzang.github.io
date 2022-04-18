# Go实现2048小游戏


Go 实现一个命令行界面的 2048 游戏，仅涉及 Git 和 Go，用来熟悉基本语言特性。原型项目来自 https://github.com/chhabraamit/2048

<!--more-->

## 1. 环境准备

Win10 环境，go 1.14.3，编辑器为 VScode，使用 Github 管理代码。

首先在网页端建立 Github 仓库，选择 MIT 协议，然后克隆仓库到本地

```bash
$ git clone https://github.com/shuzang/2048.git
```

在项目根目录创建 `main.go` 文件

```go
package main

import "fmt"

func main() {
	fmt.Println("Getting started!")
}
```

初始化项目

```bash
$ go mod init github.com/shuzang
```

## 2. 显示游戏面板

2048 的游戏界面是一个 4×4 的网格，我们使用一个二维切片作为底层结构存储数字，然后按照网格的形式输出到终端，数字随机生成。

```go
// game/board.go
package game

import (
	"fmt"
	"math/rand"
	"time"
)

// 游戏界面规格
const _rows, _cols = 4, 4

type Board interface {
	Display()
}

type board struct {
	board [][]int
}

/* 显示 4×4 网格形式的游戏界面
*/
func (b *board) Display() {
	b.board = generate()
	printHorizontalLine()
	for i := 0; i < _rows; i++ {
		printVerticalLine()
		for j := 0; j < _cols; j++ {
            // 每个数字占7个位置，如果为0，输出空字符
			if b.board[i][j] == 0 {
				fmt.Printf("%7s", "")
			} else {// 数字不为0，使其位于中间，方法是使其占4位，然后接着输出3个空字符
				fmt.Printf("%4d%3s", b.board[i][j], "")
			}
			printVerticalLine()
		}
		fmt.Println()
		printHorizontalLine()
	}
}

// 输出横线，4*7+5 = 33
func printHorizontal() {
	for i := 0; i < 33; i++ {
		fmt.Printf("-")
	}
	fmt.Println()
}

// 输出竖线
func printVertical() {
	fmt.Printf("|")
}

// 生成所需的所有随机数
func generate() [][]int {
	// Store all available numbers from 2 to 2048
	nums := make([]int, 0)
	nums = append(nums, 0)
	for i := 2; i <= 2048; i *= 2 {
		nums = append(nums, i)
	}

	// generate random numbers for init board
	rand.Seed(time.Now().UnixNano())
    matrix := make([][]int, _rows)
	for i := 0; i < _rows; i++ {
		matrix[i] = make([]int, _cols)
	}
	for i := 0; i < _rows; i++ {
		for j := 0; j < _cols; j++ {
			matrix[i][j] = nums[rand.Int()%len(nums)]
		}
	}

	return matrix
}

func NewBoard() *Board {
	return &board{}
}
```

然后修改 main.go 如下

```go
// main.go
package main

import (
	"fmt"

	"github.com/shuzang/2048/game"
)

func main() {
	fmt.Println("Getting started!")
	b := game.NewBoard()
	b.Display()
}
```

运行 `go run main.go` 可以看到一个临时的游戏面板。

## 3. 添加元素

上面的程序随机生成了 16 个数字填充游戏面板，但这只是初始测试，正式游戏的做法是：初始时随机填充两个数字，然后每个键盘输入新增一个数字。这里还要注意每一步生成的数字有两种选择，2 或 4，我们可以控制这两个数字生成的概率。

所以添加新元素被抽象为一个独立的函数，如下

```go
// game/board.go
type board struct {
	board  [][]int
	nx, ny int
}

// 被折叠的代码...

// 随机填充一个新数字
func (b *board) AddElement() {
	rand.Seed(time.Now().UnixNano())
	// 随机选择一个空白位置
	index := make([][2]int, 0)
	for i := 0; i < _rows; i++ {
		for j := 0; j < _cols; j++ {
			if b.board[i][j] == 0 {
				index = append(index, [2]int{i, j})
			}
		}
	}
	next := rand.Int() % len(index)
	nx, ny := index[next][0], index[next][1]
    
	// 按概率选择数字 2 和 4
	var number int
	if rand.Int()%100 < 80 {
		number = 2
	} else {
		number = 4
	}
	// 将数字填充到选择的位置
    b.nx, b.ny = nx, ny
	b.board[nx][ny] = number
}

func NewBoard() *board {
	matrix := make([][]int, _rows)
	for i := 0; i < _rows; i++ {
		matrix[i] = make([]int, _cols)
	}
	return &board{board: matrix}
}
```

board 结构体新增了 nx,ny 两个字段，是为了标记新添加的元素在游戏面板中的位置，我们需要将新元素以不同的颜色表示，这里用到了 fatih/color 包。

```bash
$ go get -v github.com/gatih/color
```

导入该包后修改显示函数如下，用不同的颜色输出新添加的元素。

```go
// game/board.go
func (b *board) Display() {
	c := color.New(color.FgCyan, color.Bold)
	printHorizontalLine()
	for i := 0; i < _rows; i++ {
		printVerticalLine()
		for j := 0; j < _cols; j++ {
			if b.board[i][j] == 0 {
				fmt.Printf("%7s", "")
			} else if i == b.nx && j == b.ny {
				c.Printf("%4d%3s", b.board[i][j], "")
			} else {
				fmt.Printf("%4d%3s", b.board[i][j], "")
			}
			printVerticalLine()
		}
		fmt.Println()
		printHorizontalLine()
	}
}
```

相应的，主函数修改如下，添加 10 个元素并输出

```go
// main.go
func main() {
	fmt.Println("Getting started!")
    b := game.NewBoard()
	for i := 0; i < 10; i++ {
		b.Display()
		b.AddElement()		
	}
    fmt.Println("Game over!")
}
```

## 4. 先清屏后显示

上面的程序会把每一步的面板都输出到终端，我们应当添加的一个功能是，每一步只输出当前的游戏面板。该功能通过清屏函数实现，注意，清屏的实现在不同操作系统可能会有区别，下面的实现适用于 Windows 系统。

```go
// game/board.go
func (b *board) Display() {
	// clear screen, but only works on windows
	cmd := exec.Command("cmd", "/c", "cls")
	cmd.Stdout = os.Stdout
	cmd.Run()
	c := color.New(color.FgCyan, color.Bold)
	printHorizontalLine()
	for i := 0; i < _rows; i++ {
		printVerticalLine()
		for j := 0; j < _cols; j++ {
			if b.board[i][j] == 0 {
				fmt.Printf("%7s", "")
			} else if i == b.nx && j == b.ny {
				c.Printf("%4d%3s", b.board[i][j], "")
			} else {
				fmt.Printf("%4d%3s", b.board[i][j], "")
			}

			printVerticalLine()
		}
		fmt.Println()
		printHorizontalLine()
	}
}
```

## 5. 获取键盘输入

游戏的每一步操作肯定都是根据键盘的输入来的，可以使用 {W, A, S, D} 和 方向键，如果使用 fmt 包中的输入函数，那么每次输入后都需要手动按下回车。为了不必每次输入字符后都敲一下回车键，我们使用 eiannone/keyboard 包

```bash
$ go get -v github.com/eiannone/keyboard
```

将键盘输入对应的几个操作定义为几个常量，然后调用 keyboard 包中的 GetKeyStrokes() 函数接收键盘输入，返回对应的常量，最后在 TakeInput() 函数中根据不同常量交给对应的操作函数处理。

```go
// game/board.go
type Key int

// 几个操作常量，向四个方向移动、退出和错误按键
const (
	UP Key = iota
	DOWN
	LEFT
	RIGHT
	QUIT
	ERROR_KEY
)

func (b *board) TakeInput() bool {
	key, err := GetKeyStrokes()
	if err != nil {
		fmt.Printf(err.Error())
	}
	if key == ERROR_KEY {
		b.TakeInput()
	}
	switch key {
	case UP:
		b.moveUp()
	case DOWN:
		b.moveDown()
	case LEFT:
		b.moveLeft()
	case RIGHT:
		b.moveRight()
	case QUIT:
		fmt.Println("You press ESC, game exit!")
		return false
	}
	return true
}

func GetKeyStrokes() (Key, error) {
	char, key, err := keyboard.GetSingleKey()
	if err != nil {
		return ERROR_KEY, err
	}
	//fmt.Printf("You pressed: %c, key %X\r\n", char, key)
	if int(char) == 0 {
		switch key {
		case keyboard.KeyArrowUp:
			return UP, nil
		case keyboard.KeyArrowDown:
			return DOWN, nil
		case keyboard.KeyArrowLeft:
			return LEFT, nil
		case keyboard.KeyArrowRight:
			return RIGHT, nil
		case keyboard.KeyEsc:
			return QUIT, nil
		default:
			return ERROR_KEY, errors.New("Invalid key, please press again!")
		}
	} else {
		switch char {
		case 119:
			return UP, nil
		case 97:
			return LEFT, nil
		case 115:
			return DOWN, nil
		case 100:
			return RIGHT, nil
		default:
			return ERROR_KEY, errors.New("Invalid key, please press again!")
		}
	}
}
```

游戏退出有两种情况，一个是上面程序中定义的 QUIT 操作，用于游戏过程中主动输入 ESC 按键退出，另一个是游戏面板 16 个数字已满，Game over，通过添加以下函数实现

```go
// game/board.go
func (b *board) IsOver() bool {
	blank := 0
	for i := 0; i < _rows; i++ {
		for j := 0; j < _cols; j++ {
			if b.board[i][j] == 0 {
				blank++
			}
		}
	}
	return blank == 0
}
```

最后是程序开始的逻辑，即输入任意键开始。这部分逻辑在 main 函数中

```go
// main.go
func main() {
	fmt.Println("Use {W A S D} or Arrow keys to move the board")
	fmt.Printf("Press and key to start\n")
	_, _, err := keyboard.GetSingleKey()
	if err != nil {
		log.Fatalln("error while taking input to start the game")
	}
	b := game.NewBoard()
	b.AddElement()
	b.AddElement()
	for true {
		if b.IsOver() {
			break
		}
		b.AddElement()
		b.Display()
		res := b.TakeInput()
		if !res {
			return
		}

	}
	fmt.Println("game over")
}
```

## 6. 数字移动合并

每个键盘输入都对应一个操作函数，四个方向的数字移动和合并是游戏的核心逻辑。如下，向左移动数字和合并单独实现，向右、向上和向下都能通过矩阵旋转转换为向左移动和合并的问题。

```go
// game/board.go
func (b *board) moveLeft() {
	for i := 0; i < _rows; i++ {
		old := b.board[i]
		b.board[i] = moveRow(old)
	}
}

func (b *board) moveRight() {
	b.Reverse()
	b.moveLeft()
	b.Reverse()
}

func (b *board) moveUp() {
	b.leftRotate90()
	b.moveLeft()
	b.rightRotate90()
}

func (b *board) moveDown() {
	b.rightRotate90()
	b.moveLeft()
	b.leftRotate90()
}

func (b *board) rightRotate90() {
	matrix := make([][]int, _rows)
	for i := 0; i < _rows; i++ {
		matrix[i] = make([]int, _cols)
	}
	for i := 0; i < _rows; i++ {
		for j := 0; j < _cols; j++ {
			matrix[j][_cols-1-i] = b.board[i][j]
		}
	}
	b.board = matrix
}

func (b *board) leftRotate90() {
	matrix := make([][]int, _rows)
	for i := 0; i < _rows; i++ {
		matrix[i] = make([]int, _cols)
	}
	for i := 0; i < _rows; i++ {
		for j := 0; j < _cols; j++ {
			matrix[_cols-1-j][i] = b.board[i][j]
		}
	}
	b.board = matrix
}

func (b *board) Reverse() {
	for i := 0; i < _rows; i++ {
		for j, k := 0, _cols-1; j < k; j, k = j+1, k-1 {
			b.board[i][j], b.board[i][k] = b.board[i][k], b.board[i][j]
		}
	}
}

func moveRow(row []int) []int {
	index := 0
	for i := 0; i < len(row); i++ {
		if row[i] != 0 {
			row[index], row[i] = row[i], row[index]
			index++
		}
	}
	for i := 0; i < len(row)-1; i++ {
		if row[i] == row[i+1] {
			row[i] += row[i+1]
			row[i+1] = 0
			i++
		}
	}
	index = 0
	for i := 0; i < len(row); i++ {
		if row[i] != 0 {
			row[index], row[i] = row[i], row[index]
			index++
		}
	}
	return row
}
```

由于这部分逻辑比较复杂，需要测试一下

```go
// game/board_test.go
package game

import (
	"reflect"
	"testing"
)

func TestMoveRow(t *testing.T) {
	tests := []struct {
		name  string
		input []int
		want  []int
	}{
		{
			name:  "one",
			input: []int{2, 2, 0, 0},
			want:  []int{4, 0, 0, 0},
		},
		{
			name:  "two",
			input: []int{2, 2, 4, 8},
			want:  []int{4, 4, 8, 0},
		},
		{
			name:  "three",
			input: []int{2, 4, 4, 8},
			want:  []int{2, 8, 8, 0},
		},
		{
			name:  "four",
			input: []int{2, 4, 8, 8},
			want:  []int{2, 4, 16, 0},
		},
		{
			name:  "five",
			input: []int{2, 2, 2, 2},
			want:  []int{4, 4, 0, 0},
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := moveRow(tt.input); !reflect.DeepEqual(got, tt.want) {
				t.Errorf("moveRow() = %v, want %v", got, tt.want)
			}
		})
	}
}

func TestLeftRotate90(t *testing.T) {
	type fields struct {
		matrix [][]int
	}

	tests := []struct {
		name   string
		fields fields
		want   [][]int
	}{
		{
			name: "one",
			fields: fields{
				matrix: [][]int{
					{1, 2, 3, 9},
					{4, 5, 6, 10},
					{6, 7, 8, 11},
					{16, 17, 18, 111},
				},
			},
			want: [][]int{
				{9, 10, 11, 111},
				{3, 6, 8, 18},
				{2, 5, 7, 17},
				{1, 4, 6, 16},
			},
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			b := &board{board: tt.fields.matrix}
			b.leftRotate90()
			if !reflect.DeepEqual(b.board, tt.want) {
				t.Errorf("b.leftRotate90() = %v, want %v", b.board, tt.want)
			}
		})
	}
}

func TestRightRotate90(t *testing.T) {
	type fields struct {
		matrix [][]int
	}

	tests := []struct {
		name   string
		fields fields
		want   [][]int
	}{
		{
			name: "one",
			fields: fields{
				matrix: [][]int{
					{1, 2, 3, 9},
					{4, 5, 6, 10},
					{6, 7, 8, 11},
					{16, 17, 18, 111},
				},
			},
			want: [][]int{
				{16, 6, 4, 1},
				{17, 7, 5, 2},
				{18, 8, 6, 3},
				{111, 11, 10, 9},
			},
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			b := &board{board: tt.fields.matrix}
			if b.rightRotate90(); !reflect.DeepEqual(b.board, tt.want) {
				t.Errorf("b.rightRotate90() = %v, want %v", b.board, tt.want)
			}
		})
	}
}
```

## 7. 分数计算

游戏正常结束后显示当前最大分数和总分数，算是一个小功能。

```go
//game/board.go
func (b *board) CountScore() (int, int) {
	total, max := 0, 0
	matrix := b.board
	for i := 0; i < _rows; i++ {
		for j := 0; j < _cols; j++ {
			total += matrix[i][j]
			max = maxInts(max, matrix[i][j])
		}
	}
	return max, total
}

func maxInts(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```

```go
//main.go
func main() {
	fmt.Println("Use {W A S D} or Arrow keys to move the board")
	fmt.Printf("Press and key to start\n")
	_, _, err := keyboard.GetSingleKey()
	if err != nil {
		log.Fatalln("error while taking input to start the game")
	}
	b := game.NewBoard()
	b.AddElement()
	b.AddElement()
	for true {
		if b.IsOver() {
			break
		}
		b.AddElement()
		b.Display()
		res := b.TakeInput()
		if !res {
			return
		}

	}
	fmt.Println("\n**********  game over  **********")
	max, total := b.CountScore()
	fmt.Printf("Max Score: %v \n", max)
	fmt.Printf("Total Score %v \n", total)
}
```

## 8. 代码重构

所有功能完成后，对代码进行重构整理，包括

1. 将数字移动合并的相关代码移动到单独的 `move.go` 源文件中；
2. （可选）将输入和显示的相关函数都拆分到单独的源文件中；
3. 为所有代码添加注释，并编写 README.md 文档；

## 9. 结果与收获

完整的项目代码可以查看我的 [github 仓库](https://github.com/shuzang/2048)，与原项目有一些实现上的区别，并完善了一些功能。

![2048](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20201008_2048.gif)

编写该项目的收获有

1. 开源协议的选择；
2. 对结构体和接口作用和意义的思考；
3. 一个项目是从简单到复杂一步步建立的，不要想一步登天做的很完善；
4. 方法中 (b *board) 和 (b board) 的区别；
5. 一些重要的可定制的参数可以抽取作为为常量，比如面板规格，常量命名时前面最好加下划线 `_` 加以区分；
6. fmt.Printf 可以输出固定长度的空字符用于占位，fmt.Println() 可以用来换行；
7. 随机数的生成方法，一个小技巧是使用数组存放待选择数字，然后随机生成数组长度范围内的数字作为索引进行选择；
8. 结构体对象的生成可以使用工厂模式，比如  NewBoard 函数；
9. 格式化输出的颜色控制（fatih/color包）；
10. 清屏的实现方法；
11. 无需回车不断读取键盘输入的实现方法（eiannone/keyboard包）;
12. 矩阵旋转等大量关于二维切片的算法实现（PS：刷题还是有用的）；
13. 测试用例的编写；
14. 所有功能完成后，根据情况进行重构，比如代码的解耦等，然后完成添加注释、编写文档等工作；
15. Go 文档的编写与使用；
16. 日志系统的使用。
