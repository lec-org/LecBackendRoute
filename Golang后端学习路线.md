# Golang后端学习路线

### 学习资料：

go语言圣经：https://books.studygolang.com/gopl-zh/?from=from_parent_mindnote

go标准库：http://books.studygolang.com/The-Golang-Standard-Library-by-Example/?from=from_parent_mindnote.0

gorm: https://gorm.io/zh_CN/docs/query.html

cloudWeGo 是字节跳动内部Golang框架开发团队。 可在github上直接搜索cloudwego

CloudWeGO: https://www.cloudwego.io/zh/

## golang 基础

```go
nil 是 null的意思
```

```go
package main
import "fmt"
func main() {
	fmt.Printf("Hello World")
}
//输出hello world
```

### new是返回的指针 



### 变量类型：

```go
var a int = 12
fmt.Println(a)

var b float64 = 123.132
fmt.Println(b)

// 可以直接使用var来声明变量 会根据上下文来自动匹配类型
```

### if else


```go
if b > 1000 {
		fmt.Println(b)
	} else {
		fmt.Println(b + 1)
	}
//if后面不需要紧跟大括号，else需要和末尾的大括号在同一行
```

### for


```go
for i := 1; i <= 10; i++ {
		fmt.Println(i)
}
```

### 数组的声明：

```go
var a [10][10]int

```

### 切片

```go
a := make([]string, 0)  //创建切片
a = append(a, "1")    // 添加元素
a = append(a, "2")
a = append(a, "3")
a = append(a, "4")
fmt.Println(a)
```


```go
b := make([]string, len(a))
copy(b, a)   //将a的元素复制到b中去
fmt.Println(b)
```


```go
//高级遍历
fmt.Println(b[1:3])
fmt.Println(b[:5])
fmt.Println(b[0:])
```

### 二维切片

```go
	a := make([][]int, 6)    // 先声明一个二维切片
	for i := 0; i < len(a); i++ {
		a[i] = make([]int, 6)  // 通过第一维来声明第二维
	}
	for i := 0; i < len(a); i++ {
		for j := 0; j < len(a[i]); j++ {
			a[i][j] = i * j
		}
	}
	for _, i := range a {  //  for range 枚举
		for _, j := range i {
			fmt.Println(j)
		}
	}
```

加长切片

```go
a := make([]int, 0, 10)
a = a[0 : len(a)+2]
a[1] = 1
```



### MAP ##

map是一个无序的结构

遍历输出的结构和插入的结构是不同的

```go
a := make(map[string]int)
a["123"] = 1
a["234"] = 2
fmt.Println(len(a))
fmt.Println(a)
fmt.Println(a["123"])
fmt.Println(a["asdfgas"])
c, d := a["123"]   // c获取这个键值对的key，d为一个是否存在的变量，存在为1不存在为0
fmt.Println(c, d)
delete(a, "123")
fmt.Println(a)
```

使用   for range  遍历map

```go
a := make(map[string]int) // 创建一个map
a["123"] = 1
a["234"] = 2
for index, val := range a {  // 第一个参数为下标，第二个参数为val
	fmt.Println(index, val)
}
```

map切片并且排序

```go
a := make([]map[int]int, 3)   // 创建一个map的切片
for i, _ := range a {
	a[i] = make(map[int]int, 20)   //  定义第二维
	for j := 1; j <= 10; j++ {
		a[i][j] = j     // 进行赋值
	}
}
for i, _ := range a {
	var ans []int
	for j, _ := range a[i] {
		ans = append(ans, j)   // 要先把keys给取出来，然后对keys排序，然后再输出
	}
	sort.Ints(ans)
	for _, j := range ans {
		fmt.Println(a[i][j])  // 在排序结束之后进行输出
	}
}
```

 交换键值对

```go
	a := make(map[int]string)
	a[1] = "10"
	a[2] = "20"
	a[3] = "30"
	b := make(map[string]int, len(a))   // 定义一个与a的类型相反的map，然后用循环实现互换
	for i, j := range a {
		b[j] = i
	}
	fmt.Println(b)
```



### 指针

```go
func add(a *int) {  // 定义参数的时候需要加上*
	*a++            // 在使用参数的时候也需要加上*
}
func main() {   //  
	a := 1
	add(&a)      // 在调用函数的时候需要加上&
	fmt.Println(a)
}

```

将数组的地址传给函数来减少数组的复制消耗的时间

```go
func add(a *[3]int) (sum int) {
	for i := 0; i < len(a); i++ {
		sum += a[i]
	}
	return
}
func main() {
	a := [3]int{}
	a[0] = 1
	a[1] = 2
	a[2] = 3
	fmt.Println(add(&a))
}
```

利用切片向函数传参数

```go
func sum(a []int) (sum int) {
	for _ , i := range a {
		sum += i
	}
	return
}
func main() {
	a := make([]int, 0)   // 定义切片
	a = append(a, 1)
	a = append(a, 2)
	a = append(a, 3)
	fmt.Println(sum(a))
}
```



### 定义成员函数 

注意，在Golang的函数中，如果首字母是小写，则只能在包内使用；如果首字母是大写，则可以在包外被引入使用。可以理解为，使用小写的函数，是private的，使用大写的函数，是public的。

```go
func (a qaq) check(b string) (c int) {   // 在函数名前加上变量名和结构体类型  
	if a.b == b {
		c = 1
		return c
	} else {
		c = 0
		return c
	}
}
func (a *qaq) set(b string) {   // 指针调用同理
	a.b = b
}
func main() {
	a := qaq{1, "2"}  
	b := qaq{2, "3"}
	a.set("4")
	fmt.Println(b.check("3"))
}

```

### 错误处理

```go
func faq(b int) (c *int, err error) {    // 返回类型需要是指针类型的才能返回异常
	if b != 1 {
		return &b, nil
	} else {
		return nil, errors.New("gg")
	}
}
```

### 字符串

```go
	a := "12346"
	fmt.Println(strings.Contains(a, "1"))    // 判断是否存在这个字符串是否存在
	fmt.Println(strings.Count(a, "2"))     	 // 统计某个字符串出现的次数
	fmt.Println(strings.HasPrefix(a, "12"))  // 判断时候以没某个字符串为开头
	fmt.Println(strings.HasSuffix(a, "46"))  // 判断是否以某个字符串结尾
	fmt.Println(strings.Index(a, "12"))      // 返回某个字符串的下标
	fmt.Println(strings.Join([]string{"he", "llo"}, "-"))  // 在两个字符串之间加上字符串
	fmt.Println(strings.Repeat(a, 2))     // 重复某个字符串几次
	fmt.Println(strings.Replace(a, "12", "11", -1))   // 将第一个字符串替换成第二个字符串，从第四个索引开始
	fmt.Println(strings.Split("a-b-c", "-"))   // 将字符串之间的相应字符去除
	fmt.Println(strings.ToLower(a))   //大写
	fmt.Println(strings.ToUpper(a))	  // 小写
```

### 字符串格式化

```go
fmt.Printf("%v", a)
{1 1 23}

fmt.Printf("%+v", a)
{a:1 b:1 c:23}

fmt.Printf("%#v", a)
main.qaq{a:1, b:1, c:"23"}

```

### json

```go
	a := qaq{1, 1, "23"}
	buf, err := json.Marshal(a)  // 结构体中的每一个变量名必须都是大写开头
	if err != nil {
		panic(err)
		return
	}
	fmt.Println(string(buf)) // 如果不按照string打印的会会出现16进制的数字
	var b qaq
	err = json.Unmarshal(buf, &b)   // 将buf反json化到b中去，这里的参数需要传入地址
	if err != nil {
		panic(err)
		return
	}
	fmt.Println(b)
```

json.NewDecoder用于http连接与socket连接的读取与写入，或者文件读取；
json.Unmarshal用于直接是byte的输入。

```go
```



### 数字与字符串的相互转化

```go
	f, _ := strconv.ParseFloat("12.12", 64)  // 字符串化为int的时候不需要指明进制
	fmt.Println(f)
	n, _ := strconv.ParseInt("1234", 10, 64)   // 整数在转化的时候需要指明进制
	fmt.Println(n)
	f1, _ := strconv.ParseInt("0x1111", 0, 64) // 也可以实现16进制的转化
	fmt.Println(f1)
	n2, _ := strconv.Atoi("123")  // 可以直接转化成10进制
	fmt.Println(n2)
```

### 并发

```go
func prin(i int) {
	fmt.Println("qaq: ", i)
}
func main() {
	for i := 0; i < 5; i++ {
		go func(j int) {  // 
			prin(j)
		}(i)
	}
	time.Sleep(time.Second)
}
```

并发

```go
 // 带缓存的队列可以放置生产和消费速度不一致带来的问题
	src := make(chan int)    // 创建一个无缓冲的线程
	dust := make(chan int, 3)  // 创建一个三个缓存单位的线程
	go func() {   // 开启第一个线程
		defer close(src)
		for i := 0; i < 10; i++ {
			src <- i   // 向src中加入元素
		}
	}()
	go func() {
		defer close(dust)
		for i := range src {
			dust <- i * i   // 将src的元素加入到dust中
		}
	}()
	for i := range dust {
		fmt.Println(i)    // 输出dust
	}
```

并发安全问题

```go
var (
	x    int64
	lock sync.Mutex
)
func addlock() {
	for i := 0; i < 2000; i++ {
		lock.Lock()
		x += 1
		lock.Unlock()
	}
}
func addwithoutlock() {
	for i := 0; i < 2000; i++ {
		x += 1
	}
}
func main() {
	x = 0
	for i := 0; i < 5; i++ {
		go addwithoutlock()   // 使用线程锁
	}
	time.Sleep(time.Second)
	fmt.Println("lock :", x) 
	x = 0
	for i := 0; i < 5; i++ {
		go addlock()         // 不适用线程锁
	}
	time.Sleep(time.Second)
	fmt.Println("unlock: ", x)
}

lock : 9570
unlock:  10000

```

### waitgroup

```go
func main() {
	var wg sync.WaitGroup   // 创建一个waitgroup
	wg.Add(5)   // 添加任务
	for i := 0; i < 5; i++ {
		go func(j int) {
			defer wg.Done()   // 完成任务
			fmt.Println(j)
		}(i)
	}
	wg.Wait()  // 任务结束
}
```

### 测试

```go
//编写测试代码
文件名：_test.go结尾
测试主函数：func TestMain(t *testing.T){}


func helloTom() string {
	return "Jerry"
}
func TestTom(t *testing.T) {    //测试函数必须以Test开头
	res := helloTom()
	exp := "Tom"
	if res != exp {
		t.Errorf("")
	}
}
```

 ### 通过IO来实现输入 ##

```go
func ReadFrom(reader io.Reader, num int) ([]byte, error) {
	p := make([]byte, num)
	n, err := reader.Read(p)
	if n > 0 {
		return p[:n], nil
	}
	return p, err
}
func main() {
	data, err := ReadFrom(os.Stdin, 11)
	if err != nil {
		errors.New("gg")
		return
	}
	fmt.Println(string(data))
}
//通过实现这个接口，可以实现任意形式的输入

```

### 匿名函数

```go
	a := func(c, d int) int {    // 将一个匿名函数定义到一个变量中
		return c + d
	}
	b := a     // 这个函数也可以赋值给其他函数
	e := b(1, 2)   // 调用函数
	fmt.Println(e)
```

### 返回值为函数

```go
func add() func(a int) int {
	return func(a int) int {
		return a + 2
	}
}
func add1(a int) func(b int) int {
	return func(b int) int {
		return a + b
	}
}
func main() {
	c := add()    // 函数装到变量中,但是这个函数中嵌套了函数，所以这个c也是函数
	fmt.Println(c(3))   // 传入c的值，返回内层函数
	b := add1(3)   // 同上可得，b也是一个函数
	fmt.Println(b(2))  // b函数
}

```

### 函数返回函数的高级用法

```go
func add() func(a int) int {
	x := 0
	return func(a int) int {
		x += a
		return x
	}
}
func main() {
	a := add()    // 将a设为函数
	fmt.Println(a(1))
	fmt.Println(a(20))
	b := a   // 将a赋值给b之后，b会继承a的所有字段
	fmt.Println(b(100)) 
}
//
1
21
121

```

### 修改字符串 ##

string不允许更改

```go
a := "1234"
b := []byte(a)   // string类型不允许更改，需要新定义一个变量
b[0] = '5'   // 修改b
fmt.Println(string(b))
```

### 线程安全： ##

map是非线程安全的，因为map不存在线程锁，当多个进程同时访问map的时候，就可能出现map数据错误的情况

`sync.Mutex` 是一个互斥锁，它的作用是守护在临界区入口来确保同一时间只能有一个线程进入临界区。

```go
type qaq struct {
	mu sync.Mutex    // 定义互斥锁
	a  string
}

func de(qaq *qaq) {
	qaq.mu.Lock()  // 在使用变量的时候给变量上锁，防止同时被很多线程访问
	qaq.a = "123"
	qaq.mu.Unlock()  // 使用结束之后，解锁
}

func main() {
	a := qaq{}
	a.a = "aaa"
	de(&a)
	fmt.Println(a.a)
}
```

### 反射

```go
type qaq struct {
	a int    "123456"
	b string "abcdef"
	c int    "11111"
}

func main() {
	a := qaq{1, "12", 2}
	fmt.Println(reflect.TypeOf(a).Field(0).Tag)
	fmt.Println(reflect.TypeOf(a).Field(1).Tag)
    fmt.Println(reflect.TypeOf(a).Field(2).Tag) // 通过调用 reflect.Typeof().Field().Tag来获取备注   
    
	fmt.Println(reflect.TypeOf(a).Field(0).Type)
	fmt.Println(reflect.TypeOf(a).Field(1).Type)
	fmt.Println(reflect.TypeOf(a).Field(2).Type)
}
```

### 嵌套结构体

```go
type qaq struct {
	a int    "123456"
	b string "abcdef"
	c int    "11111"
}
type faq struct {
	a int
	qaq
}

func main() {
	a := faq{}
	a.a = 1
	a.qaq.b = "123"
	fmt.Println(a)
}
 
```

### 接口

定义接口

```go
type qaq interface {
    Method (List)  return_type
}
```

实现接口

```go
type faq interface {   // 创建一个接口
	run() int
}
type qaq struct {   // 定义一个实现接口的结构体
	a int
	b int
}

func (qaq *qaq) run() int {   // 实现接口的方法
	return qaq.a * qaq.b
}

func main() {
	a := new(qaq)  // 实例化结构体
	a.b = 1   
	a.a = 2
	var b faq  // 实例化接口
	b = a  // 赋值
	fmt.Println(b.run())  // 调用接口
}
```

接口的多态

```go

type faq interface {  // 定义接口
	area() int
}
type qaq struct {
	a, b int
}
type fa struct {
	a int
}
func (qaq *qaq) area() int {  // 使用qaq结构体实现接口
	return qaq.a * qaq.b
}
func (fa *fa) area() int {  // 使用fa结构体实现接口
	return fa.a * fa.a
}
func main() {  
	a := qaq{1, 2}  
	b := fa{1}
	ans := []faq{&a, &b}  // 定义接口的数组
	for _, val := range ans {
		fmt.Println(val.area())  // 依次调用接口的实现方法
	}

}
```

检测接口的类型

```go
type qaq interface {   // 定义接口
	faq(a int) int
}
type faq struct {  // 定义结构体
	a, b int
}

func (f faq) faq(a int) int {
	return f.b * a
}
func main() {
	c := new(faq)
	c.b = 1
	c.a = 2
	var qaq qaq
	qaq = c
	if v, ok := qaq.(*faq); ok { // 判断这个接口是什么类型的
		fmt.Printf("%T", v)
	}
}

输出：*main.faq


```

空接口，可以为任意类型的值

### 内联函数 

```go
qaq := func(a, b int) int {
	return a + b
}
fmt.Println(qaq(1, 2))

func (参数列别)(返回值列表) {

}
```

用于很少次数使用的较短的函数

### 反射

1.通过反射类获取类型

```go
	var x float64 = 3.4
	fmt.Println("type:", reflect.TypeOf(x))   // 获取反射的类型
	v := reflect.ValueOf(x)    // 实现接口
	fmt.Println("value:", v) 
	fmt.Println("type:", v.Type()) // 打印类型  
	fmt.Println("kind:", v.Kind()) // 打印类型 
	fmt.Println("value:", v.Float()) // 打印值
	fmt.Println(v.Interface()) // 打印值
	fmt.Printf("value is %5.2e\n", v.Interface())
	y := v.Interface().(float64)
	fmt.Println(y)
```

2.通过反射类修改值

```go
	a := 1
	fmt.Println(reflect.TypeOf(a))
	b := reflect.ValueOf(&a)   // 获取反射
	b.Elem().SetInt(123)  // 调用elem来修改值
	fmt.Println(b.Interface())  // 打印b的地址
	fmt.Println(a) // 打印a的值
	fmt.Println(b.String()) // 打印b的反射类型
	fmt.Println(b.Type()) // 打印b的反射类型

output：

int
0xc000018088
123
<*int Value>
*int

```

反射结构：

有些时候需要反射一个结构类型。`NumField()` 方法返回结构内的字段数量；通过一个 for 循环用索引取得每个字段的值 `Field(i)`。

我们同样能够调用签名在结构上的方法，例如，使用索引 n 来调用：`Method(n).Call(nil)`。

```go
type qaq struct {
	a, b, c string
}
func (a qaq) String() string {
	return a.a + "__" + a.b + "__" + a.c
}
func main() {
	c := qaq{"a", "b", "c"}
	val := reflect.ValueOf(c)
	for i := 0; i < val.NumField(); i++ { 
		fmt.Println(val.Field(i))   // 输出每一个字段的值
	}
	qaq := val.Method(0).Call(nil)  // 调用原来结构的函数
	fmt.Println(qaq)
}

```

实现typeof接口来查看每一个变量或者方法的属性

```go
c := qaq{"a", "b", "c"}
	d := reflect.TypeOf(c)
	for i := 0; i < d.NumField(); i++ {
		fmt.Println(d.Field(i).Type)
		fmt.Println(d.Field(i).Tag)
	}
	fmt.Println(d.Method(0).Type)
```

如果需要更改某个变量的值,那么需要该变量的首字母为大写,因为大写为可导出属性

```go
type qaq struct {
	A, b, c string `13456`
}
e := reflect.ValueOf(&c).Elem()
e.Field(0).SetString("123")
```

Go 中的接口跟 Java/C# 类似：都是必须提供一个指定方法集的实现。但是更加灵活通用：任何提供了接口方法实现代码的类型都隐式地实现了该接口，而不用显式地声明。

和其它语言相比，Go 是唯一结合了接口值，静态类型检查（是否该类型实现了某个接口），运行时动态转换的语言，并且不需要显式地声明类型是否满足某个接口。该特性允许我们在不改变已有的代码的情况下定义和使用新接口。

```go
type faq interface {
	walk()
	run()
}
type qaq struct {
	a, b int
}

func abc(a faq) {
	a.walk()
	a.run()
}
func (a *qaq) walk() {
	a.a = 50
}
func (a *qaq) run() {
	a.b = 10
}
func main() {
	c := qaq{1, 2}
	abc(&c)
	fmt.Println(c.a)
	fmt.Println(c.b)
}
```

### 输入

```go
var a, b int
fmt.Scanln(&a, &b)
```

```go
reader := bufio.NewReader(os.Stdin) //  这行代码，将会创建一个读取器，并将其与标准输入绑定。
input, errr := reader.ReadString('\n')  // 通过这个指针来读取数据	
if errr != nil {}
fmt.Println(input)
```

返回的读取器对象提供一个方法 `ReadString(delim byte)`，该方法从输入中读取内容，直到碰到 `delim` 指定的字符，然后将读取到的内容连同 `delim` 字符一起放到缓冲区。

#### 从文件输入

```go
func main() {
	inputFile, err := os.Open("test.txt")   // 打开文件
	if err != nil {
		panic(err)
	}
    
	defer inputFile.Close()    // 延迟关闭文件
	inputReader := bufio.NewReader(inputFile)   // 创建一个读文件的指针
    
	for {
		inputString, err := inputReader.ReadString('\n')  // 读文件
		if err == io.EOF {
			return
		}
		fmt.Printf(inputString)  // 输出
	}
}

```

文件复制 将test1的文件复制到test2

```go
inputFileTest, err := os.Open("test1.txt")
	if err != nil {
		panic(err)
	}
	defer inputFileTest.Close()
	inputFileTest1, err := os.OpenFile("test2.txt", os.O_WRONLY|os.O_CREATE, 0644)  // 后面的为打开文件的参数
	if err != nil {
		panic(err)
	}
	defer inputFileTest1.Close()
	_, err = io.Copy(inputFileTest1, inputFileTest)
	if err != nil {
		fmt.Println(err)
	}
```

相关的打开文件的参数

```go
os.O_WRONLY | os.O_CREATE | O_EXCL           //如果已经存在，则失败

os.O_WRONLY | os.O_CREATE                 // 如果已经存在，会覆盖写，不会清空原来的文件，而是从头直接覆盖写

os.O_WRONLY | os.O_CREATE | os.O_APPEND  // 如果已经存在，则在尾部添加写
```

## 数据库基础

见java后端

## gorm

github仓库地址：https://github.com/zhamghaoran/gorm_learn.git

gorm： https://gorm.io/zh_CN/docs/query.html 

## gin

现在已被字节抛弃,建议转战Hertz

但是Gin和Hertz在很多地方还是一样的

github：https://github.com/zhamghaoran/Gin_Learn.git

### 快速开始

```go
r := gin.Default() // 返回默认的引擎
r.GET("/hello", func(c *gin.Context) {   // 访问hello这个页面的时候调用的函数指定访问方式为get
	c.JSON(200, gin.H{                     // gin.H 设置返回的json的串
		"message": "hello golang",   // 设置返回值，HTTPCODE：200，同时返回一个json。
	})
})
r.Run(":1234")  // 在1234这个接口上面运行
```

### json

```go
r.GET("/test", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{  // 通过gin.h返回json
			"name": "zhr",
			"age":  "18",
		})
	})
```

```go
r.GET("/test", func(c *gin.Context) {
		var qaq struct {
            Name string `json:"user"`// 通过结构体来构建json格式的数据，但是首字母必须是大写,也可以使用注释
			Age  int
		}
		qaq.Name = "zhr"
		qaq.Age = 18
		c.JSON(http.StatusOK, qaq)
	})
```

### 参数

```go
r.GET("/web", func(c *gin.Context) {
		// 获取浏览器的请求携带的参数
		name := c.Query("query")
		c.JSON(http.StatusOK, name)
	})

访问127.0.0.1:1234/web?query=qaq 就会返回qaq
```

根据url来获取参数

```go
r.GET("/:name/:age", func(c *gin.Context) {  // 获取输入的字段
		name := c.Param("name")
		age := c.Param("age")
		c.JSON(http.StatusOK, gin.H{
			"name": name,
			"age":  age,
		})
	})
```

### 参数绑定

```go
type UserInfo struct {
	Name     string `form:"name" json:"name"`  // 参数绑定
	Password string `form:"password" json:"password"`
}

func main() {
	r := gin.Default()
	r.GET("/user", func(c *gin.Context) {
		var qaq UserInfo
		err := c.ShouldBind(&qaq)  // 传入结构体并且获取参数
		if err != nil {
			fmt.Println("gg" + err.Error())
		} else {
			ans, _ := json.Marshal(qaq)
			fmt.Println(string(ans))
			c.JSON(http.StatusOK, gin.H{
				"name":     qaq.Name,
				"password": qaq.Password,
			})
		}
	})
	r.Run(":1234")
}
```

### 多个文件上传

```go
	import (
	"github.com/gin-gonic/gin"
	"log"
	"net/http"
)

func UploadMultipartFile(c *gin.Engine) {
	c.MaxMultipartMemory = 8 << 20 // 8MB 大小
	c.POST("/upload", func(context *gin.Context) {
		form, err := context.MultipartForm()
		if err != nil {
			context.String(http.StatusInternalServerError, "get err %s", err.Error())
			return
		}
		files := form.File["file"]
		for _, file := range files {
			log.Println(file.Filename)
			dst := "./" + file.Filename   // 设置路径
			context.SaveUploadedFile(file, dst)  // 保存文件
		}
		context.String(http.StatusOK, "upload file %s", len(files))
	})
	c.Run(":8080")
}
```

###  路由组

```go
func initRouter(r *gin.Engine) {
	// public directory is used to serve static resources
	r.Static("/static", "./public")  // 设置静态文件路径

	apiRouter := r.Group("/douyin")  // 设置第一级的url

    // 路由组的第一个参数为url，第二个参数为相应的handler方法
	// basic apis
	apiRouter.GET("/feed/", controller.Feed)   
	apiRouter.GET("/user/", controller.UserInfo)
	apiRouter.POST("/user/register/", controller.Register)
	apiRouter.POST("/user/login/", controller.Login)
	apiRouter.POST("/publish/action/", controller.Publish)
	apiRouter.GET("/publish/list/", controller.Publishlist)
	//
	//// extra apis - I
	apiRouter.POST("/favorite/action/", controller.FavoriteAction)
	apiRouter.GET("/favorite/list/", controller.FavoriteList)
	apiRouter.POST("/comment/action/", controller.CommentAction)
	apiRouter.GET("/comment/list/", controller.CommentList)
	//
	//// extra apis - II
	apiRouter.POST("/relation/action/", controller.RelationAction)
	apiRouter.GET("/relation/follow/list/", controller.FollowList)
	apiRouter.GET("/relation/follower/list/", controller.FollowerList)
}
```



## Hertz

官方文档： https://www.cloudwego.io/zh/docs/hertz/

和gin 框架很像 可以迅速上手

## kitex

Kitex[kaɪt’eks] 字节跳动内部的 Golang 微服务 RPC 框架，具有**高性能**、**强可扩展**的特点，在字节内部已广泛使用。如果对微服务性能有要求，又希望定制扩展融入自己的治理体系，Kitex 会是一个不错的选择。

官方文档：https://www.cloudwego.io/zh/docs/kitex/

尚未学习

