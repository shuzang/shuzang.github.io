# 螺旋矩阵问题


螺旋矩阵是指一个呈螺旋状的矩阵，它的数字由第一行开始到右边不断变大，向下变大，向左变大，向上变大，如此循环。给定m和n，返回一个大小为m*n的螺旋矩阵

**示例：**

```in
input:
3 3
Output:
1 2 3
8 9 4
7 6 5
```

**思路：**

1. 每一步走完，判断是否超出矩阵范围，若超出，计算向哪个方向转，无论是否转向，计算下一步所在位置的x,y坐标
2. 螺旋矩阵的转向顺序是固定的，为`右->下->左->上`

**代码：**

```go
func spiralArray(m, n int) {
	var result [][]int
	for i := 0; i < m; i++ {
		result = append(result, make([]int, n))
	}
    //x,y为当前坐标，d为方向
	x, y, d := 0, 0, 0
    //以“右->下->左->上”顺序循环，dx,dy是每一种转向的坐标变化方式
	dx := []int{0, 1, 0, -1}
	dy := []int{1, 0, -1, 0}
	for t := 0; t < m*n; t++ {
		result[x][y] = t
		nx, ny := x+dx[d], y+dy[d] //计算下一个坐标
		if nx < 0 || nx >= m || ny < 0 || ny >= n || result[nx][ny] != 0 {
			d = (d + 1) % 4 //下一个坐标有问题，采用下一种转向方式
			nx = x + dx[d]
			ny = y + dy[d]
		}
		x = nx
		y = ny
	}
	w := bufio.NewWriter(os.Stdout)
	for i := 0; i < m; i++ {
		for j := 0; j < n; j++ {
			w.WriteString(fmt.Sprintf("%d", result[i][j]))
			if j != n-1 {
				w.WriteString(fmt.Sprintf(" "))
			}
		}
		w.WriteString(fmt.Sprintf("\n"))
	}
	w.Flush()
}
```




