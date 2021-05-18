# go 包管理

## 包package

### package基本使用

- 项目中的每个文件夹都是一个package。package下可以嵌套package。但每个.go文件的package以其所在的当前文件夹为准。
- 在GoLand中新建文件夹时，在该层目录下新建的.go文件，其package默认为当前文件夹名（大小写一致）。当.go文件创建后，再去修改文件夹名时，需要注意其下的.go文件中的package是否自动修改。
- ==同一层目录下的.go文件，package必须一致==，否则编译出错。

```
同层目录不包括嵌套目录

如下所示：
a1.go和a2.go属于dir A这一级目录下，其package必须一致。
b1.go和b2.go属于dir B这一级目录。

projectName
    src
        dir A
             dir B
                b1.go
                 b2.go
        a1.go
        a2.go
```

例如，文件夹world下的所有.go文件，package必须一致,package可以随便起，不一定与文件夹名一样。

### 包（package）可见性

> 在了解可见性之前，我们先了解一个概念：可导出的。
> 在go语言中，变量或函数名的首字母大写时，其就是可导出的，小写时则是不可导出的。

- 函数和变量的可访问性是以包做隔离的。

```
包          函数或变量      可访问性

同一个包    可/不可导出的     都能访问
其他的包    可导出的          可访问
其他的包    不可导出的        不能访问
```

同一个包内，均可访问导出/不可导出的变量或函数。无论变量是全局变量还是struct的成员变量，无论函数是全局函数还是隶属于某个struct的成员函数。

> 这里的“全局”指的是不通过“对象.变量或函数”，而是直接通过package.变量或函数即可访问的变量或函数。
> · 全局变量在同一个包内，直接访问，包外通过package.变量访问，函数亦然。当然struct对象也是一种特殊的全局数据结构，遵循这个规则。
> · 对象成员变量或函数，必须通过“对象实例.变量或函数”访问。

例如：

```
//项目结构
project
    src
        dir A
            a1.go
            a2.go
        dir B
            b.go

//项目代码
------------- a1.go
package A

//可导出的：包内包外都能访问
//不可导出的：包内可见，包外不可见

//A1是可导出的
type A1 struct{
    Name string //可导出的
    age  int //不可导出的
}

//a1是不可导出的
type a1 struct{

}

var V string //可导出（全局变量）

var v string //不可导出（全局变量）

func F1(){} //可导出的（全局函数）

func f1(){} //不可导出的（全局函数）

func (A *A1)F1(){} //可导出的（对象函数）

func (A *A1)f1(){} //不可导出的（对象函数）

func (a *a1)F1(){} //可导出的（对象函数）

func (a *a1)f1(){} //不可导出的（对象函数）


------------- a2.go
package A

func A2(){
    //同一个包内,无论导出性与否，均可直接访问
    obj_A1:=new(A1) 
    obj_a1:=new(a1)
    _:=V        //"_"表示接收右边的变量，但不使用
    _:=v
    F1()
    f1()
    obj_A1.F1() //对象成员（包括函数和变量），需要通过"对象.成员"方式访问
    obj_A1.f1()
    obj_a1.F1()
    obj_a1.f1()
}

------------- b.go
package B

import (
    "A"
)

//引用其他包，需要import "指定的包"
//使用其他包的成员，需要通过 "指定包.成员" 方式访问
//"指定包.成员"中，只有可导出的成员是可见的
func B(){
    //下面都是正确的（除了最后一个）
    obj_A1:=new(A.A1)
    _:=A.V
    A.F1()
    obj_A1.F1()
    obj_A1.f1()     //错误

    //下面都是错误的。不可导出成员包外不可见
    obj_a1:=new(A.a1)
    _:=A.v
    A.f1()
    obj_a1.F1()
    obj_a1.f1()
}
```

### package main之间运行时不能互相访问

> 备注：
> main是个特殊的package。go的可执行方法，只有main包下的main方法。

同一级文件夹下的package是main，当该文件夹下的某个.go文件调用同级目录下的变量或方法时，静态编译不报错，但是执行run动态编译时，就会报undefined错误。

```
# command-line-arguments
src/main.go:11: undefined: B
```

如：

```
dir A
    a1.go
    a2.go


---- a1.go
package main

func main(){
    //静态编译（即写代码时）不报错
    //运行该main()方法时，报错
    B() //调用a2.go文件的B（）方法
}


----- a2.go
package main

func B(){
    fmt.Plintln("I am B()")
}
```

main包的文件夹下嵌套main包的子目录时，文件夹之间的.go文件彼此不可见。如

```
dir A
    dir B
        b1.go
        b2.go
    a1.go
    a2.go

---- a1.go
package main

func A1(){
    A2()    //静态编译通过，动态编译找不到
    B1()    //静态编译报错，B1()在这里不可见
}

---- a2.go
package main

func A2(){}

---- b1.go
package main

func B1(){
    B2()    //静态编译通过，动态编译找不到
    A2()    //静态编译报错，A2()在这里不可见
}

---- b2.go
package main

func B2(){}
```

### 包内嵌套同样的包（非main包）

相互调用是允许的，但不建议这么做，容易出错。

```
src 
    dir world
        dir sag
            a2.go 
        a1.go
    main.go

---- a1.go
package world

func A(){
    fmt.Println("I am A1()")
}


---- a2.go
package world

func A(){
    fmt.Println("I am A2()")
}

---- main.go
package main

//注意这里！！！
//存在包内嵌套同样的包时，使用全目录名代表对应目录下的包package
import (
    "world"     
    world2 "world/sag"  
)

func main(){
    world.A()   //a1.go的
    world2.A()  //a2.go的
}

运行结果：
I am A1()
I am A2()


注意：

由于2个嵌套目录下的包package都是world，为了做区分
，import引用相应包时，go使用全目录名（默认是src目
录下）代表该目录下的包。即使这2个目录下的包不是world，
而是其他值，只要包相同，则使用目录代替。


总结：

1. 其实，无论是否存在相同的包，go import时都使用
全目录名代表引用该目录下的包。使用该包的变量或函
数时，用"包.变量or函数“;如果import时给引用的目录
起了别名，就用"别名.变量or函数"引用。

2. 目录名与包名可以相同，也可不同。
```

看一个相同包的例子

![image](https://note.youdao.com/yws/public/resource/4caf8407e7f254449851a7833a32c668/xmlnote/39293BCC93AA4DF0A095CBBD6823E11C/7262)

再强调一次
==**总结：**==

1. 其实，无论是否存在相同的包，go import时都使用
   “**全目录名**” 代表”引用该目录下的包package”。使用该包的变量或函
   数时，用”**包.变量or函数**“;如果import时给引用的目录
   起了别名，就用”**别名.变量or函数**“引用。
2. 目录名与包名**可以相同，也可不同**。

### go初始化顺序init

```
package main

import(
    //以下引用均为举例，"项目名/src/"是默认前缀
    "a/b/c" //引用"项目名/src/a/b/c目录下的包"
    "b/c/d" 
    "x/xx/xxx"  //"项目名/src/x/xx/xxx目录下的包"
)

func main(){
    //程序的执行入口
}
```

前面提过，main包下的main函数是程序的可执行函数，即入口函数。
go程序启动时，执行过程如下：

![image](https://note.youdao.com/yws/public/resource/4caf8407e7f254449851a7833a32c668/xmlnote/E5C288F27F3441928BD243B31E699787/7348)

从图中可知，运行main方法时：

1. 先初始化import的所有包
2. 先初始化const常量，然后是全局变量var，再是init()函数
3. 当import下的包都初始化结束后，最后执行main的const,var，init(),main()

### 包循环引用import

从go的初始化图中可知，go的初始化存在顺序问题，这就可能导致“循环引用”，循环引用import在go中是不允许的，动态编译会报错。

举例：

```
package main

import（
    "a" //main 引用a
）

---------
package a

import（
    "b" //a 引用b
）

---------
package b

import（
    "c" //b 引用c
）

---------
package c

import（
    "a" //c 引用a
）
```

从例子中可见，当指定main（）函数时，由于go的初始化顺序，
· 先初始main包的import。这里引用a包
· 再初始化a包。a包引用了b包。
· 接着初始化b包。b包引用了c包。
· 然后初始化c包。c包引用了a包。
· 由于c包引用了a包，需要初始化话a包。

包初始化顺序:
main -> a -> b -> c -> a

这就形成了循环引用。动态编译不通过。所以，写go时，注意避开循环引用。