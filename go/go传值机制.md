# Go语言参数传递是传值还是传引用

## 什么是传值（值传递）

传值的意思是：函数传递的总是原来这个东西的一个副本，一副拷贝。比如我们传递一个`int`类型的参数，传递的其实是这个参数的一个副本；传递一个指针类型的参数，其实传递的是这个该指针的一份拷贝，而不是这个指针指向的值。

对于int这类基础类型我们可以很好的理解，它们就是一个拷贝，但是指针呢？我们觉得可以通过它修改原来的值，怎么会是一个拷贝呢？下面我们看个例子。

```go
func main() {
   i:=10
   ip:=&i
   fmt.Printf("原始指针的内存地址是：%p\n",&ip)
   modify(ip)
   fmt.Println("int值被修改了，新值为:",i)
}

func modify(ip *int){
   fmt.Printf("函数里接收到的指针的内存地址是：%p\n",&ip)
   *ip=1
}
```



我们运行，可以看到输入结果如下：

```
原始指针的内存地址是：0xc42000c028 
函数里接收到的指针的内存地址是：0xc42000c038 
int值被修改了，新值为: 1
```

首先我们要知道，任何存放在内存里的东西都有自己的地址，指针也不例外，它虽然指向别的数据，但是也有存放该指针的内存。

所以通过输出我们可以看到，这是一个指针的拷贝，因为存放这两个指针的内存地址是不同的，虽然指针的值相同，但是是两个不同的指针。

[![指针传递解释](https://www.flysnow.org/uploads/2018/02/pointer-param.png)](https://www.flysnow.org/uploads/2018/02/pointer-param.png)

通过上面的图，可以更好的理解。 首先我们看到，我们声明了一个变量`i`,值为`10`,它的内存存放地址是`0xc420018070`,通过这个内存地址，我们可以找到变量`i`,这个内存地址也就是变量`i`的指针`ip`。

指针`ip`也是一个指针类型的变量，它也需要内存存放它，它的内存地址是多少呢？是`0xc42000c028`。 在我们传递指针变量`ip`给`modify`函数的时候，是该指针变量的拷贝,所以新拷贝的指针变量`ip`，它的内存地址已经变了，是新的`0xc42000c038`。

不管是`0xc42000c028`还是`0xc42000c038`，我们都可以称之为指针的指针，他们指向同一个指针`0xc420018070`，这个`0xc420018070`又指向变量`i`,这也就是为什么我们可以修改变量`i`的值。

## 什么是传引用(引用传递)

Go语言(Golang)是没有引用传递的，这里我不能使用Go举例子，但是可以通过说明描述。

以上面的例子为例，如果在`modify`函数里打印出来的内存地址是不变的，也是`0xc42000c028`，那么就是引用传递。

## 迷惑Map

了解清楚了传值和传引用，但是对于Map类型来说，可能觉得还是迷惑，一来我们可以通过方法修改它的内容，二来它没有明显的指针。

```go
func main() {
	persons:=make(map[string]int)
	persons["张三"]=19

	mp:=&persons

	fmt.Printf("原始map的内存地址是：%p\n",mp)
	modify(persons)
	fmt.Println("map值被修改了，新值为:",persons)
}

 func modify(p map[string]int){
	 fmt.Printf("函数里接收到map的内存地址是：%p\n",&p)
	 p["张三"]=20
 }
```



运行打印输出：

````
原始map的内存地址是：0xc42000c028
函数里接收到map的内存地址是：0xc42000c038
map值被修改了，新值为: map[张三:20]
````

`两个内存地址是不一样的，所以这又是一个值传递（值的拷贝），那么为什么我们可以修改Map的内容呢？先不急，我们先看一个自己实现的`struct`。

```go
func main() {
	p:=Person{"张三"}
	fmt.Printf("原始Person的内存地址是：%p\n",&p)
	modify(p)
	fmt.Println(p)
}

type Person struct {
	Name string
}

 func modify(p Person) {
	 fmt.Printf("函数里接收到Person的内存地址是：%p\n",&p)
	 p.Name = "李四"
 }
```



运行打印输出：

```
原始Person的内存地址是：0xc4200721b0
函数里接收到Person的内存地址是：0xc4200721c0
{张三}
```

我们发现，我们自己定义的`Person`类型，在函数传参的时候也是值传递，但是它的值(`Name`字段)并没有被修改，我们想改成`李四`，发现最后的结果还是`张三`。

这也就是说，`map`类型和我们自己定义的`struct`类型是不一样的。我们尝试把`modify`函数的接收参数改为`Person`的指针。

```go
func main() {
	p:=Person{"张三"}
	modify(&p)
	fmt.Println(p)
}

type Person struct {
	Name string
}

 func modify(p *Person) {
	 p.Name = "李四"
 }
```

在运行查看输出，我们发现，这次被修改了。我们这里省略了内存地址的打印，因为我们上面`int`类型的例子已经证明了指针类型的参数也是值传递的。 指针类型可以修改，非指针类型不行，那么我们可以大胆的猜测，我们使用`make`函数创建的`map`是不是一个指针类型呢？看一下源代码:

```go
// makemap implements a Go map creation make(map[k]v, hint)
// If the compiler has determined that the map or the first bucket
// can be created on the stack, h and/or bucket may be non-nil.
// If h != nil, the map can be created directly in h.
// If bucket != nil, bucket can be used as the first bucket.
func makemap(t *maptype, hint int64, h *hmap, bucket unsafe.Pointer) *hmap {
    //省略无关代码
}
```

通过查看`src/runtime/hashmap.go`源代码发现，的确和我们猜测的一样，`make`函数返回的是一个`hmap`类型的指针`*hmap`。也就是说`map===*hmap`。 现在看`func modify(p map)`这样的函数，其实就等于`func modify(p *hmap)`，和我们前面第一节**什么是值传递**里举的`func modify(ip *int)`的例子一样，可以参考分析。

所以在这里，Go语言通过`make`函数，字面量的包装，为我们省去了指针的操作，让我们可以更容易的使用map。这里的`map`可以理解为引用类型，但是记住引用类型不是传引用。

## chan类型

`chan`类型本质上和`map`类型是一样的，这里不做过多的介绍，参考下源代码:

```go
func makechan(t *chantype, size int64) *hchan {
    //省略无关代码
}
```



`chan`也是一个引用类型，和`map`相差无几，`make`返回的是一个`*hchan`。

## 和map、chan都不一样的slice

`slice`和`map`、`chan`都不太一样的，一样的是，它也是引用类型，它也可以在函数中修改对应的内容。

```go
func main() {
	ages:=[]int{6,6,6}
	fmt.Printf("原始slice的内存地址是%p\n",ages)
	modify(ages)
	fmt.Println(ages)
}

func modify(ages []int){
	fmt.Printf("函数里接收到slice的内存地址是%p\n",ages)
	ages[0]=1
}
```



运行打印结果，发现的确是被修改了，而且我们这里打印`slice`的内存地址是可以直接通过`%p`打印的,不用使用`&`取地址符转换。

这就可以证明`make`的slice也是一个指针了吗？不一定，也可能`fmt.Printf`把`slice`特殊处理了。

```go
func (p *pp) fmtPointer(value reflect.Value, verb rune) {
	var u uintptr
	switch value.Kind() {
	case reflect.Chan, reflect.Func, reflect.Map, reflect.Ptr, reflect.Slice, reflect.UnsafePointer:
		u = value.Pointer()
	default:
		p.badVerb(verb)
		return
	}
	//省略部分代码
}
```



通过源代码发现，对于`chan`、`map`、`slice`等被当成指针处理，通过`value.Pointer()`获取对应的值的指针。

```
// If v's Kind is Slice, the returned pointer is to the first
// element of the slice. If the slice is nil the returned value
// is 0.  If the slice is empty but non-nil the return value is non-zero.
func (v Value) Pointer() uintptr {
	// TODO: deprecate
	k := v.kind()
	switch k {
	//省略无关代码
	case Slice:
		return (*SliceHeader)(v.ptr).Data
	}
}
```



很明显了，当是`slice`类型的时候，返回是`slice`这个结构体里，字段Data第一个元素的地址。

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}

type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```



所以我们通过`%p`打印的`slice`变量`ages`的地址其实就是内部存储数组元素的地址，`slice`是一种结构体+元素指针的混合类型，通过元素`array`(`Data`)的指针，可以达到修改`slice`里存储元素的目的。

所以修改类型的内容的办法有很多种，类型本身作为指针可以，类型里有指针类型的字段也可以。

单纯的从`slice`这个结构体看，我们可以通过`modify`修改存储元素的内容，但是永远修改不了`len`和`cap`，因为他们只是一个拷贝，如果要修改，那就要传递`*slice`作为参数才可以。

```go
func main() {
	i:=19
	p:=Person{name:"张三",age:&i}
	fmt.Println(p)
	modify(p)
	fmt.Println(p)
}

type Person struct {
	name string
	age  *int
}

func (p Person) String() string{
	return "姓名为：" + p.name + ",年龄为："+ strconv.Itoa(*p.age)
}

func modify(p Person){
	p.name = "李四"
	*p.age = 20
}
```



运行打印输出结果为：

```bash
姓名为：张三,年龄为：19
姓名为：张三,年龄为：20
```

通过这个`Person`和`slice`对比，就更好理解了，`Person`的`name`字段就类似于`slice`的`len`和`cap`字段，`age`字段类似于`array`字段。在传参为非指针类型的情况下，只能修改`age`字段，`name`字段无法修改。要修改`name`字段，就要把传参改为指针，比如：

```go
modify(&p)
func modify(p *Person){
	p.name = "李四"
	*p.age = 20
}
```

这样`name`和`age`字段双双都被修改了。

所以`slice`类型也是引用类型。

## 小结

最终我们可以确认的是Go语言中所有的传参都是值传递（传值），都是一个副本，一个拷贝。因为拷贝的内容有时候是非引用类型（int、string、struct等这些），这样就在函数中就无法修改原内容数据；有的是引用类型（指针、map、slice、chan等这些），这样就可以修改原内容数据。

是否可以修改原内容数据，和传值、传引用没有必然的关系。在C++中，传引用肯定是可以修改原内容数据的，在Go语言里，虽然只有传值，但是我们也可以修改原内容数据，因为参数是引用类型。

这里也要记住，引用类型和传引用是两个概念。

再记住，Go里只有传值（值传递）。