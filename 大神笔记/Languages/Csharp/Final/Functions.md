[toc]

## 3. 函数

### 3.1 定义函数

    class Program
    {
        static void Write()
        {
        	Console.WriteLine("Text output from function.");
        }
        static void Main(string[] args)
        {
        	Write();
        	Console.ReadKey();
        }
    }

> `static`与面向对象有关，后面讲。

> 函数名使用PascalCase。

#### 3.1.1 返回值

    static <returnType> <FunctionName>()
    {
        ...
        return <returnValue>;
    }

例子：

    static double GetVal()
    {
    	return 3.2;
    }

#### 3.1.2 参数

    static <returnType> <FunctionName>(<paramType> <paramName>, ...)
    {
    	...
    	return <returnValue>;
    }

例子：

    static double Product(double param1, double param2)
    {
    	return param1 * param2;
    }

##### 参数数组

实现可变参数函数：一个特殊的参数，必须是参数列表中最后一个参数，加`params`关键字，称为参数数组。实参可以不是数组，可以一个个值传入。

    static <returnType> <FunctionName>(<p1Type> <p1Name>, ..., params <type>[] <name>)
    {
    	...
    	return <returnValue>;
    }

例子：

    class Program
    {
        static int SumVals(params int[] vals)
        {
        	int sum = 0;
            foreach (int val in vals)
            {
            	sum += val;
            }
            return sum;
        }
        static void Main(string[] args)
        {
            int sum = SumVals(1, 5, 2, 9, 8);
            Console.WriteLine("Summed Values = {0}", sum);
            Console.ReadKey();
        }
    }

> C# version 4 introduced alternative ways to specify function parameters, including a far more readable way to include optional parameters. You will learn about these methods in Chapter 14, which looks at how the C# language has evolved since it was created.

##### 引用和值参数

之前使用都是值参数。还可以传入引用。加`ref`关键字：

    static void ShowDouble(ref int val)
    {
    	val *= 2;
    	Console.WriteLine("val doubled = {0}", val);
    }

调用函数的时候也要加`ref`关键字：

    int myNumber = 5;
    Console.WriteLine("myNumber = {0}", myNumber);
    ShowDouble(ref myNumber);
    Console.WriteLine("myNumber = {0}", myNumber);

输出：

    myNumber = 5
    val doubled = 10
    myNumber = 10

两个限制：1、由于函数内可能改变引用参数。因此实参不能是常量。2、必须使用已初始化的变量。于是，不要指望可以在函数内初始化引用参数。

##### Out参数

加`out`关键字将参数标记为out参数。Out参数与引用参数类似，但也有很大区别：

- 未赋值的变量不能做引用参数；但却可以作为Out参数。
- 使用它的函数必须当Out参数还未被赋值。即虽然可以让已被赋值的变量做Out参数，但当函数执行时，变量中的值丢失。

例子：

    static int MaxValue(int[] intArray, out int maxIndex)
    {
        int maxVal = intArray[0];
        maxIndex = 0;
        for (int i = 1; i < intArray.Length; i++)
        {
            if (intArray[i] > maxVal)
            {
            	maxVal = intArray[i];
            	maxIndex = i;
            }
        }
        return maxVal;
    }

函数调用时也需要添加`out`关键字：

    int[] myArray = { 1, 8, 3, 6, 2, 5, 9, 3, 0, 2 };
    int maxIndex;
    Console.WriteLine("The maximum value in myArray is {0}", MaxValue(myArray, out maxIndex));
    Console.WriteLine("The first occurrence of this value is at element {0}", maxIndex + 1);

### 3.2 Struct函数

    struct CustomerName
    {
    	public string firstName, lastName;
        public string Name()
        {
        	return firstName + " " + lastName;
        }
    }

调用：

    CustomerName myCustomer;
    myCustomer.firstName = "John";
    myCustomer.lastName = "Franklin";
    Console.WriteLine(myCustomer.Name());

### 3.3 重载函数

返回值不算签名。

### 3.4 delegate

**delegate**是一个类型，可以存储到函数的引用。其意义在本书后面将事件处理器时会显现。Delegate声明很像一个函数声明，但没有函数体，且加`delegate`关键字。delegate声明时需要指定返回值类型和参数列表。

定义delegate类型后，可以声明一个它的变量。可以把一个函数赋给这个变量，要求有相同的返回值类型和参数列表。最后，你可以使用delegate类型的变量调用一个函数。

    class Program
    {
        delegate double ProcessDelegate(double param1, double param2);
        static double Multiply(double param1, double param2)
        {
        	return param1 * param2;
        }
        static double Divide(double param1, double param2)
        {
        	return param1 / param2;
        }
        static void Main(string[] args)
        {
            ProcessDelegate process; // 声明Delegate类型的变量
            Console.WriteLine("Enter 2 numbers separated with a comma:");
            string input = Console.ReadLine();
            int commaPos = input.IndexOf(',');
            double param1 = Convert.ToDouble(input.Substring(0, commaPos));
            double param2 = Convert.ToDouble(input.Substring(commaPos + 1,
            	input.Length - commaPos - 1));
            Console.WriteLine("Enter M to multiply or D to divide:");
            input = Console.ReadLine();
            if (input == "M")
            	process = new ProcessDelegate(Multiply);
            else
            	process = new ProcessDelegate(Divide);
            Console.WriteLine("Result: {0}", process(param1, param2)); // 调用
            Console.ReadKey();
        }
    }

Delegate变量的赋值支持更简单的语法：

    if (input == "M")
    	process = Multiply;
    else
    	process = Divide;

利用Delegate类型向方法传递函数：

    static void ExecuteFunction(ProcessDelegate process)
    {
    	process(2.2, 3.3);
    }
