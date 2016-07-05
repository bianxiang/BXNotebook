[toc]


## 7. Swift数据类型、常量变量

一个应用中可以混用Swift和Objective-C。

### 7.2 Swift数据类型

**7.2.1 整数**

Swift支持 8, 16, 32 and 64 位整数（表示成 `Int8`, `Int16`, `Int32` and `Int64`）。于此对应的无符号数：UInt8, UInt16, UInt32 and UInt64。

官方推荐只使用`Int`类型，而不是使用特定大小的类型。The Int data type will use the appropriate integer size for the platform on which the code is running.

所有整数类型都包含边界属性，用于获得该类型支持的最大最小值。

```
println("Int32 Min = \(Int32.min) Int32 Max = \(Int32.max)")
```

**7.2.2 浮点数**

Swift提供两种浮点类型：`Float`和`Double`。`Double`类型用于存储最大64位浮点数，精度至少15个数字。`Float`类型，存储32位浮点数，精度至少6个数字（取决于代码运行的平台）。

**7.2.3 布尔类型**

Two Boolean constant values (`true` and `false`) are provided by Swift specifically for working with `Boolean` data types.

**7.2.4 字符类型**

The Swift `Character` data type is used to store a single Unicode character.

```
var myChar1 = "f"
```

**7.2.5 字符串类型**

字符串可以通过插值的方式构建，例子：

```
var userName = "John"
var inboxCount = 25
let maxCount = 100
var message = "\(userName) has \(inboxCount) message. Message capacity remaining is \(maxCount - inboxCount)"
println(message)
```

转义字符的例子：

```
var newline = "\n"
var backslash = "\\"
```

其他：

- `\u{nn}` – Single byte Unicode scalar where nn is replaced by two hexadecimal digits representing the Unicode character.
- `\u{nnnn}` – Double byte Unicode scalar where nnnn is replaced by four hexadecimal digits representing the Unicode character.
- `\U{nnnnnnnn}` – Four byte Unicode scalar where nnnnnnnn is replaced by four hexadecimal digits representing the Unicode character.

### 7.5 声明常量和变量

变量通过`var`关键字声明；可以初始化。若变量声明时不带初始值，则必须声明为可选的（**optional**，见后面章节）。

```
var userCount = 10
```

常量用`let`关键字声明。声明时必须初始化，而后值不能变。

```
let maxUserCount = 20
```

### 7.6 类型注解与类型推断

Swift是强类型的。有两种方式给予变量类型。一种方式是显式声明：
`var userCount: Int = 10`

另一种方式是通过初始化推断类型：

```
var signalStrength = 2.231
let companyName = "My Company"
```

### 7.7 类型强制转换与检查

The following code, for example, lets the compiler know that the value returned from the `objectForKey` method needs to be treated as a String type:
`let myValue = record.objectForKey("comment") as String`

`is`用于判定对象是否为特定类型。例如，检查对象是否是类`MyClass`的一个实例：

```
if myobject is MyClass {
}
```

### 7.8 Swift Tuple

元组，用于临时的将多个值组织成一项。例子：

```
let myTuple = (10, 432.433, “This is a String”)
```

元组中的单个元素可以通过下标访问：

```
let myTuple = (10, 432.433, "This is a String")
var myString = myTuple.2
println(myString)
```

元组中的值可以在一条语句中被抽出赋给多个变量或常量：

```
let (myInt, myFloat, myString) = myTuple
```

也可以抽出部分值。不关心的位置上放下划线占位，如：

```
var (myInt, _, myString) = myTuple
```

创建元素时可以为每个值分配一个名字：

```
let myTuple = (count: 10, length: 432.433, message: "This is a String")
```

其后，可以通过名字访问元组中的值：

```
println(myTuple.message)
```

元组最大的威力在于可以让函数返回多个值。

### 7.9 Swift Optional 类型

The purpose of the optional type is to provide a safe and consistent approach to handling situations where a variable or constant may not have any value assigned to it.

要吧变量声明为可选的，在其类型声明后添加一个`?`字符。例如声明一个可选的`Int`变量：

```
var index: Int?
```

现在`index`变量可能被赋予一个值，也可能没有。Behind the scenes, and as far as the compiler and runtime are concerned, an optional with no value assigned to it actually has a value of `nil`.

判定optional是否已被赋值：

```
var index: Int?
if index != nil {
    // index variable has a value assigned to it
} else {
    // index variable has no value assigned to it
}
```

If an optional has a value assigned to it, that value is said to be “wrapped” within the optional. 访问被optional包裹的值的方法称为强制解包。在optinal变量名后加`!`。

```
var index: Int?
index = 3
var treeArray = ["Oak", "Pine", "Yew", "Birch"]
if index != nil {
    println(treeArray[index!])
} else {
    println("index does not contain a value")
}
```

若不解包，编译器会报错，类似于：

	Value of optional type Int? not unwrapped

除了强制解包，optional的值可以通过**optional binding**分配给一个临时变量或常量，语法是：

```
if let constantname = optionalName {
    ...
}
if var variablename = optionalName {
}
```

上面的代码有两个目的。第一判定optional是否包含一个值。若有值，值赋给一个临时变量，**仅在if语句内可用**。The previous forced unwrapping example could, therefore, be modified as follows to use optional binding instead:

```
var index: Int?
index = 3
var treeArray = ["Oak", "Pine", "Yew", "Birch"]
if let myvalue = index {
    println(treeArray[myvalue])
} else {
    println("index does not contain a value")
}
```

可以声明optional是隐式解包的。此时，底层的值不必通过强制解包或optional binding就能访问。声明为隐式解包的，`?`替换为`!`。例如：

```
var index: Int! // Optional is now implicitly unwrapped
index = 3
var treeArray = ["Oak", "Pine", "Yew", "Birch"]
if index != nil {
    println(treeArray[index])
} else {
    println("index doex not contain a value")
}
```

只有optionals可被赋予`nil`。**其他变量或常量不能被赋予`nil`**。例如，下面的代码会报错：

```
var myInt = nil // Invalid code
var myString: String = nil // Invalid Code
let myConstant = nil // Invalid code
```

## 8. Swift运算符与表达式

```
x %= y
x && y
x || y

x++ // Increment x by 1
x-- // Decrement x by 1
```

NOT (!), AND (&&), OR (||) and XOR (^).

范围运算符。有两种。

`x...y`包括开头和结尾的x和y。例如`5...8`包括5, 6, 7, 8。

`x..<y`包括开头x但不包括结尾y。`5..<8`包括5, 6, 7。

三元运算符：

```
condition ? true expression : false expression
```

## 9. Swift流控制

Swift提供两类循环，传统的、和 for-in 循环。

传统形式：

```
for initializer; conditional expression; increment expression {
    // statements to be executed
}
```

例子：

```
for var i = 0; i < 10; ++i {
	println("Value of i is \(i)")
}
```

for-in用于遍历集合或范围。for-in循环的语法是：

```
for constant name in collection or range {
}
```

例子：

```
for index in 1...5 {
    println("Value of index is \(index)")
￼}
```

for-in可以用于任何包含多个元素的数据。例如，用于字符串：

```
for char in "This is a string" {
    println(char)
}
```

while循环语法：

```js
while condition {
}
```

例子：

```js
var myCount = 0
while myCount < 100 {
    ++myCount
}
```

**do while** 循环语法：

```
do {
    // Swift statements here
} while conditional expression
```

`break`，`continue`

```
var j = 10
for var i = 0; i < 100; ++i
{
    j += j
    if j > 100 {
        break
    }
    println("j = \(j)")
}
```

if语句的基本语法：

```
if boolean expression {
    // Swift code to be performed when expression evaluates to true
}
```

与其他语言不同的是，大括号`{}`对于Swift是关键的，即使只跟一行代码。

例子：

```
var x = 10
if x > 9 {
    println("x is greater than 9!")
}
```

## 10. Swift Switch

switch语句的基本语法是：

```
switch expression
{
    case match1:
        statements
    case match2:
        statements
    case match3, match4:
        statements
    default:
        statements
}
```

例子：

```
var value = 4
switch (value)
{
    case 0:
        println("zero")
    case 1:
        println("one")
    case 2:
        println("two")
    default:
        println("Integer out of range")
}
```

组合case：

```
var value = 1
switch (value)
{
    case 0, 1, 2:
        println("zero, one or two")
    case 3:
        println("three")
}
```

范围匹配：

```
var temperature = 83
switch (temperature)
{
    case 0...49:
        println("Cold")
    case 50...79:
        println("Warm")
    case 80...110:
        println("Hot")
    default:
        println("Temperature out of range")
}
```

The `where` statement may be used within a switch case match to add additional criteria required for a positive match. {{如果where判断失败，还会执行后面的case吗？}}

```
var temperature = 54
switch (temperature)
{
    case 0...49 where temperature % 2 == 0:
        println("Cold and even")
    case 50...79 where temperature % 2 == 0:
        println("Warm and even")
    case 80...110 where temperature % 2 == 0:
        println("Hot and even")
    default:
        println("Temperature out of range or odd")
}
```

Swift默认自动在每个case后break。其他语言的 fallthrough 效果可以通过 `fallthrough` 语句实现：

```
var temperature = 10
switch (temperature)
{
    case 0...49 where temperature % 2 == 0:
        println("Cold and even")
        fallthrough
    case 50...79 where temperature % 2 == 0:
        println("Warm and even")
        fallthrough
    case 80...110 where temperature % 2 == 0:
        println("Hot and even")
        fallthrough
    default:
        println("Temperature out of range or odd")
}
```

Although `break` is less commonly used in Swift switch statements, it is useful when no action needs to be taken for the `default` case. For example:

```
. . .
    default:
        break
}
```

## 11. Swift函数和闭包概述

### 11.2 声明函数

Swift函数声明语法：

	func <function name> (<para name>: <para type>, <para name>: <para type>, ... ) -> <return type> {
		// Function code
	}

例子，无参数无返回值的函数：

```
func sayHello() {
	println("Hello")
}
```

例2：

```
func buildMessage(name: String, count: Int) -> String {
    return("\(name), you are customer number \(count)")
}
```

### 11.3 调用

函数调用语法：

	<function name> (<arg1>, <arg2>, ... )

例子：

```
sayHello()

var message = buildMessage("John", 100)
```

### 11.4 声明外部参数名

为了让代码更易读、更加自我解释，调用函数时最好能引用函数名。将参数名声明为外部的（external），在函数声明中，参数名前加前缀`#`。例如：

```
func buildMessage(#name: String, #count: Int) -> String {
```

有了外部名，调用时即可引用它：

```
var message = buildMessage(name: "John", count: 100)
```

若内部名和外部名需要不通，语法是先声明外部名，在声明内部名：

```
func buildMessage(customerName name: String, customerCount count: Int) -> String {
	return ("\(name), you are customer number \(count)")
}
var message = buildMessage(customerName: "John", customerCount: 100)
```

### 11.5 声明默认参数

对于默认参数，若未显式指定外部名，Swift将自动根据内部名产生外部名，后续调用函数时必须使用该名字：

```
func buildMessage(count: Int, name: String = "Customer") -> String {
	return ("\(name), you are customer number \(count)")
}
```

现在函数调用时可以省略默认参数：

```
var message = buildMessage(100)
println(message)
```

调用时若不想使用默认`customer`值，即显式传入它的值，此时必须引用外部名：

```
var message = buildMessage(100, name: "John")
```

### 11.6 函数返回多个值

通过元组函数可以返回多个值。

```
func sizeConverter (length: Float) -> (yards: Float, centimeters: Float, meters: Float) {
    var yards = length * 0.0277778
    var centimeters = length * 2.54
    var meters = length * 0.0254
    return (yards, centimeters, meters)
}
```

调用：

```
var lengthTuple = sizeConverter(20)
println(lengthTuple.yards)
```

### 11.7 可变数量参数

可变数量参数（Variadic parameters），其类型后跟三个点，表示参数接受零到多个值。在函数内部，参数是一个数组对象：

```
func displayStrings(strings: String...) {
    for string in strings {
        println(string)
    }
}
displayStrings("one", "two", "three", "four")
```

### 11.8 参数作为变量

函数参数默认为常量，防止函数内修改参数的值。若需要在函数内修改参数的值，参数必须声明为变量。

```
func calcuateArea (var length: Float, var width: Float) -> Float {
    length = length * 2.54
    width = width * 2.54
    return length * width
}
println(calcuateArea(10, 20))
```

### 11.9 In-Out参数

在函数内修改参数值，也要改变形参的值（原值），需要将参数声明为in-out参数。声明时加关键字`inout`：

```
func doubleValue (inout value: Int) -> Int {
    value += value
    return(value)
}
```

调用时，inout参数形参必须前缀`&`：

```
println("doubleValue call returned \(doubleValue(&myValue))")
```

### 11.10 函数作为参数或返回值

可以将函数赋给一个常量或变量：

```
func inchesToFeet (inches: Float) -> Float {
    return inches * 0.0833333
}
let toFeet = inchesToFeet

var result = toFeet(10)
```

函数的类型包括其参数的类型和返回值的类型。里面上面`inchesToFeet`函数的类型是`(Float) -> Float`。

函数的参数为一个函数：

```
func inchesToFeet (inches: Float) -> Float {
    return inches * 0.0833333
}
func inchesToYards (inches: Float) -> Float {
    return inches * 0.0277778
}
let toFeet = inchesToFeet
let toYards = inchesToYards

func outputConversion(converterFunc: (Float) -> Float, value: Float) {
    var result = converterFunc(value)
    println("Result of conversion is \(result)")
}

outputConversion(toYards, 10) // Convert to Yards
outputConversion(toFeet, 10) // Convert to Inches
```

函数作为返回值：

```
func decideFunction (feet: Bool) -> (Float) -> Float {
    if feet {
        return toFeet
    } else {
        return toYards
    }
}
```

### 11.11 闭包表达式

闭包表达式是自包含的代码块。下面的代码，声明一个闭包表达式，然后将其分配给常量`sayHello`，然后通过该常量调用函数：

```
let sayHello = { println("Hello") }
sayHello()
```

闭包表达式也可以接收函数、返回结果。语法：

```
    {(<para name>: <para type>, <para name> <para type>, ... ) -> <return type> in
        // Closure expression code here
    }
```

例如，下面的闭包表达式，接收两个整数，返回一个整数：

```
let multiply = {(val1: Int, val2: Int) -> Int in
    return val1 * val2
}
let result = multiply(10, 20)
```

闭包表达式长用于做回调函数：

```
eventStore?.requestAccessToEntityType(EKEntityTypeReminder, completion: {(granted: Bool, error: NSError!) -> Void in
    if !granted {
        println(error.localizedDescription)
    }
})
```

在上面这个例子中，实际上编译器能够推测出闭包表达式的参数的返回值，因为它们可以省略：

```
eventStore?.requestAccessToEntityType(EKEntityTypeReminder, completion: {(granted, error) in
    if !granted {
        println(error.localizedDescription)
    }
})
```

### 11.12 Swift中的闭包

通常意义上的闭包，指自包含的代码块（如函数或闭包表达式）加一个或多个变量存在于围绕代码块的上下文。例如，下面这个Swift函数：

```
func functionA() -> () -> Int {
    var counter = 0
    func functionB() -> Int {
        return counter + 10
    }
    return functionB
}
let myClosure = functionA()
let result = myClosure()
```

`functionA`返回另一个函数`functionB`。`functionB`实际是一个闭包，因为它依赖其作用域之外的`counter`变量。

某种程度上，闭包和闭包表达式这两个名词可以互换使用。关键是，Swift二者都支持。

## 12. 基本面向对象编程

类方法前面加`class`关键字：

```
class BankAccount {
    var accountBalance: Float = 0
    var accountNumber: Int = 0
    func displayBalance() {
        println("Number \(accountNumber)")
        println("Current balance is \(accountBalance)")
    }
    class func getMaxBalance() -> Float {
        return 100000.00
    }
}
```

创建类实例：

```
var account1: BankAccount = BankAccount()
```

定义实例的初始化方法：

```
class BankAccount {
    var accountBalance: Float = 0
    var accountNumber: Int = 0
    init(number: Int, balance: Float) {
        accountNumber = number
        accountBalance = balance
    }
    func displayBalance() {
        println("Number \(accountNumber)")
        println("Current balance is \(accountBalance)")
    }
}
```

此时，实例化需要传入需要的参数：

```
var account1 = BankAccount(number: 12312312, balance: 400.54)
```

Swift运行时系统销毁实例时，可以执行预定义的销毁方法：

```
class BankAccount {
    var accountBalance: Float = 0
    var accountNumber: Int = 0
    init(number: Int, balance: Float) {
        accountNumber = number
        accountBalance = balance
    }
    deinit {
        // Perform any necessary clean up here
    }
}
```

### 12.9 存储的（Stored）和计算的（Computed）属性

属性分为两类：存储的属性和计算的属性。存储的属性就是正常的，值存放在一个变量或常量中的属性。

计算的属性从其他属性派生而出。计算属性包含一个 getter 和一个可选的 setter 方法。例子：

```
class BankAccount {
    var accountBalance: Float = 0
    var accountNumber: Int = 0;
    let fees: Float = 25.00
    var balanceLessFees: Float {
        get {
            return accountBalance - fees
        }
    }
    init(number: Int, balance: Float) {
        accountNumber = number
        accountBalance = balance
    }
    . . .
}
```

含setter方法：

```
    var balanceLessFees: Float {
    	get {
    		return accountBalance - fees
        }
    	set(newBalance) {
    		accountBalance = newBalance - fees
    	}
    }
```

计算的属性与普通存储的属性的访问方式相同：

```
var balance1 = account1.balanceLessFees
account1.balanceLessFees = 12123.12
```

### 12.10 self

加前缀`self`访问属性或方法，表示属性或方法属于当前类的实例。

```
class MyClass {
    var myNumber = 1
    func addTen() {
        self.myNumber += 10
    }
}
```

多数情况`self`可以省略，如

```
func addTen() {
    myNumber += 10
}
```

The use of `self`, for example, is mandatory in the following closure expression:

```
document?.openWithCompletionHandler({(success: Bool) -> Void in
    if success {
        self.ubiquityURL = resultURL
    }
})
```

## 13. Swift继承介绍

创建`BankAccount`的子类`SavingsAccount`：

```
class SavingsAccount: BankAccount { }
```

扩展子类功能：

```
class SavingsAccount: BankAccount {
    var interestRate: Float
    func calculateInterest() -> Float {
        return interestRate * accountBalance
    }
}
```

### 13.4 覆盖继承的方法

覆盖的方法必须与父方法具有相同的参数（数量和类型），相同的返回值。

例子，覆盖父类`BankAccount`的`displayBalance`方法。注意前缀`override`关键字。

```
class SavingsAccount: BankAccount {
    var interestRate: Float
    func calculateInterest() -> Float {
        return interestRate * accountBalance
    }
    override func displayBalance() {
        println("Number \(accountNumber)")
        println("Current balance is \(accountBalance)")
        println("Prevailing interest rate is \(interestRate)")
    }
}
```

可以在子类的覆盖方法中调用父类被覆盖的方法，使用`super`关键字：

```
override func displayBalance() {
    super.displayBalance()
    println("Prevailing interest rate is \(interestRate)"
}
```

### 13.5 初始化子类

`SavingsAccount`目前继承的时父类`BankAccount`的初始化器。但创建`SavingsAccount`类实例时，我们还需要初始化`SavingsAccount`扩展的属性。为此我们需要重新定义构造器。在新构造器中，调用父类的构造器。

```
class SavingsAccount: BankAccount {
    var interestRate: Float
    init(number: Int, balance: Float, rate: Float) {
        interestRate = rate
        super.init(number: number, balance: balance)
    }
    . . .
}
```

### 13.6 使用`SavingsAccount`类

```
var savings1 = SavingsAccount(number: 12311, balance: 600.00, rate: 0.07)
println(savings1.calculateInterest())
savings1.displayBalance()
```

## 14. 数组与字典集合

Swift中的集合（Collection）有可变和不可变两种。不可变集合的内容在初始化后是不能变的。不可变的集合，即赋给常量的集合；可变的集合，即赋给一个变量的。

### 14.1 Swift数组初始化

数组可以用数组字面量初始化：

```
var variableName: [type] = [value 1, value2, value3, ....... ]
```

例子，一个可变数组：

```
var treeArray = ["Pine", "Oak", "Yew"]
```

同样的数组，赋给一个常量就不可变了：

```
let treeArray = ["Pine", "Oak", "Yew"]
```

上面两个声明，编译器使用类型推断确定变量类型。也可以显式声明类型：

```
var treeArray: [String] = ["Pine", "Oak", "Yew"]
```

数组变量声明时可以不必初始化。下面的语法可以创建一个空数组：

```
var variableName = [type]()
```

例子：

```
var priceArray = [Float]()
```

或者，初始化数组到指定长度，使用指定值初始化每个元素：

```
var nameArray = [String](count: 10, repeatedValue: "My String")
```

若两个数组元素类型相同，可以相加得到一个新数组：

```
var firstArray = ["Red", "Green", "Blue"]
var secondArray = ["Indigo", "Violet"]
var thirdArray = firstArray + secondArray
```

### 14.3 使用Swift数组

数组元素个数可以通过`count`属性获取：

```
var treeArray = ["Pine", "Oak", "Yew"]
var itemCount = treeArray.count
println(itemCount)
```

检查数组是否为空通过`isEmpty`属性：

```
var treeArray = ["Pine", "Oak", "Yew"]
if treeArray.isEmpty {
    // Array is empty
}
```

访问指定位置上的元素：

```
var treeArray = ["Pine", "Oak", "Yew"]
println(treeArray[2])

treeArray[1] = "Redwood"
```

向数组添加元素，可以使用`append`方法，或`+`、`+=`运算符。{{！两个方法结果不同吧！第二种方法会产生新数组？！}}

    treeArray.append("Redwood")
    treeArray += ["Redwood"]
    treeArray += ["Redwood", "Maple", "Birch"]

向指定位置插入元素通过`insert(atIndex:)`方法：

```
treeArray.insert("Maple", atIndex: 0)
```

移除特定位置上的元素：

```
treeArray.removeAtIndex(2)
```

移除最后一个位置上的元素：

```
treeArray.removeLast()
```

遍历数组：

```
var treeArray = ["Pine", "Oak", "Yew", "Maple", "Birch", "Myrtle"]
for tree in treeArray {
    println(tree)
}
```

### 14.5 Swift字典

目前，只有`String`、`Int`、`Double`和`Bool`可以用于做字典的键。

新字典，可以用字典字面量初始化：

```
var variableName: [key type: value type] = [key 1: value 1, key 2: value2 .... ]
```

例子：

```
var bookDict = ["100-432112" : "Wind in the Willows",
    "200-532874" : "Tale of Two Cities",
    "202-546549" : "Sense and Sensibility",
    "104-109834" : "Shutter Island"]
```

或，显式指出键和值的类型：

```
var bookDict: [String: String] = ["100-432112" : "Wind in the Willows",
    "200-532874" : "Tale of Two Cities",
    "202-546549" : "Sense and Sensibility",
    "104-109834" : "Shutter Island"]
```

与数组语法类型，可以创建空字典，如：

```
var myDictionary = [Int: String]()
```

通过`count`获取字典中项数：

```
println(bookDict.count)
```

根据键获取值：

```
println(bookDict["200-532874"])
```

更新值：

```
bookDict["200-532874"] = "Sense and Sensibility"
```

或通过：

```
bookDict.updateValue("The Ruins", forKey: "200-532874")
```

移除一个键值对，可以通过赋`nil`值，或调用`removeValueForKey`方法实现：

```
bookDict["300-898871"] = nil
bookDict.removeValueForKey("300-898871")
```

遍历字典：

```
for (bookid, title) in bookDict {
    println("Book ID: \(bookid) Title: \(title)")
}
```