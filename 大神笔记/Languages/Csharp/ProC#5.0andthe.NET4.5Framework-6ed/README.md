[toc]

## 5. 理解封装

### 5.1 类类型

在Car.cs中，定义：

    class Car {
    	public string petName;
	    public int currSpeed;
        // A custom default constructor.
	    public Car() {
        	petName = "Chuck";
            currSpeed = 10;
        }

	    public Car(string pn) {
            petName = pn;
        }

        public Car(string pn, int cs) {
            petName = pn;
            currSpeed = cs;
        }
        ...
    }

    static void Main(string[] args) {
    	Car chuck = new Car();
    	chuck.PrintState();

	    Car mary = new Car("Mary");
        mary.PrintState();

		Car daisy = new Car("Daisy", 75);
        daisy.PrintState();
        ...
    }

### 5.2 理解构造器

默认构造器没有参数。确保类中字段都被赋予合适的默认值。若不定义其他构造器，会自动创建一个默认构造器。但如果提供了其他构造器，不会自动创建默认构造器。

自定义多个构造器：

    public Car() {
    	petName = "Chuck";
        currSpeed = 10;
    }

    public Car(string pn) {
    	petName = pn;
    }

    public Car(string pn, int cs) {
    	petName = pn;
        currSpeed = cs;
    }

> Visual Studio IDE中，键入`ctor`并按两下Tab。IDE将自动定义默认构造器。

### 5.3 this关键字

    public void SetDriverName(string name) {
    	this.name = name;
    }

#### 利用this链式调用构造器

如果类有多个构造器，可以把参数最多的作为主构造器。其他构造器利用this调用这个构造器。这种技术称为构造器链。

    public Motorcycle() {}

    public Motorcycle(int intensity)
    	: this(intensity, "") {}

    public Motorcycle(string name)
    	: this(0, name) {}

    // This is the 'master' constructor that does all the real work.

    public Motorcycle(int intensity, string name) {
    	if (intensity > 10) {
        	intensity = 10;
        }

        driverIntensity = intensity;
        driverName = name;
    }

调用非主构造器时，构造器先请主构造器执行，然后再执行自己。

#### 再议可选实参

假如有构造器

    public Motorcycle(int intensity = 0, string name = "") {
    	if (intensity > 10) {
        	intensity = 10;
	    }
        driverIntensity = intensity;
        driverName = name;
    }

上述构造器接受零个、一个或两个实参。利用命名参数可以跳过有默认值的参数。

    Motorcycle m1 = new Motorcycle();
    Motorcycle m2 = new Motorcycle(name:"Tiny");
    Motorcycle m3 = new Motorcycle(7);

### 5.4 static关键字

利用`static`定义静态成员。此时成员要通过类访问，不能通过对象访问。

例如，不能：

	Console c = new Console();
    c.WriteLine("I can't be printed...");

而要：

	Console.WriteLine("Much better! Thanks...");

`static`关键字可以施加到：类的数据成员、类的方法、类的属性、构造器或整个类定义。

#### 定义静态数据成员

	public static double currInterestRate = 0.04;

#### 定义静态方法

	public static double currInterestRate = 0.04;
    public static void SetInterestRate(double newRate) {
    	currInterestRate = newRate;
    }
    public static double GetInterestRate() {
    	return currInterestRate;
    }

#### 定义静态构造器

典型构造器用于初始化实例级别的数据。静态构造器用于初始化静态成员变量。

    public static double currInterestRate;

    static SavingsAccount() {
        Console.WriteLine("In static ctor!");
        currInterestRate = 0.04;
    }

静态构造器由CLR自动调用。只会被调用一次。

一个类只能定义一个静态构造器。静态构造器不能加访问控制符，也不能带参数。

#### 定义静态类

类可以被定义为静态的。此时类只能包含静态成员。不能实例化静态类。

    static class TimeUtilClass {
    	public static void PrintTime() {
        	Console.WriteLine(DateTime.Now.ToShortTimeString());
        }
        public static void PrintDate() {
            Console.WriteLine(DateTime.Today.ToShortDateString());
        }
    }

### 5.6 C#访问修饰符

- `public`：可以用于类型或类型中的成员。无访问限制。
- `private`：用于类型成员或嵌套类型。只能被所在类访问。
- `protected`：用于类型成员或嵌套类型。被所在类或子类访问。However, protected items cannot be accessed from the outside world using the C# dot operator.
- `internal`：用于类型或类型成员。Internal items are accessible only within the current assembly. Therefore, if you define a set of internal types within a .NET class library, other assemblies are not able to make use of them.
- `protected internal`：用于类型成员或嵌套类型。When the protected and internal keywords are combined on an item, the item is accessible within the defining assembly, the defining class, and by derived classes.

类型成员默认是private，类型默认是internal。

As mentioned in Table 5-1, the private, protected, and protected internal access modifiers can be applied to a nested type. By way of example, here is a private enumeration (named `CarColor`) nested within a public class (named SportsC``ar):

    public class SportsCar {
    	private enum CarColor {
        	Red, Green, Blue
        }
    }

### 5.7 封装

#### 使用getter和setter封装

#### 使用.NET属性封装

    class Employee {
    	// 字段
        private string empName;
        private int empID;
        private float currPay;
        // 属性
	    public string Name {
        	get { return empName; }
            set {
            	if (value.Length > 15)
                	Console.WriteLine("Error! Name must be less than 16 characters!");
    			else
                	empName = value;
            }
    	}
	    // We could add additional business rules to the sets of these properties;
        // however, there is no need to do so for this example.
        public int ID {
        	get { return empID; }
            set { empID = value; }
        }
        public float Pay {
        	get { return currPay; }
            set { currPay = value; }
        }
        ...
    }

在属性的“set”中，可以使用符号`value`，表示调用者传入的值。这个符号不是真正的C#关键它被字，它被称作contextual keyword。

访问或修改属性使用点运算符。

    // Set and get the Name property.
    emp.Name = "Marv";
    Console.WriteLine("Employee is named: {0}", emp.Name);

可以对属性施加C#内建的运算符。例如，使用getter和setter实现属性增1：

	Employee joe = new Employee();
	joe.SetAge(joe.GetAge() + 1);

但如果`Age`是属性，可以：

	Employee joe = new Employee();
	joe.Age++;

#### 在类定义中使用属性

在类中使用属性，例如在构造器中给属性赋初始值，如`Name = "aa"`，也将调用属性的set方法。

#### 只读属性和只写属性
 
When encapsulating data, you might want to configure a read-only property. To do so, simply omit the 
setblock. Likewise, if you want to have a write-only property, omit the getblock. For example, assume 
you wanted a new property named SocialSecurityNumber, which encapsulates a private stringvariable 
named empSSN. If you want to make this a read-only property, you could write: 
public string SocialSecurityNumber 
{ 
get { return empSS; } 
} 
Now assume our class constructor has a new parameter to let the caller set the SSN of the object. 
Since the SocialSecurityNumberproperty is read only, we cannot set the value as so: 
public Employee(string name, int age, int id, float pay, string ssn) 
{ 
Name = name; 
Age = age; 
ID = id; 
Pay = pay; 
// OOPS! This is no longer possible if the property is read 
// only. 
SocialSecurityNumber = ssn; 
} 
Unless we are willing to redesign the property as read/write, your only choice would be to use the 
underlying empSSNmember variable within your constructor logic as so: 
public Employee(string name, int age, int id, float pay, string ssn) 
{ 
... 
empSSN = ssn; 
} 
To wrap up the story thus far, recall that C# prefers properties to encapsulate data. These syntactic 
entities are used for the same purpose as traditional accessor (get)/mutator (set) methods. The benefit 
of properties is that the users of your objects are able to manipulate the internal data point using a single 
named item. 


