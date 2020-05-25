### 变量声明：

基本类型：

利用const实现枚举类型和自增



### 控制语句

- if：条件不用括号括起来，语句要用大括号括起来，在判断语句执行前可以在if后执行语句
- switch：自带break，可选fallthrough
- 循环（只有for）: for的三个组件都可以省略



### 函数

- 使用func定义函数

  ```go
  func functionname(a,b int)(int){
  	return a+b
  }
  ```

- 多个返回值
  
  - 接收返回值时使用逗号分割，也可以用下划线接收，那就是不使用这个返回值的意思（没必要新建一个变量）
- 参数类型可以是func
- 支持可变长参数



### 内置容器

- **Array**

- [10]int 和 [20]int 是不同类型，数组类型相同要长度和元素类型完全相同才可以

- `func loopArray(arr2 [3]int)` 是值传递，入参会拷贝一份数组，所以如果数组很大，从内存和性能上函数传递数组值都是很大的开销，需要避免（使用指针可以实现"引用传递" `func loopArray(arr2 *[3]int)`，调用方传入 `&arr2`）

- 在 Go 中一般不直接使用数组，而是使用切片

- 数组是定长的，不可扩展，切片相当于动态数组

- 使用

  ```go
  // 定义数组，不赋初值（使用默认值）
      var arr1 [5]int // [0 0 0 0 0]
      // 定义数组，赋初值
      arr2 := [3]int{1, 2, 3} // [1 2 3]
      // 定义数组，由编译器来计算长度，不可写成[]，不带长度或者 ... 的表示切片
      arr3 := [...]int{4, 5, 6, 7} // [4 5 6 7]
      // 创建指针数组
      arr4 := [2]*string{new(string), new(string)}
      *arr4[0] = "hello"
      *arr4[1] = "go"
      // 为指定索引位置设置值
      arr5 := [3]int{1:10} // [0,10,0]
      // 二维数组
      var grid [4][5]int // [[0 0 0 0 0] [0 0 0 0 0] [0 0 0 0 0] [0 0 0 0 0]]
      // 数组拷贝，直接复制一份 arr2 给 arr6
      arr6 := arr2
  ```



- **Slice**

- 底层是一个定长数组，一个slice包含三个参数，指向底层数组的指针，当前长度，最大长度

  ```go
      // 1、使用make函数创建一个字符串切片，长度和容量都是5
      slice1 := make([]string, 5)
      // 2、创建一个int切片，长度是3，容量是5
      slice2 := make([]int, 3, 5)
      // 3、使用字面量创建切片，长度是3，容量是3
      slice3 := []int{1, 2, 3} // [3]int{1, 2, 3}
      // 4、创建 nil 切片，长度为0，容量为0
      var slice4 []int
      // 5、创建空切片，长度为0，容量为0
      slice5 := make([]int, 0)
      slice6 := []int{}
  
      //source[i:j:k] 长度=j-i 容量=k-i
      slice := source[2:3:3]
  
      // 6、自定义底层数组，通过该底层数组创建切片
      arr := [5]int{1, 2, 3, 4, 5}
      // 数组转化为切片，左闭右开 [arr[2]~arr[4])
      slice7 := arr[2:4] // [3,4]
      slice8 := arr[2:]  // [3,4,5]
      slice9 := arr[:4]  // [1,2,3,4]
      slice10 := arr[:]   // [1,2,3,4,5]
  ```

- 如果有两个切片底层是同一个数组，那么一个切片修改了某个元素，也会修改数组里的元素，因而另一个切片里的值也会被改变，但是注意

- 这个容量就是这个切片的首个元素在底层数组中的位置到底层数组中最后一个位置的长度

- 使用append可以增加元素，如果超过了容量就会触发扩容，扩容机制是新建一个长度为原数组长度2倍（或1.25倍，大于1000时）的数组，然后将原数组元素拷贝到新数组中。

- slice可以合并，即首尾相接

- 迭代slice

  ```go
      slice := []int{10, 20, 30, 40}
      // 与数组迭代一样，可以使用 for range + 普通 for 循环
      for index,value := range slice {
          fmt.Println(index, value)
      }
  ```

- 调用函数时，如果使用切片作为参数，由于一个切片底层是数组，因而会传递数组指针以及长度及容量的一个拷贝，对切片的操作会影响原数组的内容。



- **Map**

- 创建方法，K和V使用冒号分割，KV和KV之间使用逗号分割

  ```go
      // 1、使用 make 创建 map，key为string，value为int
      map1 := make(map[string]int)
  
      // 2、使用字面量创建 map - 最常用的姿势，key为string，value为slice
      map2 := map[string][]string{"hi": {"go", "c"}, "hello": []string{"java"}}
  
  	// 3、创建空映射
      map3 := map[string]string{} // map3 := map[string]string nil映射
      fmt.Println(map1, map2, map3)
  
      // 4、向映射添加值
      fmt.Println("map put")
      map3["a"] = "x"
      map3["b"] = "y"
      fmt.Println(map3) // map[a:x b:y]
  
      // 5、获取值并判断是否存在
      value, exist := map3["c"]
      if exist {
          fmt.Println(value)
      } else {
          fmt.Println("map3[\"c\"] does not exist")
      }
  
      // 6、迭代
      fmt.Println("iterat")
      for key, value := range map3 {
          fmt.Println(key, value)
      }
  
      // 7、从 map 中删除元素
      delete(map3, "a")
      fmt.Println(map3) // map[b:y]
  ```

- 函数传递map作为参数不会拷贝一个新的map



### 自定义类

- 定义：成员变量可以是一个类

  ```go
  // user 类
  type user struct {
      name       string
      email      string
      ext        int
      privileged bool
  }
  ```

- 实例化对象

  ```go
      // 1. 创建 user 变量，所有属性初始化为其零值
      var bill user
  
      // 2. 创建 user 变量，并初始化属性值
      lisa := user{
          name:       "nana",
          email:      "117@qq.com",
          ext:        123,
          privileged: true,
      }
  
      // 3. 直接使用属性值，属性值的顺序要与 struct 中定义的一致
      lisa2 := user{"nana", "117@qq.com", 123, true}
  ```



### 方法（与函数有区别）

- 普通的函数定义 func 方法名(入参) 返回值
- 自定义类型的方法定义 func **(接收者)** 方法名(入参) 返回值，表示这个函数是该接收者（自定义类）的的成员函数，只能通过该类的对象去调用。
- 接收者可以使用类，也可以使用类的指针。

> If the receiver is a map, func or chan, don't use a pointer to it.
> If the receiver is a slice and the method doesn't reslice or reallocate the slice, don't use a pointer to it.
> If the method needs to mutate（使 receiver 改变） the receiver, the receiver must be a pointer.
> If the receiver is a struct that contains a sync.Mutex or similar synchronizing field, the receiver must be a pointer to avoid copying.
> If the receiver is a large struct or array, a pointer receiver is more efficient. How large is large? Assume it's equivalent to passing all its elements as arguments to the method. If that feels too large, it's also too large for the receiver.
> Can function or methods, either concurrently or when called from this method, be mutating the receiver? A value type creates a copy of the receiver when the method is invoked, so outside updates will not be applied to this receiver. If changes must be visible in the original receiver, the receiver must be a pointer.
> If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention more clear to the reader.
> If some of the methods of the type must have pointer receivers, the rest should too, so the method set is consistent regardless of how the type is used
> If the receiver is a small array or struct that is naturally a value type (for instance, something like the time.Time type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, a value receiver makes sense.
> A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap. (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first.
> Finally, when in doubt, use a pointer receiver.



### 嵌入类型

- 在一个自定义类中可以嵌入另一个类型，注意跟前面的成员变量不同，嵌入的类型不是作为一个成员而是对这个自定义类的一个扩展，相当于继承（允许多继承）
- 嵌入了一个类，那么以这个类为接受类的方法可以被新的类型的实例调用。（里氏替换原则）



### 接口

- 定义接口

```go
func notifier interface{
	notify()
}
```

- 实现接口（跟写一个方法一样），需要指定接受者（值或指针），那么这个接收者就实现了这个接口（实现了所有的方法）
- 然后这个这个接口可以作为参数传入某个函数中，传参时传入一个实现类
  - 如果你实现接口里的方法是使用**值接受者(不修改原对象)**，那么传入接口实现类作为参数的时候**可以使用值传递也可以使用指针传递**
  - 如果你实现接口里的方法是使用**指针接受者（修改原对象）**，那么传入接口实现类作为参数的时候**只能使用指针传递**



### 访问控制及多态的实现

- 接口名和接口方法大写表示public，小写表示包隔离级别（在同一个包内可以访问）
- 跟java一样，运行时多态会一直向父类找实现直到找到实现位置
- 组合接口：就是有一个大接口内包含两个子接口，如果说有一个自定义类同时实现了这两个接口，那么这个自定义类可以被当做大接口的实现类来使用
- 当我们[嵌入](http://golang.org/doc/effective_go.html#embedding)一个类型，这个类型的方法就变成了外部类型的方法，但是当它被调用时，方法的接受者是内部类型(嵌入类型)，而非外部类型。
- **闭包**：函数中可以声明其他函数，并且可以以函数作为返回值



### 资源管理和错误处理

- 资源处理：用来关闭一些资源
- 使用defer关键词 + 要执行的代码
- 在一个函数中可以使用多次defer，会以栈的形式存储这些语句，在函数执行完之后，然后出栈执行。



- 如何自定义一个错误（应该多用这个）

  - 实现error接口

    ```go
    type HTTPError struct {
        Code        int
        Description string
    }
    
    func (h *HTTPError) Error() string {
        return fmt.Sprintf("status code %d: %s", h.Code, h.Description)
    }
    ```

  - 使用`errors.New("Error info")`

    ```go
    //你也可以直接返回一个New出来的错误，但是先声明好的话会有利于抛出错误之后去判断错误类型
    var ErrNoSpaceLeft = errors.New("no space left on the device")
    var ErrPermissionDenied = errors.New("permission denied")
    
    func doStuff() error {
        if someCondition {
            return ErrNoSpaceLeft
        } else {
            return ErrPermissionDenied 
        }
    }
    
    if err == ErrNoSpaceLeft {
        // handle this particular error
    }
    ```

  - 



- 错误处理：(跟exception有区别不要滥用)

- panic 
  - 停止当前程序运行
  - 一直向上返回，执行每一层的 defer
  - 如果没有遇见 recover，程序退出

- recover

  - 仅在 defer 调用中使用
  - 获取 panic 的值

  ```go
          r := recover()
          if err, ok := r.(error); ok {
              fmt.Println(err) // runtime error: integer divide by zero
          } else {
              panic(errors.New("not known error"))
          }
  ```

  - 如果无法处理，可重新 panic
  
- 使用场景：

  - 严重错误，必须退出进程，再进行下去意义不大，比如参数不合规范
  - 快速退出错误处理，正常情况下用error一层层往上返回也是可以的，但是调用栈太深的话冗余代码就会很多，因此使用panic可以快速pop调用栈。



### Goroutine

- go语言的最大特色就是从语言层面上支持并发，Goroutine是最基本的执行单元。每一个Go程序至少有一个Goroutine：主Goroutine，在程序启动后它会自动创建。**多个goroutine可以在同一个线程中，有一个工作队列保存一个线程里的goroutine。**

- 有两种方式创建一个Goroutine:

  - 匿名函数实现方式 `go func() {xxx}()`
  - 普通函数 funcA 实现方式 `go funcA()`

- 一个正在运行的goroutine在工作结束前，可以被停止并重新调度

- 锁的使用：

  ```go
          lock.Lock()
          { 
              // Lock() 与 UnLock() 之间的代码都属于临界区，{}是可以省略的，加上看起来清晰
              value := counter
              // 当前的 goroutine 主动让出资源，从线程退出，并放回到队列，
              // 让其他 goroutine 进行执行
              // 但是因为锁没有释放，调度器还会继续安排执行该 goroutine
              runtime.Gosched()
              value ++
              counter = value
          }
          lock.Unlock()
  ```

- Go的并发模型：

  - 多线程共享内存

    - 同一时刻只能有一个 goroutine 对共享资源进行读和写操作，类似java，需要同步操作

  - CSP并发模型：“以通信的方式来共享内存”

    - goroutine之间使用管道进行通信，
    -  messages := make(chan string)  创建一个名为messages的管道，chan是关键字

    ```go
    import "fmt"
    func main() {
       messages := make(chan string)
       //向管道里传数据
       go func() { messages <- "ping" }()
       //从管道里取数据到msg
       msg := <-messages
       fmt.Println(msg)
    }
    ```

    - 有两种channel，分别是无缓冲的和有缓冲的
      - 无缓冲：传、取时都会阻塞，直到有另外的goroutine向这个管道取或传，传取双方都必须准备好，创建方法就是上面这种
      - 有缓冲：`messages := make(chan string, num)`，num表示容量，如果channel关闭，依然可以取出内容但是不可以再向里面放