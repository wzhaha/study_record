# go语言学习记录
## 包管理
1. 可运行的包内的必须是package main
2. 一个文件夹内只能有一个

	```
	func main(){}
	```

3. main内的.go文件可以直接引用其他文件内的函数以及变量，但不可以单独运行
4. 引用其他包内的文件内容时，需要import "package name"
5. 当你导入一个包的时候，Go 使用这个包的包声明创建一个变量。如果你用一个名字导入多个包，将会导致冲突，因此需要使用别名

	```
	import{
		"fmt"
		_ "greet"  
		child "greet/greet"   //这里的"greet"与"greet/greet"在go的内部生成的的变量都是greet，因此需要指定别名
	}
	```
	
## 条件语句
1. if else 与c，Java相同
2. switch, 不需要break，**并且可以不设定switch之后的条件表达式**，当加fallthrough时会强制执行下一句

	```
	switch {
		case grade == "A" :
			fmt.Printf("优秀!\n" )
		case grade == "B", grade == "C" :
			fmt.Printf("良好\n" )
		case grade == "D" :
			fmt.Printf("及格\n" )
		case grade == "F":
			fmt.Printf("不及格\n" )
		default:
			fmt.Printf("差\n" );
	}
	```
3. ==**select 不会用**==，需要同chan一起使用，迷

## 循环语句
1. 就是for循环，go语言中没有while循环
2. break与continue同其他语言的用法
3. for循环中的range进行序号

	```
	numbers := [6]int{1, 2, 3, 5} 
	for i,x:= range numbers {
      fmt.Printf("第 %d 位 x 的值 = %d\n", i,x)
   }
	
	```

## 函数
1. example

	```
	<!--可以返回多个值，放在括号里，需要指定传入与返回的值的类型-->
	func swap(x, y string) (string, string) {
   		return y, x
	}
	```
	
2. 注意函数的引用传递与值传递

	```
	<!--传入参数的地址时进行引用传递-->
	func swap(x *int, y *int) {
	   var temp int
	   temp = *x    /* 保存 x 地址上的值 */
	   *x = *y      /* 将 y 值赋给 x */
	   *y = temp    /* 将 temp 值赋给 y */
	}
	```
3. go函数闭包(return a function)

	```
	func getSequence() func() int {
		i:=0
		return func() int {
			i+=1
			return i
		}
	}
	```
	
4. 方法(一个包含了接受者的函数)

	```
	//该 method 属于 Circle 类型对象中的方法
	func (c Circle) getArea() float64 {
	  //c.radius 即为 Circle 类型对象中的属性
	  return 3.14 * c.radius * c.radius
	}
	```

## 变量作用域
没啥可说的：全局变量与局部变量

##数组
1. 声明：

	```
	var variable_name [SIZE] variable_type
	```
	
2. 初始化
	
	```
	指定大小
	var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
	不指定大小，根据个数自动设置
	var balance = [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
	```

3. 多维数组

## 指针
1. 空指针判断

	```
	if(ptr != nil)     /* ptr 不是空指针 */
	if(ptr == nil)    /* ptr 是空指针 */
	```
	
2. 指针数组(数组内保存的是指针)

	```
	package main

	import "fmt"
	
	const MAX int = 3
	
	func main() {
	   a := []int{10,100,200}
	   var i int
	   var ptr [MAX]*int;
	
	   for  i = 0; i < MAX; i++ {
	      ptr[i] = &a[i] /* 整数地址赋值给指针数组 */
	   }
	
	   for  i = 0; i < MAX; i++ {
	      fmt.Printf("a[%d] = %d\n", i,*ptr[i] )
	   }
	}
	```

3. 指向指针的指针（不知道有啥卵用）

## 结构体
1. 定义

	```
	type Books struct {
	   title string
	   author string
	   subject string
	   book_id int
	}
	```
	
2. 新建与使用

	```
	var Book1 Books
	Book1.title = "Go 语言"
   Book1.author = "www.runoob.com"
   Book1.subject = "Go 语言教程"
   Book1.book_id = 6495407
	/* 打印 Book1 信息 */
   fmt.Printf( "Book 1 title : %s\n", Book1.title)
   fmt.Printf( "Book 1 author : %s\n", Book1.author)
   fmt.Printf( "Book 1 subject : %s\n", Book1.subject)
   fmt.Printf( "Book 1 book_id : %d\n", Book1.book_id)
	```
	
3. 作为函数参数

	```
	func printBook( book Books ) {
	   fmt.Printf( "Book title : %s\n", book.title);
	   fmt.Printf( "Book author : %s\n", book.author);
	   fmt.Printf( "Book subject : %s\n", book.subject);
	   fmt.Printf( "Book book_id : %d\n", book.book_id);
	}
	```

4. 匿名字段:  当我们创建结构体时，字段可以**只有类型，而没有字段名**。

	```
	type Person struct {  
	    string
	    int
	}
	
	func main() {  
	    var p1 Person
	    p1.string = "naveen"
	    p1.int = 50
	    fmt.Println(p1)
	}
	```
	
5. 嵌套结构体

	```
	type Address struct {  
	    city, state string
	}
	type Person struct {  
	    name string
	    age int
	    address Address
	}
	
	func main() {  
	    var p Person
	    p.name = "Naveen"
	    p.age = 50
	    p.address = Address {
	        city: "Chicago",
	        state: "Illinois",
	    }
	    fmt.Println("Name:", p.name)
	    fmt.Println("Age:",p.age)
	    fmt.Println("City:",p.address.city)
	    fmt.Println("State:",p.address.state)
	}
	```
## 切片
1. 定义

	```
	使用数组来定义:var identifier []type
	使用make来创建:var slice1 []type = make([]type, len)
	```
2. len()--返回切片的大小 和 cap()--返回切片的容量
3. append() 和 copy() 函数

	```
	var numbers = make([]int,3,5)

	printSlice(numbers)
	numbers = append(numbers,1)
	printSlice(numbers)

	/* 创建切片 numbers1 是之前切片的两倍容量*/
	numbers1 := make([]int, len(numbers), (cap(numbers))*2)

	/* 拷贝 numbers 的内容到 numbers1 */
	copy(numbers1,numbers)
	printSlice(numbers1)
	```
	
## Map集合
1. 定义

	```
	/* 声明变量，默认 map 是 nil */
	var map_variable map[key_data_type]value_data_type
	
	/* 使用 make 函数 */
	map_variable := make(map[key_data_type]value_data_type)
	```
	
2. 使用实例

	```
	var countryCapitalMap map[string]string /*创建集合 */
    countryCapitalMap = make(map[string]string)

    /* map插入key - value对,各个国家对应的首都 */
    countryCapitalMap [ "France" ] = "巴黎"
    countryCapitalMap [ "Italy" ] = "罗马"
    countryCapitalMap [ "Japan" ] = "东京"
    countryCapitalMap [ "India " ] = "新德里"

    /*使用键输出地图值 */ 
    for country := range countryCapitalMap {
        fmt.Println(country, "首都是", countryCapitalMap [country])
    }

    /*查看元素在集合中是否存在 */
    capital, ok := countryCapitalMap [ "American" ] /*如果确定是真实的,则存在,否则不存在 */
    /*fmt.Println(capital) */
    /*fmt.Println(ok) */
    if (ok) {
        fmt.Println("American 的首都是", capital)
    } else {
        fmt.Println("American 的首都不存在")
    }
	```
	
## 接口(interface)
1. example

	```
	package main
	
	import (
	    "fmt"
	)
	
	type Phone interface {
	    call()
	}
	
	type NokiaPhone struct {
	}
	
	func (nokiaPhone NokiaPhone) call() {
	    fmt.Println("I am Nokia, I can call you!")
	}
	
	type IPhone struct {
	}
	
	func (iPhone IPhone) call() {
	    fmt.Println("I am iPhone, I can call you!")
	}
	
	func main() {
	    var phone Phone
	
	    phone = new(NokiaPhone)
	    phone.call()
	
	    phone = new(IPhone)
	    phone.call()
	
	}
	```
	
2. 多态的实现(方法接受参数是一个接口类型，多个结构体实现了接口)

## 错误处理
1. example
	**golang的错误处理流程：当一个函数在执行过程中出现了异常或遇到	panic()，正常语句就会立即终止，然后执行defer语句，再报告异常信息，最后退出 goroutine。如果在defer中使用了recover() 函数,则会捕获错误信息，使该错误信息终止报告**

	```
	func testError(divide int) (int){

		defer func() {
			err := recover()
			if  err != nil {
				//说明捕获到异常
				log.Println("err=",err)
			}
		}()
	
		if divide == 0{
			panic("param is 0")
		}
		return 100/divide
	}
	```
	
## 并发
1. 通道channel,通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯

	```
	<!--传递数据到通道c-->
	func sum(s []int, c chan int) {
		sum := 0
		for _, v := range s {
			sum += v
		}
		c <- sum // 把 sum 发送到通道 c
	}
	
	func main() {
		s := []int{7, 2, 8, -9, 4, 0}
	
		c := make(chan int) //定义一个通道
		go sum(s[:len(s)/2], c)
		go sum(s[len(s)/2:], c)
		//x, y := <-c, <-c // 从通道 c 中接收数据
		x := <-c
		y := <-c
		fmt.Println(x, y, x+y)
	}
	```
	
2. 通道缓存区: c := make(chan int, 10),缓存区为10，若不指定缓存区，则必须写一个读一个，否则会堵塞


## 其他问题
1. 切片是引用类型，被多个变量引用时改变一个都会变
2. map是引用类型
3. string中输出每个字符时，注意占位问题，遇到占位两个字节的编码会出问题。，可以使用**==rune==**解决这个问题
> rune 是 Go 语言的内建类型，它也是 int32 的别称。在 Go 语言中，rune 表示一个代码点。代码点无论占用多少个字节，都可以用一个 rune 来表示

	```
	func printChars(s string) {
	    runes := []rune(s)
	    for i:= 0; i < len(runes); i++ {
	        fmt.Printf("%c ",runes[i])
	    }
	}
	
	func main() {
	    name := "Hello World"
	    printChars(name)
	    name = "Señor"
	    printChars(name)
	}
	```

4. Go 中的字符串是不可变的。一旦一个字符串被创建，那么它将无法被修改,是指不能修改比如 string[i],但是可以修改整个string
5. 结构体是值类型