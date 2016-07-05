Version 3.5.0

[toc]

## String

### 转驼峰：camelCase

	_.camelCase([string=''])

Converts string to camel case.

Arguments

- [string=''] (string): The string to convert.

Returns

(string): Returns the camel cased string.

Example

    _.camelCase('Foo Bar');
    // → 'fooBar'
    _.camelCase('--foo-bar');
    // → 'fooBar'
    _.camelCase('__foo_bar__');
    // → 'fooBar'

异常情况：

    _.camelCase(null) // ""
    _.camelCase(2) // "2"

### 字符串首字母大写：capitalize

	_.capitalize([string=''])

Capitalizes the first character of string.

Arguments

- [string=''] (string): The string to capitalize.

Returns
(string): Returns the capitalized string.

Example

    _.capitalize('fred'); // → 'Fred'
    _.capitalize("abc dd") // "Abc dd"

### 转基本拉丁字母：deburr

	_.deburr([string=''])

Deburrs string by converting latin-1 supplementary letters to basic latin letters. 参见https://en.wikipedia.org/wiki/Latin-1_Supplement_(Unicode_block)#Character_table
Arguments

- [string=''] (string): The string to deburr.

Returns
(string): Returns the deburred string.

Example

    _.deburr('déjà vu');
    // → 'deja vu'

### endsWith

	_.endsWith([string=''], [target], [position=string.length])

Checks if string ends with the given target string.

Arguments

- [string=''] (string): The string to search.
- [target] (string): The string to search for.
- [position=string.length] (number): The position to search from.

Returns
(boolean): Returns true if string ends with target, else false.

Example

    _.endsWith('abc', 'c'); // → true
    _.endsWith('abc', 'b'); // → false
    _.endsWith('abc', 'b', 2); // → true
    _.endsWith("abc", "bc") // true






