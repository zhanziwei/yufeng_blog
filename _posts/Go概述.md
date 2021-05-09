## Go入门语法

Go语言是谷歌2009发布的第二款开源编程语言，它意在使人们能够方便的构建简单、可靠、高效的软件。

Go语言借鉴了Unix的设计哲学，汲取了C语言的优势，针对多处理器系统应用程序编程进行了专门优化，使用Go编译的程序可以媲美C或C++代码的速度，而且支持并行进程，还更加安全。Go语言有望成为互联网时代的C语言。

#### Go的优势

Go简洁、高效，同时又适用互联网时代的编程需求

**主要语言特性**

- 自动垃圾回收
- 更丰富的内置类型
- 函数多返回值
- 错误处理
- 匿名函数和闭包
- 类型和接口
- 并发编程
- 反射
- 语言交互性

#### 包、变量和函数

```go
package main
import (
	"fmt"
    "math/rand"
)    

func main(){
    fmt.println("我最喜欢的数字是", rand.Intn(10))
}

func add(x int, y int) int {
    return x + y
}

func add(x, y int) int {
    return x + y
}

// 多值返回
func swap(x, y string) (string, string) {
    return y, x
}
func main() {
    a, b:=swap("hello", "world")
    fmt.Println(a, b)
}

// 命名返回值
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

// 变量
var c, python bool
// 短变量声明，只能在func内适用
k := 3

// 基本类型
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名

rune // int32 的别名
    // 表示一个 Unicode 码点

float32 float64

complex64 complex128

// 类型转换
f := float64(i)

// 常量，不能适用:=声明
const Pi = 3.14
```

没有明确初始值的变量声明会被赋予它们的 **零值**。

零值是：

- 数值类型为 `0`，
- 布尔类型为 `false`，
- 字符串为 `""`（空字符串）。

#### 流程控制语句

```go
for i:=0; i < 10; i++ {
    if x < 0 {
        
    }
}
// while
for sum<1000 {
    sum += sum
}

switch os := runtime.GOOS; os {
	case "darwin":
    case "linux":
    default:
    
}

// 推迟执行,defer是一个栈结构，先进去的语句，后执行
func main () {
    defer fmt.Println("world")
    fmt.Println("hello")
}
```

#### 更多类型

```go
// 指针
var p *int
i:=42
p = &i
*p = 21

// 结构体
type Vertex struct {
    X, Y int
}

func main() {
    v := Vertex{1, 2}
    v.X = 4
    p := &v
    p.X = le9
    p = &Vertex{1, 2}
    fmt.Println(Vertex{1, 2})
}

// 数组
var a [2]string
primes := [6]int{2,3,5,7,9,11}
var s []int = primes[1:6]
// 切片和python的不同，更改切片的元素会修改其底层数组中对应的元素
// 切片文法
q := []int{2, 3, 5, 7, 11, 13}
// 切片的长度就是它所包含的元素个数。
len(q)
// 切片的容量是从它的第一个元素开始数，到其底层数组元素末尾的个数
cap(q)

// nil 切片的长度和容量为 0 且没有底层数组。
var s []int
// s.append元素
s = append(s, 1)

// Range
var pow = []int{1,2,3,4,5,6,7}
func main(){
    for i, v:=range pow {
        fmt.Printf("2**%d = %d\n", i, v)
    }
    for i, _:=range pow {}
}

// map
type Vertex struct {
    Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}
// 省略类型名
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}

func main(){
    m = make(map[String] Vertex)
    
    m["Bell Labs"] = Vertex{40.7,54.43,}
    gg := make(map[string]int)
    gg["ans"] = 42
    delete(m, "ans")
    // 若ans存在于m中，v为值，若不在，v为零值，且ok为false
    v, ok:=m["ans"]
    fmt.Println(m["Bell Labs"])
}

// 函数也是值
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}

// 闭包是一个函数值，引用了其函数体之外的变量。该函数可以访问并赋予其引用的变量的值
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

#### 方法和接口

Go没有类，可以为结构体类型定义方法。方法为一类带特殊的接收者参数的函数

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}

// 接收者的类型定义和方法声明必须在同一包内；不能为内建类型声明方法。

// 可以为指针接收者声明方法,指针接收者的方法可以修改接收者指向的值
func (v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func main(){
    v := Vertex{3,4}
    v.Scale(10)
    fmt.Println(v.Abs())
}

// 以指针为接收者的方法被调用时，接收者既能为值又能为指针
var v Vertex
func(v *Vertex) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

p:=&v
v.scale(10)
p.Scale(10)
// Go语言会把p翻译为&v

// 所有给定类型的方法都应该有值或指针接收者，而不应该两者混用
```

#### 接口

接口类型是由一组方法签名定义的集合

```go
type Abser interface {
    Abs() float64
}

func main() {
    var a Abser
    f:=MyFloat(-math.Sqrt2)
    v:=Vertex{3,4}
    a = f
    a = &v
    // 由于下面一行，v是一个Vertex，没有实现Abser
    a = v
    fmt.Println(a.Abs())
}
type MyFloat float64
func(f MyFloat) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}

type Vertex struct {
    X, Y float64
}
func (v *Vertex) Abs() float64 {
    return math.Sqrt(v.X*v,X+v,v*Y*v*Y)
}

// 类型通过实现一个接口的所有方法来实现该接口，不需要显式声明implements

type I interface {
	M()
}

type T struct {
	S string
}

// 此方法表示类型 T 实现了接口 I，但我们无需显式声明此事。
func (t T) M() {
	fmt.Println(t.S)
}

// 接口也是值，接口值可以用作函数的参数或返回值。
```



