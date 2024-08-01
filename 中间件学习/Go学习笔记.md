### Go并发实践

##### Go中的Sync包的使用

- Mutex 互斥锁
  1. Lock()
  2. UnLock()
- RWMutex 读写互斥锁
  1. Lock() 写的时候，排斥其他的写锁和读锁
  2. UnLock() 释放写锁
  3. Rlock() 在读取的时候 不会阻塞其他的读锁，但是会排斥写锁 Lock()
  4. RUnlock() 释放读锁
- Once Once.Do(一个函数) 这个方法无论被调用多少次，只会执行一次

```go
//使用方法：声明后即可调用
var once sync.Once
once.Do(func())
```

- WaitGroup
  1. Add(delta int) 设定需要Done多少次
  2. Done() 没执行一次加1
  3. Wait() 阻塞到 执行Done() 的次数 和 Add(delta int)的delta次数相同 示例代码详见 chapter_11/02_channel_done/channelWaitGroup_2
- Map 一个并发字典, 可并发读写
  1. Store(key, val) 写入字典中数据
  2. Load(key) 读取字段中数据
  3. LoadOrStore(key, defaultVal) 读不到设置key默认值defaultVal
  4. Range(func(key, value interface{}) bool {})
     传入一个函数，遍历字典（函数每返回true, 开始遍历下一个元素，返回false终止遍历）
  5. Delete(key) 删除字典中key
- Pool 并发池 通过Put 将数据丢到Pool中，然后Get() 但是没有顺序，可以用完再丢回去
  1. Put
  2. Get
- Cond 通知锁
  1. NewCond(lock) 创建一个Cond
  2. cond.L.Lock() ~ cond.L.Unlock() 创建一个锁区间 在区域内部可以cond.Wait()
  3. cond.Wait() 在锁区间内部可以cond.Wait()
  4. cond.Broadcast() 全部释放cond.Wait()
  5. cond.Signal() 解锁一个cond.Wait()

##### Go并发中slice和map踩的坑

slice的数据结构为

```go
type slice struct {
	array unsafe.Pointer //指向底层数组
	len   int
	cap   int
}
```

map的数据结构为

```go
type hmap struct {
    count     int // 元素的个数
    flags     uint8 // 标记读写状态，主要是做竞态检测，避免并发读写
    B         uint8  // 可以容纳 2 ^ N 个bucket
    noverflow uint16 // 溢出的bucket个数
    hash0     uint32 // hash 因子

    buckets    unsafe.Pointer // 指向数组buckets的指针
    oldbuckets unsafe.Pointer // growing 时保存原buckets的指针
    nevacuate  uintptr        // growing 时已迁移的个数

    extra *mapextra
}
```

在进行make的时候，`make([]int,0)`返回slice结构体，`make(map[int]int,0)`返回*hmap指针。

在并发地增加元素的时候，如果存在浅拷贝行为，即`slice1 := slice`和`map1 := map`，那么

* slice1在扩容阶段中会更改array指针，但slice不会有任何改变，因为slice1本质是结构值
* map1在扩容阶段会更改buckets指针，并且map也会一起改变，因为map1本质是*hmap指针

综上，在并发操作中，如果存在切片扩容缩容行为，那么使用时一定要使用切片指针（`&[]int{}`或者`slice:=make(int[],0) and slicePointer = &slice`）进行切片传递，而map则不需要（但是map不可以并发操作，需要锁）

### Go反射

go反射主要使用"reflect"包

普遍使用

```go
xType := reflect.TypeOf(x) //获取x的type
xValue :=relfect.ValueOf(x) //或者x的reflect.Value
y := xValue.Interface().(float64) //y will have type float64，获得具体的接口值
```

修改一个反射对象之前，必须确保其是可以修改的

```go
//修改一个float

//case 1
var x float64 = 3.14
v := reflect.ValueOf(x)
v.SetFloat(2.8) 
// 会引发pannic，因为其值是不可修改的，调用v.CanSet()会得到false

//case 2
var x float64 = 3.14
p := reflect.ValueOf(&x)  // Note: take the address of x.
v := p.Elem() //获取其对象
v.SetFloat(2.8) //成功

//修改一个结构体

type T struct {
  A int
  B string
}
 
t := T{23,"hello world"}
s := reflect.ValueOf(&t).Elem()
typeOfT := s.Type()
for i:=0; i<s.NumField(); i++ {
  f := s.Field(i)
  fmt.Printf("%d: %s %s = %v\n", i,typeOfT.Field(i).Name, f.Type(), f.Interface())
}
```

### Go调试

* core dump:
  * 应用程序在运行过程中由于各种异常或者bug导致退出，在满足一定条件下产生一个core文件。
  * 通常core文件包含了程序运行时内存、寄存器状态、堆栈指针、内存管理信息以及函数调用堆栈信息。
  * 通过ulimit -c unlimited可以打开coredump

* GOTRACEBACK:`GOTRACEBACK` 控制程序崩溃时输出的详细程度。 它可以采用不同的值：

  - `none` 不显示任何 goroutine 栈 trace。
  - `single`, 默认选项，显示当前 goroutine 栈 trace。
  - `all` 显示所有用户创建的 goroutine 栈 trace。
  - `system` 显示所有 goroutine 栈 trace,甚至运行时的 trace。
  - `crash` 类似 `system`, 而且还会生成 core dump。
  - 设置GOTRACEBACK=crash可以在程序崩溃时输出详细的信息并且生成core dump

* Delve:

  * Delve 是使用 Go 编写的 Go 程序的调试器。
  * 它可以通过在用户代码以及运行时中的任意位置添加断点来逐步调试，甚至可以使用以二进制文件和 core dump 为参数的命令 `dlv core` 调试 core dump。
  * 本地可使用goland远程连接进行debug，远程机器输入`dlv debug --headless --listen=:8345 --api-version=2 --accept-multiclient`（需要注意的是，加了 `--accept-multiclient` 后将无法执行 `ctrl+c` 终止）
  * 需要调试的程序要关闭编译器优化和内联优化方便查看`-gcflags="all=-N -l"`

* Go gcflags/ldflags 

  * `-gcflags`：可以控制编译器优化和内联优化

    * go build 时可以使用 `-gcflags` 指定编译选项，`-gcflags` 参数的格式是`-gcflags="pattern=arg list"`

    * pattern 是选择包的模式，它可以有以下几种定义:

      - `main`: 表示 main 函数所在的顶级包路径
      - `all`: 表示 GOPATH 中的所有包。如果在 modules 模式下，则表示主模块和它所有的依赖，包括 test 文件的依赖
      - `std`: 表示 Go 标准库中的所有包

    * arg list 是空格分割的编译选项，如果编译选项中含有空格，可以使用引号包起来

      下面介绍几种常用的编译选项:

      - `-N`: 禁止编译器优化
      - `-l`: 关闭内联 (inline)

  * `-ldflags` ：可以设置链接选项

    * `-w` 不生成 DWARF 调试信息
    * `-s` 关闭符号表（`-w` 和 `-s` 通常一起使用，用来减少[可执行文件](https://so.csdn.net/so/search?q=可执行文件&spm=1001.2101.3001.7020)的体积）
    * `-X`进行运行时的变量注入，如`go build -ldflags "-X main.Version=0.0.9" -o main main.go`，在运行时为main中的Version变量进行注入


### Go时间控制

**Timer:**定时发送信号到channel中，不需要时需要stop回收资源

```go
func NewTimer(d Duration) *Timer
func (t *Timer) Reset(d Duration) bool
func (t *Timer) Stop() bool
func After(d Duration) <-chan Time

//timer例子
func main() {
   timer := time.NewTimer(3 * time.Second)  //启动定时器，生产一个Timer对象
   select {
   case <-timer.C:
      fmt.Println("3秒执行任务")
   }
   timer.Stop() // 不再使用了，结束它
}
 
//time.After例子
func main() {
   tChannel := time.After(3 * time.Second) // 其内部其实是生成了一个Timer对象,但不可复用
   select {
   case <-tChannel:
      fmt.Println("3秒执行任务")
   }
}

//timer复用
func main() {
   timer := time.NewTimer(3 * time.Second)
   for {
      timer.Reset(3 * time.Second) // 这里复用了 timer
      select {
      case <-timer.C:
         fmt.Println("每隔3秒执行一次")
      }
   }
}
```

**Ticker:**Ticker 跟 Timer 的不同之处，就在于 Ticker 时间达到后不需要人为调用 Reset 方法，会自动续期，不需要时需要stop回收资源

```go
func NewTicker(d Duration) *Ticker
func Tick(d Duration) <-chan Time
func (t *Ticker) Stop()
 
func main() {
  ticker := time.NewTicker(3 * time.Second)
  for range ticker.C {
    fmt.Print("每隔3秒执行任务")
  }
  ticker.Stop()
}
```

