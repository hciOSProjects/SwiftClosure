#Swift Closure 

### 目录
- 语法表达式
- 起别名
- 尾随闭包
- 捕获值
- 逃逸闭包
- 自动闭包

_ _ _

#### 语法表达式
- ##### 一般形式
```
let addBlock: (Int, Int) -> Int = {
        (arg1: Int, arg2: Int) -> Int in
        return arg1 + arg2
    }
```
- Swift可以根据闭包上下文推断参数和返回值的类型，所以上面的例子可以简化如下
```
 let addBlock2: (Int, Int) -> Int = {
        arg1, arg2 in      // 也可以写成(a,b) in
        return arg1 + arg2
    }
```
- 单行表达式闭包可以隐式返回，如下，省略return
```
let addBlock3: (Int, Int) -> Int = { (arg1, arg2) in arg1 + arg2 }
```

- 如果闭包没有参数, 则 `in` 可以省略
```
let addBlock4: () -> Int = {return 100 + 150}
```

- 这个是既没有参数也没返回值，所以把return和in都省略了
```
let addBlock5: () -> Void = {print("Hello world") }
```

`print("一般形式: \(addBlock(100, 150))")  // 打印结果：250`

_ _ _

#### 起别名
- 也可以关键字“typealias”先声明一个闭包数据类型。类似于OC中的typedef起别名
`    typealias MinusBlock = (Int, Int) -> Int`
```
let minusBlock: MinusBlock = {
        (arg1, arg2) in
        return arg1 - arg2
    }
```
`调用输出结果为 250`
`let result = minusBlock(500, 250)`
 `print("起别名: \(result)")`
_ _ _

#### 尾随闭包
-  若将闭包作为函数最后一个参数，可以省略参数标签,然后将闭包表达式写在函数调用括号后面
```
    func myFoo(testBlock: () -> Void) {
        // 这里需要传进来的闭包类型是无参数和无返回值的
        testBlock()
    }
```
`调用尾随闭包`
```
	myFoo() {
        print("尾随闭包写法")
    }
```

     也可以把括号去掉，也是尾随闭包写法。**推荐写法**
```
    myFoo { 
        print("去掉括号的尾随闭包写法")
    }
```

_ _ _

#### 捕获值
- 闭包可以在其被定义的上下文中捕获常量或变量。Swift中，可以捕获值的闭包的最简单形式是嵌套函数，也就是定义在其他函数的函数体内的函数。
 ```
 func captureValue(sums amount: Int) -> (() -> Int) {
        var total = 0
        func incrementer() -> Int {
            total += amount
            return total
        }
        return incrementer
    }
```
- 这里没有值捕获的原因是，没有去用一个常量或变量去引用函数，所以每次使用的函数都是新的。有点类似于OC中的匿名对象。

`print(captureValue(sums: 10)())`
`print(captureValue(sums: 10)())`
`print(captureValue(sums: 10)())`
    
    输出结果为
    10
    10
    10

- 这里值捕获了，是因为函数被引用了，所以没有立即释放掉。所以函数体内的值可以被捕获
` let referFunc = captureValue(sums: 10)`
` print(referFunc())`
` print(referFunc())`
` print(referFunc())`

    输出结果为
    10
    20
    30

** 由上面的例子都可以证得，函数和闭包都是引用类型。**

_ _ _

#### 逃逸闭包
- 当一个闭包作为参数传到一个函数中，需要这个闭包在函数返回之后才被执行，我们就称该闭包从函数种逃逸。一般如果闭包在函数体内涉及到异步操作，但函数却是很快就会执行完毕并返回的，闭包必须要逃逸掉，以便异步操作的回调。
- 逃逸闭包一般用于异步函数的回调，比如网络请求成功的回调和失败的回调。语法：在函数的闭包形参前加关键字“@escaping”。

`例1`
  ```
func doSomething(some: @escaping () -> Void){
        //延时操作，注意这里的单位是秒
        DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 1) {
            //1秒后操作
            some()
        }
        print("函数体")
    }
```

`例2`
 ```
var comletionHandle: ()->String = {"约吗?"}
    func doSomething2(some: @escaping ()->String){
        comletionHandle = some
    }
```

- 将一个闭包标记为@escaping意味着你必须在闭包中显式的引用self。
- 其实@escaping和self都是在提醒你，这是一个逃逸闭包，别误操作导致了循环引用！而非逃逸包可以隐式引用self。
    `例子如下`

 ```
    var completionHandlers: [() -> Void] = []

    //逃逸
    func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
            completionHandlers.append(completionHandler)
        }

    //非逃逸
    func someFunctionWithNonescapingClosure(closure: () -> Void) {
            closure()
        }
 ```

`var x = 10`
    
 调用逃逸闭包

 ```
  doSomething {
      print("逃逸闭包")
  }
 ```

 ```
  print(comletionHandle())
 ```
 ```
  doSomething2 {
     return "叔叔，我们不约"
  }
 ```
 ```
  print(comletionHandle())
 ```

 ```
/// 将一个闭包标记为@escaping意味着你必须在闭包中显式的引用self。
/// 其实@escaping和self都是在提醒你，这是一个逃逸闭包，
/// 别误操作导致了循环引用！而非逃逸包可以隐式引用self。
doSomething {
    self.someFunctionWithEscapingClosure { self.x = 100 }
    self.someFunctionWithNonescapingClosure { self.x = 200 }
}
 ```
 输出结果
` 约吗？`
`叔叔,我们不约`
`逃逸闭包`

_ _ _

#### 自动闭包
- 顾名思义，自动闭包是一种自动创建的闭包，封装一堆表达式在自动闭包中，然后将自动闭包作为参数传给函数。而自动闭包是不接受任何参数的，但可以返回自动闭包中表达式产生的值。
- 自动闭包让你能够延迟求值，直到调用这个闭包，闭包代码块才会被执行。说白了，就是语法简洁了，有点懒加载的意思。
```
 var array = ["I", "have", "an", "apple"]
 print(array.count)  // print "4"

 //测试了下，这里代码超过一行，返回值失效。
 let removeBlock = {array.remove(at: 3)}   // print "4"
 print("执行代码块移除\(removeBlock())")   // print "执行代码块移除apple", 这里自动闭包返回了apple值

 print(array.count)  // print "3"
```

