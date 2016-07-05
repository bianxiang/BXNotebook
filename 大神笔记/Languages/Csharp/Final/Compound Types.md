[toc]

## 2. 复杂类型

### 2.1 枚举

利用`enum`关键字定义枚举：

    enum <typeName>
    {
        <value1>,
        <value2>,
        <value3>,
        ...
        <valueN>
    }

接下来可以声明此种类型的变量：

	<typeName> <varName>;

赋值：

	<varName> = <typeName>.<value>;

枚举有一个底层的类型用于存储。每个枚举值对应一个底层类型的值。若不指定默认是`int`。或显式指定：

    enum <typeName> : <underlyingType>
    {
        <value1>,
        <value2>,
        <value3>,
        ...
        <valueN>
    }

底层类型可以是数值：byte, sbyte, short, ushort, int, uint, long和ulong。默认枚举值按其定义顺序分别被赋予0、1、2等值。可以覆盖底层值：

    enum <typeName> : <underlyingType>
    {
        <value1> = <actualVal1>,
        <value2> = <actualVal2>,
        <value3> = <actualVal3>,
        ...
        <valueN> = <actualValN>
	}

多个枚举值可以共用一个底层值：

    enum <typeName> : <underlyingType>
    {
        <value1> = <actualVal1>,
        <value2> = <value1>,
        <value3>,
        ...
        <valueN> = <actualValN>
    }

Any values left unassigned are given an underlying value automatically, whereby the values used are in a sequence starting from 1 greater than the last explicitly declared one. In the preceding code, for example, `<value3>` will get the value `<value1> + 1`.

Note that this can cause problems, with values specified after a definition such as `<value2> = <value1>` being identical to other values. For example, in the following code `<value4>` will have the same value as `<value2>`:

    enum <typeName>: <underlyingType>
    {
        <value1> = <actualVal1>,
        <value2>,
        <value3> = <value1>,
        <value4>,
        ...
        <valueN> = <actualValN>
    }

Of course, if this is the behavior you want, then this code is fine. Note also that assigning values in a circular fashion will cause an error:

    enum <typeName>: <underlyingType>
    {
        <value1> = <value2>,
        <value2> = <value1>
    }

The following Try It Out shows an example of all of this. The code defines and then uses an enumeration called orientation.

枚举类型变量可以与底层类型变量显式类型转换：

    directionByte = (byte)myDirection;
    myDirection = (orientation)myByte;

可以获得枚举值的字符串，使用`Convert.ToString()`或`ToString()`命令：

	directionString = Convert.ToString(myDirection);
	directionString = myDirection.ToString();

字符串可以转换成枚举，使用`Enum.Parse()`：

	(enumerationType)Enum.Parse(typeof(enumerationType), enumerationValueString);

用到了运算符`typeof`。


### 2.2 结构

使用`struct`关键字定义结构体：

    struct <typeName>
    {
    	<memberDeclarations>
    }

`<memberDeclarations>`包含结构体的数据成员。每个成员：

	<accessibility> <type> <name>;

To allow the code that calls the struct to access the struct’s data members, you use the keyword `public` for `<accessibility>`. For example:

    struct route
    {
        public orientation direction;
        public double distance;
    }

定义结构体的变量：

	route myRoute;

访问：

    myRoute.direction = orientation.north;
    myRoute.distance  = 2.5;


### 2.3 数组

数组下标从0开始。数组中的元素类型必须相同。

#### 声明数组

数组声明：

	<baseType>[] <name>;

`<baseType>`可以是任何类型，如枚举和结构体。

数组初始化有两种形式，指定初始化列表，或指定大小。

	int[] myIntArray = { 5, 9, 10, 2, 99 };
	int[] myIntArray = new int[5];

注意`new`关键字。第二种形式，数组所有元素都被初始化为**默认值**（对于数字是0）

指定大小时，长度可以不是常量：

	int[] myIntArray = new int[arraySize];

或者，在指定长度的同时，给出初始化列表。此时指定长度必须与列表长度匹配。

	int[] myIntArray = new int[5] { 5, 9, 10, 2, 99 };

而且常量变量必须是常量：

    const int arraySize = 5;
    int[] myIntArray = new int[arraySize] { 5, 9, 10, 2, 99 };

若省略了const关键字，将报错。

与其他变量一样，没必要一定要在初始化时赋值：

    int[] myIntArray;
    myIntArray = new int[5];

访问数组长度：`friendNames.Length`。

#### foreach

    foreach (string friendName in friendNames)
    {
    	Console.WriteLine(friendName);
    }

##### 多维数组

{{C#支持直接多维数组，不是数组的数组。}}

二维数组：

	<baseType>[,] <name>;

更多维度：

	<baseType>[,,,] <name>;

例子，声明和初始化二维数组：

	double[,] hillHeight = new double[3,4];

利用字面量初始化二维数组：

	double[,] hillHeight = { { 1, 2, 3, 4 }, { 2, 3, 4, 5 }, { 3, 4, 5, 6 } };

访问数组元素：

	hillHeight[2,1]

The foreach loop gives you access to all elements in a multidimensional way, just as with single-dimensional arrays:

	double[,] hillHeight = { { 1, 2, 3, 4 }, { 2, 3, 4, 5 }, { 3, 4, 5, 6 } };
	foreach (double height in hillHeight)
	{
		Console.WriteLine("{0}", height);
	}



#### 数组的数组

数组的数组可以实现第二维数组长度不同。

	int[][] jaggedIntArray;

但初始化数组的数组不简单，例如不能：

	jaggedIntArray = new int[3][4];

也不能：

	jaggedIntArray = { { 1, 2, 3 }, { 1 }, { 1, 2 } };

有两个选择。或者一层层初始化：

    jaggedIntArray = new int[2][];
    jaggedIntArray[0] = new int[3];
    jaggedIntArray[1] = new int[4];

或是使用另一种形式的列表：

	jaggedIntArray = new int[3][] { new int[] { 1, 2, 3 }, new int[] { 1 }, new int[] { 1, 2 } };

用foreach逐层分解：

    foreach (int[] divisorsOfInt in divisors1To10)
    {
    	foreach(int divisor in divisorsOfInt)
    	{
    		Console.WriteLine(divisor);
    	}
    }

### 2.4 字符串

`string`可以被看成只读的字符数组。

	string myString = "A string";
	char myChar = myString[1];

利用`ToCharArray()`命令可以将其变成真正的字符数组：

	string myString = "A string";
	char[] myChars = myString.ToCharArray();

可以用`foreach`遍历字符：

    foreach (char character in myString)
    {
    	Console.WriteLine("{0}", character);
    }

利用`myString.Length`获取字符串长度（字符数）。

其他的，`<string>.ToLower()`和`<string>.ToUpper()`；

`<string>.Trim()`。还可以用来移除任意其他字符：

    char[] trimChars = {' ', 'e', 's'};
    string userResponse = Console.ReadLine();
    userResponse = userResponse.ToLower();
    userResponse = userResponse.Trim(trimChars);

还有`<string>.TrimStart()`和`<string>.TrimEnd()`。



