[toc]

## Part II. 类型与运算符

## 4. Python 对象类型入门

Python中一些都是对象。包括数字，运算符。

### Python核心数据结构

包括：

- 数字：`1234, 3.1415, 3+4j, 0b111, Decimal(), Fraction()`
- 字符串：`'spam', "Bob's", b'a\x01c', u'sp\xc4m'`
- List：`[1, [2, 'three'], 4.5], list(range(10))`
- Dictionary：`{'food': 'spam', 'taste': 'yum'}, dict(hours=10)`
- Tuple：`(1, 'spam', 4, 'U'), tuple('spam'), namedtuple`
- 文件：`open('eggs.txt'), open(r'C:\ham.bin', 'wb')`
- Set：`set('abc'), {'a', 'b', 'c'}`
- 其他核心类型（布尔、类型、`None`）
- 程序单元类型（函数、模块、类）

注意到，函数、模块、类也是对象；可以被传递。

Just as importantly, once you create an object, you bind its operation set for all time — you can perform only string operations on a string and list operations on a list. 即这意味着Python是**动态类型**的，a model that keeps track of types for you automatically instead of requiring declaration code, 但它也是**强类型**的，a constraint that means you can perform on an object only operations that are valid for its type.

### 数字

Python 3.X’s integer type automatically provides extra precision for large numbers like this when needed (in 2.X, a separate long integer type handles numbers too large for the normal integer type in similar ways).

在 2.7 或 3.1之前，使用浮点数时可能发现：

    >>> 3.1415 * 2  # repr: as code (Pythons < 2.7 and 3.1)
    6.2830000000000004
    >>> print(3.1415 * 2)  # str: user-friendly
    6.283

第一个结果不是Bug。是显示问题。在Python中有两种打印对象的方法：完整精度和友好形式。第一个形式被称为对象的**as-code repr**，第二种是**str**。In older Pythons, the floating-point **repr** sometimes displays more precision than you might expect.

从 2.7 和最新的 3.X，浮点数显示问题得到优化。

    >>> 3.1415 * 2  # repr: as code (Pythons >= 2.7 and 3.1)
    6.283

数学模块：

    >>> import math
    >>> math.pi
    3.141592653589793
    >>> math.sqrt(85)
    9.219544457292887
    >>> import random
    >>> random.random()
    0.7082048489415967
    >>> random.choice([1, 2, 3, 4])
    1

Python还支持复数、定点数和有理数。

### 字符串

#### 序列操作

字符串是Python中序列（Sequence）的第一个例子。

    >>> S = 'Spam'  # Make a 4-character string, and assign it to a name
    >>> len(S)  # Length
    4
    >>> S[0]  # The first item in S, indexing by zero-based position
    'S'
    >>> S[1]  # The second item from the left
    'p'

索引可以从后向前

    >>> S[-1]  # The last item from the end in S
    'm'
    >>> S[-2]  # The second-to-last item from the end
    'a'

分片：

    >>> S  # A 4-character string
    'Spam'
    >>> S[1:3]  # Slice of S from offsets 1 through 2 (not 3)
    'pa'

分别的左边界默认为0，右边界默认为序列的长度。因此下面的等价的：

    >>> S[1:]  # Everything past the first (1:len(S))
    'pam'
    >>> S  # S itself hasn't changed
    'Spam'
    >>> S[0:3]  # Everything but the last
    'Spa'
    >>> S[:3]  # Same as S[0:3]
    'Spa'
    >>> S[:-1]  # Everything but the last again, but simpler (0:-1)
    'Spa'
    >>> S[:]  # All of S as a top-level copy (0:len(S))
    'Spam'

字符串的连接与重复：

    >>> S
    'Spam'
    >>> S + 'xyz'  # Concatenation
    'Spamxyz'
    >>> S  # S is unchanged
    'Spam'
    >>> S * 8  # Repetition
    'SpamSpamSpamSpamSpamSpamSpamSpam'

#### 不可变性

Python中字符串是不可变的。

改变字符串中的字符有两种方式：

    >>> S = 'shrubbery'
    >>> L = list(S)  # Expand to a list: [...]
    >>> L
    ['s', 'h', 'r', 'u', 'b', 'b', 'e', 'r', 'y']
    >>> L[1] = 'c'  # Change it in place
    >>> ''.join(L)  # Join with empty delimiter
    'scrubbery'
    >>> B = bytearray(b'spam')  # A bytes/list hybrid (ahead)
    >>> B.extend(b'eggs')  # 'b' needed in 3.X, not 2.X
    >>> B  # B[i] = ord(c) works here too
    bytearray(b'spameggs')
    >>> B.decode()  # Translate to normal string
    'spameggs'

`bytearray`需要2.6或3.0之后版本。且要求字符串是8位编码（比如ASCII）。

#### 方法

    >>> S = 'Spam'
    >>> S.find('pa')  # Find the offset of a substring in S
    1
    >>> S
    'Spam'
    >>> S.replace('pa', 'XYZ')  # Replace occurrences of a string in S with another
    'SXYZm'
    >>> S
    'Spam'

格式化。可以使表达式形式（最初形式）、方法调用（2.6或3.0）；2.7和3.1开始允许省略位置数字：

    >>> '%s, eggs, and %s' % ('spam', 'SPAM!')  # Formatting expression (all)
    'spam, eggs, and SPAM!'
    >>> '{0}, eggs, and {1}'.format('spam', 'SPAM!')  # Formatting method (2.6+, 3.0+)
    'spam, eggs, and SPAM!'
    >>> '{}, eggs, and {}'.format('spam', 'SPAM!')  # Numbers optional (2.7+, 3.1+)
    'spam, eggs, and SPAM!'

    >>> '{:,.2f}'.format(296999.2567)  # Separators, decimal digits
    '296,999.26'
    >>> '%.2f | %+05d' % (3.14159, −42)  # Digits, padding, signs
    '3.14 | −0042'

#### 寻求帮助

For more details, you can always call the built-in `dir` function. This function lists variables assigned in the caller’s scope when called with no argument; more usefully, it returns a list of all the attributes available for any object passed to it. 假设`S`是字符串，下面是Python 3.3的显示（Python 2.X略有不同）：

    >>> dir(S)
    ['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__',
    '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__',
    '__getnewargs__', '__gt__', '__hash__', '__init__', '__iter__', '__le__',
    '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__',
    '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__',
    '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count',
    'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index',
    'isalnum', 'isalpha', 'isdecimal', 'isdigit', 'isidentifier', 'islower',
    'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust',
    'lower', 'lstrip', 'maketrans', 'partition', 'replace', 'rfind', 'rindex',
    'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith',
    'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']

若想查看具体方法的描述：

    >>> help(S.replace)
    Help on built-in function replace:
    replace(...)
        S.replace(old, new[, count]) -> str

        Return a copy of S with all occurrences of substring
        old replaced by new. If the optional argument count is
        given, only the first count occurrences are replaced.

You can also ask for help on an *entire string*(e.g., `help(S)`), but you may get more or less help than you want to see — information about every string method in older Pythons, and probably no help at all in newer versions because strings are treated specially. It’s generally better to ask about a specific method.

`dir`和`help`的参数可以是对象（如字符串`S`）也可以是数据类型的名字（如`str`、`list`和`dict`）。The latter form returns the same list for `dir` but shows full type details for `help`, and allows you to ask about a specific method via type name (e.g., help on `str.replace`).

#### 其他形式的字符串

    >>> S = 'A\nB\tC'  # \n is end-of-line, \t is tab
    >>> len(S)  # Each stands for just one character
    5
    >>> ord('\n')  # \n is a byte with the binary value 10 in ASCII
    10
    >>> S = 'A\0B\0C'  # \0, a binary zero byte, does not terminate string
    >>> len(S)
    5
    >>> S  # Non-printables are displayed as \xNN hex escapes
    'a\x00B\x00C'

字符串可以被单引号或双引号包围。三个引号（单引号或双信号）允许跨行字符串：

    >>> msg = """
    aaaaaaaaaaaaa
    bbb'''bbbbbbbbbb""bbbbbbb'bbbb
    cccccccccccccc
    """
    >>> msg
    '\naaaaaaaaaaaaa\nbbb\'\'\'bbbbbbbbbb""bbbbbbb\'bbbb\ncccccccccccccc\n'

Python还支持原生字符串，此时不会处理转义。例如`r'C:\text\new'`。

#### Unicode字符串

Python 3.X，普通的str类型字符串能处理Unicode字符。另一种类型，字节字符串表示原始字节值；为了兼容2.X，2.X支持2.X的 Unicode字符串（3.X中不必使用Unity字符串，可以直接使用普通字符串）：

    >>> 'sp\xc4m'  # 3.X: normal str strings are Unicode text
    'spÄm'
    >>> b'a\x01c'  # bytes strings are byte-based data
    b'a\x01c'
    >>> u'sp\u00c4m'  # The 2.X Unicode literal works in 3.3+: just str
    'spÄm'

在Python 2.X中普通str类型字符串只能处理8比特字符或原始字节值；一个额外的、Unicode字符串类型表示Unicode文本；为兼容3.X，2.6及之后版本支持3.X的字节字面量（在2.X中，使用普通str就行）：

    >>> print u'sp\xc4m'  # 2.X: Unicode strings are a distinct type
    spÄm
    >>> 'a\x01c'  # Normal str strings contain byte-based text/data
    'a\x01c'
    >>> b'a\x01c'  # The 3.X bytes literal works in 2.6+: just str
    'a\x01c'


    >>> 'spam'  # Characters may be 1, 2, or 4 bytes in memory
    'spam'
    >>> 'spam'.encode('utf8')  # Encoded to 4 bytes in UTF-8 in files
    b'spam'
    >>> 'spam'.encode('utf16')  # But encoded to 10 bytes in UTF-16
    b'\xff\xfes\x00p\x00a\x00m\x00'

Both 3.X and 2.X also support coding non-ASCIIcharacters with \xhexadecimal and short \u and long \U Unicode escapes, as well as file-wide encodings declared in program source files. Here’s our non-ASCII character coded three ways in 3.X (add a leading “u” and say “print” to see the same in 2.X):

    >>> 'sp\xc4\u00c4\U000000c4m'
    'spÄÄÄm'

What these values mean and how they are used differs between text strings, which are the normal string in 3.X and Unicode in 2.X, and byte strings, which are bytes in 3.X and the normal string in 2.X. All these escapes can be used to embed actual Unicode code-point ordinal-value integers in text strings. By contrast, byte strings use only \x hexadecimal escapes to embed the encoded form of text, not its decoded code point values—encoded bytes are the same as code points, only for some encodings and characters:

    >>> '\u00A3', '\u00A3'.encode('latin1'), b'\xA3'.decode('latin1')
    ('£', b'\xa3', '£')

As a notable difference, Python 2.X allows its normal and Unicode strings to be mixed in expressions as long as the normal string is all ASCII; in contrast, Python 3.X has a tighter model that never allows its normal and byte strings to mix without explicit conversion:

    u'x' + b'y'  # Works in 2.X (where b is optional and ignored)
    u'x' + 'y'  # Works in 2.X: u'xy'
    u'x' + b'y'  # Fails in 3.3 (where u is optional and ignored)
    u'x' + 'y'  # Works in 3.3: 'xy'
    'x' + b'y'.decode()  # Works in 3.X if decode bytes to str: 'xy'
    'x'.encode() + b'y'  # Works in 3.X if encode str to bytes: b'xy'

Apart from these string types, Unicode processing mostly reduces to transferring text data to and from files—text is encodedto bytes when stored in a file, and decoded into characters (a.k.a. code points) when read back into memory. Once it is loaded, we usually process text as strings in decoded form only.

Because of this model, though, files are also content-specific in 3.X: text files implement named encodings and accept and return str strings, but binary files instead deal in bytes strings for raw binary data. In Python 2.X, normal files’ content is str bytes, and a special codecs module handles Unicode and represents content with the `unicode `type.

We’ll meet Unicode again in the files coverage later in this chapter, but save the rest of the Unicode story for later in this book. It crops up briefly in a Chapter 25 example in conjunction with currency symbols, but for the most part is postponed until this book’s advanced topics part. Unicode is crucial in some domains, but many programmers can get by with just a passing acquaintance. If your data is all ASCII text, the string and file stories are largely the same in 2.X and 3.X. And if you’re new to programming, you can safely defer most Unicode details until you’ve mastered string basics.

#### 模式匹配

    >>> import re
    >>> match = re.match('Hello[ \t]*(.*)world', 'Hello Python world')
    >>> match.group(1)
    'Python '

### List

序列的一种。有序集合。大小不固定。可变。

#### 序列操作

因为也是序列，因此字符串的序列操作也适应与List：

    >>> L = [123, 'spam', 1.23]  # A list of three different-type objects
    >>> len(L)  # Number of items in the list
    3
    >>> L[0]  # Indexing by position
    123
    >>> L[:-1]  # Slicing a list returns a new list
    [123, 'spam']
    >>> L + [4, 5, 6]  # Concat/repeat make new lists too
    [123, 'spam', 1.23, 4, 5, 6]
    >>> L * 2
    [123, 'spam', 1.23, 123, 'spam', 1.23]
    >>> L  # We're not changing the original list
    [123, 'spam', 1.23]

#### 方法

    >>> L.append('NI')  # Growing: add object at end of list
    >>> L
    [123, 'spam', 1.23, 'NI']
    >>> L.pop(2)  # Shrinking: delete an item in the middle
    1.23
    >>> L  # "del L[2]" deletes from a list too
    [123, 'spam', 'NI']

    >>> M = ['bb', 'aa', 'cc']
    >>> M.sort()
    >>> M
    ['aa', 'bb', 'cc']
    >>> M.reverse()
    >>> M
    ['cc', 'bb', 'aa']

#### 边界检查

    >>> L
    [123, 'spam', 'NI']
    >>> L[99]
    ...error text omitted...
    IndexError: list index out of range
    >>> L[99] = 1
    ...error text omitted...
    IndexError: list assignment index out of range

#### 嵌套

    >>> M = [[1, 2, 3],  # A 3 × 3 matrix, as nested lists
    [4, 5, 6],  # Code can span lines if bracketed
    [7, 8, 9]]
    >>> M
    [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    >>> M[1]  # Get row 2
    [4, 5, 6]
    >>> M[1][2]  # Get row 2, then get item 3 within the row
    6

#### 推导（Comprehension）

高级操作：list comprehension expression。例如，抽出矩阵的第二列：

    >>> col2 = [row[1] for row in M]  # Collect the items in column 2
    >>> col2
    [2, 5, 8]
    >>> M  # The matrix is unchanged
    [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

其他例子：

    >>> [row[1] + 1 for row in M]  # Add 1 to each item in column 2
    [3, 6, 9]
    >>> [row[1] for row in M if row[1] % 2 == 0]  # Filter out odd items
    [2, 8]

    >>> doubles = [c * 2 for c in 'spam']  # Repeat characters in a string
    >>> doubles
    ['ss', 'pp', 'aa', 'mm']


These expressions can also be used to collect multiple values, as long as we wrap those values in a nested collection. The following illustrates using `range` — a built-in that generates successive integers, and requires a surrounding listto display all its values in 3.X only (2.X makes a physical list all at once):

    >>> list(range(4))  # 0..3 (list() required in 3.X)
    [0, 1, 2, 3]
    >>> list(range(−6, 7, 2))  # −6 to +6 by 2 (need list() in 3.X)
    [−6, −4, −2, 0, 2, 4, 6]
    >>> [[x ** 2, x ** 3] for x in range(4)]  # Multiple values, "if" filters
    [[0, 0], [1, 1], [4, 8], [9, 27]]
    >>> [[x, x / 2, x * 2] for x in range(−6, 7, 2) if x > 0]
    [[2, 1, 4], [4, 2, 8], [6, 3, 12]]

### Dictionary

字典不是序列。可变。

#### 映射操作

	>>> D = {'food': 'Spam', 'quantity': 4, 'color': 'pink'}
    >>> D['food']  # Fetch value of key 'food'
    'Spam'
    >>> D['quantity'] += 1  # Add 1 to 'quantity' value
    >>> D
    {'color': 'pink', 'food': 'Spam', 'quantity': 5}

As we’ll learn later, we can also make dictionaries by passing to the `dict` type name either **keyword arguments**, or the result of **zipping** together sequences of keys and values obtained at runtime (e.g., from files). Both the following make the same dictionary as the prior example and its equivalent `{}` literal form, though the first tends to make for less typing:

    >>> bob1 = dict(name='Bob', job='dev', age=40)  # Keywords
    >>> bob1
    {'age': 40, 'name': 'Bob', 'job': 'dev'}
    >>> bob2 = dict(zip(['name', 'job', 'age'], ['Bob', 'dev', 40]))  # Zipping
    >>> bob2
    {'job': 'dev', 'name': 'Bob', 'age': 40}

#### 嵌套

    >>> rec = {'name': {'first': 'Bob', 'last': 'Smith'},
    	'jobs': ['dev', 'mgr'],
    	'age': 40.5}

	>>> rec['name']  # 'name' is a nested dictionary
    {'last': 'Smith', 'first': 'Bob'}
    >>> rec['name']['last']  # Index the nested dictionary
    'Smith'
    >>> rec['jobs']  # 'jobs' is a nested list
    ['dev', 'mgr']
    >>> rec['jobs'][-1]  # Index the nested list
    'mgr'
    >>> rec['jobs'].append('janitor')  # Expand Bob's job description in place
    >>> rec
    {'age': 40.5, 'jobs': ['dev', 'mgr', 'janitor'], 'name': {'last': 'Smith',
    'first': 'Bob'}}

#### 不存在的键

给一个新键赋值是可以的。但获取不存在的键会报错：

    >>> D = {'a': 1, 'b': 2, 'c': 3}
    >>> D
    {'a': 1, 'c': 3, 'b': 2}
    >>> D['e'] = 99  # Assigning new keys grows dictionaries
    >>> D
    {'a': 1, 'c': 3, 'b': 2, 'e': 99}
    >>> D['f']  # Referencing a nonexistent key is an error
    ...error text omitted...
    KeyError: 'f'

通过`in`测试键是否存在：

    >>> 'f' in D
    False
    >>> if not 'f' in D:  # Python's sole selection statement
    	print('missing')
    missing

其他方法包括，`get`方法；Python 2.X的`has_key`方法；三元if-else：

    >>> value = D.get('x', 0)  # Index but with a default
    >>> value
    0
    >>> value = D['x'] if 'x' in D else 0  # if/else expression form
    >>> value
    0

#### 排序键

先排序键在访问：

    >>> Ks = list(D.keys())  # Unordered keys list
    >>> Ks  # A list in 2.X, "view" in 3.X: use list()
    ['a', 'c', 'b']
    >>> Ks.sort()  # Sorted keys list
    >>> Ks
    ['a', 'b', 'c']
    >>> for key in Ks:  # Iterate though sorted keys
    	print(key, '=>', D[key])  # <== press Enter twice here (3.X print)
    a => 1
    b => 2
    c => 3

新版本提供`sorted`函数完成相同功能：

    >>> D
    {'a': 1, 'c': 3, 'b': 2}
    >>> for key in sorted(D):
        print(key, '=>', D[key])
    a => 1
    b => 2
    c => 3

#### 遍历与优化

某个对象类型是可迭代的，需要它们遵守迭代协议：they respond to the `iter` call with an object that advances in response to `next` calls and raises an exception when finished producing values.

### Tuple

是序列。不可变。固定集合。支持任意类型。任意嵌套。

    >>> T = (1, 2, 3, 4)  # A 4-item tuple
    >>> len(T)  # Length
    4
    >> T + (5, 6)  # Concatenation
    (1, 2, 3, 4, 5, 6)
    >>> T[0]  # Indexing, slicing, and more
    1

Tuples also have type-specific callable methods as of Python 2.6 and 3.0, but not nearly as many as lists:

    >>> T.index(4)  # Tuple methods: 4 appears at offset 3
    3
    >>> T.count(4)  # 4 appears once
    1

### 文件

    >>> f = open('data.txt', 'w')  # Make a new file in output mode ('w' is write)
    >>> f.write('Hello\n')  # Write strings of characters to it
    6
    >>> f.write('world\n')  # Return number of items written in Python 3.X
    6
    >>> f.close()  # Close to flush output buffers to disk


    >>> f = open('data.txt')  # 'r' (read) is the default processing mode
    >>> text = f.read()  # Read entire file into a string
    >>> text
    'Hello\nworld\n'
    >>> print(text)  # print interprets control characters
    Hello
    world
    >>> text.split()  # File content is always a string
    ['Hello', 'world']

文件对象提供迭代器，逐行读：

    >>> for line in open('data.txt'): print(line)

#### 二进制字节文件

The prior section’s examples illustrate file basics that suffice for many roles. Technically, though, they rely on either the platform’s Unicode encoding default in Python 3.X, or the 8-bit byte nature of files in Python 2.X. Text files always encode strings in 3.X, and blindly write string content in 2.X. This is irrelevant for the simple ASCII data used previously, which maps to and from file bytes unchanged. But for richer types of data, file interfaces can vary depending on both content and the Python line you use.

Python 3.X 区别文件中的文本和二进制数据：text files represent content as normal `str` strings，读写数据时自动进行Unicode编解码，while binary files represent content as a special **bytes string** and allow you to access file content unaltered. Python 2.X supports the same dichotomy, but doesn’t impose it as
rigidly, and its tools differ.

For example, binary filesare useful for processing media, accessing data created by C programs, and so on. To illustrate, Python’s `struct` module can both create and unpack packed binary data—raw bytes that record values that are not Python objects—to be written to a file in binary mode. We’ll study this technique in detail later in the book, but the concept is simple: the following creates a binary file in Python 3.X (binary files work the same in 2.X, but the “b” string literal prefix isn’t required and won’t be displayed):

    >>> import struct
    >>> packed = struct.pack('>i4sh', 7, b'spam', 8)  # Create packed binary data
    >>> packed  # 10 bytes, not objects or text
    b'\x00\x00\x00\x07spam\x00\x08'
    >>>
    >>> file = open('data.bin', 'wb')  # Open binary output file
    >>> file.write(packed)  # Write packed binary data
    10
    >>> file.close()

Reading binary data back is essentially symmetric; not all programs need to tread so deeply into the low-level realm of bytes, but binary files make this easy in Python:

    >>> data = open('data.bin', 'rb').read()  # Open/read binary data file
    >>> data  # 10 bytes, unaltered
    b'\x00\x00\x00\x07spam\x00\x08'
    >>> data[4:8]  # Slice bytes in the middle
    b'spam'
    >>> list(data)  # A sequence of 8-bit bytes
    [0, 0, 0, 7, 115, 112, 97, 109, 0, 8]
    >>> struct.unpack('>i4sh', data)  # Unpack into objects again
    (7, b'spam', 8)

#### Unicode文本文件

Luckily, this is easier than it may sound. To access files containing non-ASCII Unicode textof the sort introduced earlier in this chapter, we simply pass in an encoding name if the text in the file doesn’t match the default encoding for our platform. 在该模式下，Python会在写入读取时自动进行编解码。In Python 3.X:

    >>> S = 'sp\xc4m'  # Non-ASCII Unicode text
    >>> S
    'spÄm'
    >>> S[2]  # Sequence of characters
    'Ä'
    >>> file = open('unidata.txt', 'w', encoding='utf-8')  # Write/encode UTF-8 text
    >>> file.write(S)  # 4 characters written
    4
    >>> file.close()
    >>> text = open('unidata.txt', encoding='utf-8').read()  # Read/decode UTF-8 text
    >>> text
    'spÄm'
    >>> len(text)  # 4 chars (code points)
    4

If needed, though, you can also see what’s truly stored in your file by stepping into binary mode:

    >>> raw = open('unidata.txt', 'rb').read()  # Read raw encoded bytes
    >>> raw
    b'sp\xc3\x84m'
    >>> len(raw)  # Really 5 bytes in UTF-8
    5

可以手工编解码：

    >>> text.encode('utf-8')  # Manual encode to bytes
    b'sp\xc3\x84m'
    >>> raw.decode('utf-8')  # Manual decode to str
    'spÄm'

Python 2.X中，Unicode字符串要加前缀`u`，二进字字符串不需要前缀`b`，Unicode文本文件必须通过`codecs.open`打开，才能像3.X的`open`一样指定编码, and uses the special `unicode` string to represent content in memory. Binary file mode may seem optional in 2.X since normal files are just byte-based data, but it’s required to avoid changing line ends if present (more on this later in the book):

    >>> import codecs
    >>> codecs.open('unidata.txt', encoding='utf8').read()  # 2.X: read/decode text
    u'sp\xc4m'
    >>> open('unidata.txt', 'rb').read()  # 2.X: read raw bytes
    'sp\xc3\x84m'
    >>> open('unidata.txt').read()  # 2.X: raw/undecoded too
    'sp\xc3\x84m'

### 其他核心类型

You create sets by calling the built-in `set` function or using new set literals and expressions in 3.X and 2.7, and they support the usual mathematical set operations (the choice of new {...} syntax for set literals makes sense, since sets are much like the keys of a valueless dictionary):

    >>> X = set('spam')  # Make a set out of a sequence in 2.X and 3.X
    >>> Y = {'h', 'a', 'm'}  # Make a set with set literals in 3.X and 2.7
    >>> X, Y  # A tuple of two sets without parentheses
    ({'m', 'a', 'p', 's'}, {'m', 'a', 'h'})
    >>> X & Y  # Intersection
    {'m', 'a'}
    >>> X | Y  # Union
    {'m', 'h', 'a', 'p', 's'}
    >>> X - Y  # Difference
    {'p', 's'}
    >>> X > Y  # Superset
    False
    >>> {n ** 2 for n in [1, 2, 3, 4]}  # Set comprehensions in 3.X and 2.7
    {16, 1, 4, 9}

布尔和`NONE`：

    >>> 1 > 2, 1 < 2  # Booleans
    (False, True)
    >>> bool('spam')  # Object's Boolean value
    True
    >>> X = None  # None placeholder
    >>> print(X)
    None
    >>> L = [None] * 100  # Initialize a list of 100 Nones
    >>> L
    [None, None, None, None, None, None, None, None, None, None, None, None,
    None, None, None, None, None, None, None, None, ...a list of 100 Nones...]

#### type

`type`对象，是函数`type`的返回值；its result differs slightly in 3.X, because types have merged with classes completely. Assuming L is still the list of the prior section:

    # In Python 2.X:
    >>> type(L)  # Types: type of L is list type object
    <type 'list'>
    >>> type(type(L))  # Even types are objects
    <type 'type'>
    # In Python 3.X:
    >>> type(L)  # 3.X: types are classes, and vice versa
    <class 'list'>
    >>> type(type(L))  # See Chapter 32 for more on class types
    <class 'type'>

Besides allowing you to explore your objects interactively, the typeobject in its most practical application allows code to check the types of the objects it processes. In fact, there are at least three ways to do so in a Python script:

    >>> if type(L) == type([]):  # Type testing, if you must...
    print('yes')
    yes
    >>> if type(L) == list:  # Using the type name
    print('yes')
    yes
    >>> if isinstance(L, list):  # Object-oriented tests
    print('yes')
    yes

## 5. 数值类型

### （未）5.1 数值类型基础

### （未）5.2 数值实践

### （未）5.3 其他数字类型

### （未）5.4 Numeric Extensions

## （未）6. The Dynamic Typing Interlude

## 7. 字符串基础

### 7.1 本章内容范围

本章关注的是`str`类型的字符串。关于Unicode的内容见第37章。

- Python 3.X有三种字符串类型：`str`用于Unicode文本，`bytes`用于二进制数据（包括编码过的文本）；`bytearray`是`bytes`的可变版本。文件有两种模式：*text*，将文件内容表示成`str`，实现Unicode编码；*binary*，处理原始的字节。
- Python 2.X中，`unicode`字符串表示Unicode文本，`str`字符串处理8比特文本和二进制数据，`bytearray`从2.6后才出现，为了兼容3.X。普通文件的内容一般是字节，由`str`表示；但`codecs`模块可以打开Unicode编码的文件，将文件内容表示为`unicode`对象。

### 7.2 字符串基础

    S = "spam's"  // Double quotes, same as single
    S = 's\np\ta\x00m' // Escape sequences
    S = """...multiline...""" // Triple-quoted block strings
    S = r'\temp\spam' // Raw strings (no escapes)
    B = b'sp\xc4m' // Byte strings in 2.6, 2.7, and 3.X (Chapter 4, Chapter 37)
    U = u'sp\u00c4m' // Unicode strings in 2.X and 3.3+ (Chapter 4, Chapter 37)
    "a %s parrot" % kind // String formatting expression
    "a {0} parrot".format(kind) // String formatting method in 2.6, 2.7, and 3.X

### 7.4 字符串字面量

- Single quotes: `'spa"m'`
- Double quotes: `"spa'm"`
- Triple quotes: `'''... spam ...''', """... spam ..."""`
- Escape sequences: `"s\tp\na\0m"`
- Raw strings: `r"C:\new\test.spm"`
- Bytes literals in 3.X and 2.6+ (see Chapter 4, Chapter 37): `b'sp\x01am'`
- Unicode literals in 2.X and 3.3+ (see Chapter 4, Chapter 37): `u'eggs\u0020spam'`

单引号和双引号可以互换。

Python会自动连接相邻的字符串

    >>> title = "Meaning " 'of' " Life"  # Implicit concatenation
    >>> title
    'Meaning of Life'

内建函数`len`返回字符串中的字符数——不管字符串被如何编码和显示。

实际上3.X将`str`定义为Unicode code points的序列，而不是字节序列。详见第37章。

转义：

- \xhh Character with hex value hh(exactly 2 digits)
- \ooo Character with octal value ooo(up to 3 digits)
- \0 Null: binary 0 character (doesn’t end string)
- \N{ id } Unicode database ID
- \uhhhh Unicode character with 16-bit hex value
- \Uhhhhhhhh Unicode character with 32-bit hex value
- \other Not an escape (keeps both \and other)

Finally, if Python does not recognize the character after a \ as being a valid escape code, it simply keeps the backslash in the resulting string:

    >>> x = "C:\py\code"  # Keeps \ literally (and displays it as \\)
    >>> x
    'C:\\py\\code'
    >>> len(x)
    10

### 7.4 字符串实战

Extended slicing (`S[i:j:k]`) accepts a step (or stride) k, which defaults to +1. `[::−1]`可以用于反序。如`"hello"[::−1]`的结果是"olleh"。

Later in the book, we’ll also learn that slicing is equivalent to indexing with a `slice` object, a finding of importance to class writers seeking to support both operations:

    >>> 'spam'[1:3]  # Slicing syntax
    'pa'
    >>> 'spam'[slice(1, 3)]  # Slice objects with index syntax + object
    'pa'
    >>> 'spam'[::-1]
    'maps'
    >>> 'spam'[slice(None, None, −1)]
    'maps'

应用：去掉第一个字符：`sys.argv[1:]`；去掉最后一个：`line[:−1]`。

不能将字符串与数字相加：

    # Python 3.X
    >>> "42" + 1
    TypeError: Can't convert 'int' object to str implicitly
    # Python 2.X
    >>> "42" + 1
    TypeError: cannot concatenate 'str' and 'int' objects

需要时需要显式转换：

    >>> int("42"), str(42)  # Convert from/to string
    (42, '42')
    >>> repr(42)  # Convert to as-code string
    '42'
    >>> str(3.1415), float("1.5")
    ('3.1415', 1.5)
    >>> text = "1.234E-10"
    >>> float(text)  # Shows more digits before 2.7 and 3.1
    1.234e-10

### 7.5 字符串方法

#### （未）The Original string Module’s Functions (Gone in 3.X)

