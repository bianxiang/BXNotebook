[toc]

## 7. 调试和错误处理

### （未）7.1 在VS中调试

### 7.2 错误处理

异常定义在命名空间中，如`System.IndexOutOfRangeException`。

#### 7.2.1 try…catch…finally

    try
    {
    	...
    }
    catch (<exceptionType> e)
    {
    	...
    }
    finally
    {
    	...
    }

若catch中`<exceptionType> e`省略，则表示捕获所有异常。

例子：

    try
    {
        ThrowException(eType);
    }
    catch (System.IndexOutOfRangeException e) // Line 24
    {
        Console.WriteLine("Main() System.IndexOutOfRangeException catch"
            + " block reached. Message:\n\"{0}\"", e.Message);
    }
    catch // Line 30
    {
        Console.WriteLine("Main() general catch block reached.");
    }
    finally
    {
        Console.WriteLine("Main() finally block reached.");
    }

	static void ThrowException(string exceptionType)
	{
		Console.WriteLine("ThrowException(\"{0}\") reached.", exceptionType);
		switch (exceptionType)
		{
			case "none":
				Console.WriteLine("Not throwing an exception.");
				break; // Line 50
			case "simple":
				Console.WriteLine("Throwing System.Exception.");
				throw new System.Exception(); // Line 53
            case "index":
				Console.WriteLine("Throwing System.IndexOutOfRangeException.");
				eTypes[4] = "error"; // Line 56
				break;
			case "nested index":
				try // Line 59
				{
					Console.WriteLine("ThrowException(\"nested index\") " +
						"try block reached.");
					Console.WriteLine("ThrowException(\"index\") called.");
					ThrowException("index"); // Line 64
                }
				catch // Line 66
                {
					Console.WriteLine("ThrowException(\"nested index\") general"
						+ " catch block reached.");
				}
				finally
				{
					Console.WriteLine("ThrowException(\"nested index\") finally"
						+ " block reached.");
				}
				break;
			}
		}
	}

利用`throw`关键字抛出异常。

#### （未）7.2.2 列出和配置异常

#### （未）7.2.3 异常处理备注




