[toc]

## DOM树游走

### .children()

Returns: jQuery

Get the children of each element in the set of matched elements, optionally filtered by a selector.

```js
.children( [selector ] )
```

- `selector`：选择符。一个字符串，包含选择符表达式，匹配元素。

`.children()`与`.find()`的区别是，`.children()`只会向下寻找一级。但`.find()`会向下寻找多级。与多数jQuery方法一样`.children()`不返回文本节点；to get all children including text and comment nodes, use .contents().

### .find()

Returns: jQuery

Get the descendants of each element in the current set of matched elements, filtered by a selector, jQuery object, or element.

```
.find( selector )
.find( element )
```

- `selector`：选择符。一个字符串，包含选择符表达式，匹配元素。
- `element`：类型：Element 或 jQuery。An element or a jQuery object to match elements against.

Given a jQuery object that represents a set of DOM elements, the `.find()` method allows us to search through the descendants of these elements in the DOM tree and construct a new jQuery object from the matching elements. `.children()`与`.find()`的区别是，`.children()`只会向下寻找一级。但`.find()`会向下寻找多级。

> Unlike most of the tree traversal methods, the selector expression is required in a call to `.find()`. If we need to retrieve all of the descendant elements, we can pass in the universal selector `'*'` to accomplish this.

Selector context is implemented with the .find() method; 因此 `$( "li.item-ii" ).find( "li" )` 与 `$( "li", "li.item-ii" )` 等价。

从jQuery 1.6开始，we can also filter the selection with a given jQuery collection or element. With the same nested list as above, if we start with:

```js
var allListElements = $( "li" );
```

And then pass this jQuery object to find:

```js
$( "li.item-ii" ).find( allListElements );
```

This will return a jQuery collection which contains only the list elements that are descendants of item II. {{看成一种取交集的方法，先通过一种方式选出一组元素，然后在其中选出符合`find()`条件的}}

Similarly, an element may also be passed to find:

```js
var item1 = $( "li.item-1" )[ 0 ];
$( "li.item-ii" ).find( item1 ).css( "background-color", "red" );
```

### .parent()

Returns: jQuery

Get the parent of each element in the current set of matched elements, optionally filtered by a selector.

```
.parent( [selector ] )
```

- selector：A string containing a selector expression to match elements against.

Given a jQuery object that represents a set of DOM elements, the **parent()** method traverses to the **immediate** parent of each of these elements in the DOM tree and constructs a new jQuery object from the matching elements.

This method is similar to `.parents()`, except `.parent()` only travels a single level up the DOM tree. Also, `$( "html" ).parent()` method returns a set containing document whereas `$( "html" ).parents()` returns an empty set.

{{若指定了选择符，可返回的jQuery对象中可能包含一个或零个元素。}}

### .parents()

Returns: jQuery

Get the ancestors of each element in the current set of matched elements, optionally filtered by a selector.

```
.parents( [selector ] )
```

- `selector`：A string containing a selector expression to match elements against.

返回元素的顺序从最近的到最远的。若原来的集合中有多个DOM元素，the resulting set will be in reverse order of the original elements as well, with duplicates removed.

This method is similar to `.parents()`, except `.parent()` only travels a single level up the DOM tree. Also, `$( "html" ).parent()` method returns a set containing document whereas `$( "html" ).parents()` returns an empty set.

### .parentsUntil()

Returns: jQuery

Get the ancestors of each element in the current set of matched elements, up to but not including the element matched by the selector, DOM node, or jQuery object.

```
.parentsUntil( [selector ] [, filter ] )
.parentsUntil( [element ] [, filter ] )
```

- `selector`：A string containing a selector expression to indicate where to stop matching ancestor elements.
- `filter`：A string containing a selector expression to match elements against.
- `element`：Type: Element or jQuery。A DOM node or jQuery object indicating where to stop matching ancestor elements.

Given a selector expression that represents a set of DOM elements, the `.parentsUntil()` method traverses through the ancestors of these elements until it reaches an element matched by the selector passed in the method's argument. The resulting jQuery object contains all of the ancestors up to but not including the one matched by the `.parentsUntil()` selector.

If the selector is not matched or is not supplied, all ancestors will be selected; in these cases it selects the same elements as the `.parents()` method does when no selector is provided.

As of jQuery 1.6, A DOM node or jQuery object, instead of a selector, may be used for the first `.parentsUntil()` argument.

The method optionally accepts a selector expression for its second argument. If this argument is supplied, the elements will be filtered by testing whether they match it.

### .offsetParent()

Returns: jQuery

Get the closest ancestor element that is positioned.

```
.offsetParent()
```

This method does not accept any arguments.

Given a jQuery object that represents a set of DOM elements, the `.offsetParent()` method allows us to search through the ancestors of these elements in the DOM tree and construct a new jQuery object wrapped around the closest positioned ancestor. An element is said to be positioned if it has a CSS position attribute of relative, absolute, or fixed. This information is useful for calculating offsets for performing animations and placing objects on the page.

### .closest()

Returns: jQuery

对于集合中的每个元素，获取匹配条件的第一个元素，**从自己开始** 测试，然后沿着DOM树测试其各个祖先。

```js
.closest( selector )
.closest( selector [, context ] )
.closest( selection )
.closest( element )
```

- `selector`：字符串，选择符表达式
- `context`：类型：Element。A DOM element within which a matching element may be found. If no context is passed in then the context of the jQuery set will be used instead.
- `selection`：类型：jQuery。A jQuery object to match elements against.
- `element`：类型：Element。An element to match elements against.

`.parents()`和`.closest()`都会沿着DOM树向上查找。但区别是：

- `.closest()`从当前元素开始找。而`.parents()`从父元素开始。
- `.closest()`沿着DOM树找，直到找到匹配选择符的为止。而`.parents()`沿着DOM树移植找到文档根元素，将每个祖先元素加入到一个临时的集合，然后根据选择符过滤这个集合。
- `.closest()`返回的jQuery对象，对于源集合中的每个元素，包含零个或一个元素，按照文档顺序。而`.parents()`返回的jQuery对象，对于源集合中的每个元素，包含零个或多个元素，按照反向的文档顺序。

```
    <ul id="one" class="level-1">
      <li class="item-i">I</li>
      <li id="ii" class="item-ii">II
        <ul class="level-2">
          <li class="item-a">A</li>
          <li class="item-b">B
            <ul class="level-3">
              <li class="item-1">1</li>
              <li class="item-2">2</li>
              <li class="item-3">3</li>
            </ul>
          </li>
          <li class="item-c">C</li>
        </ul>
      </li>
      <li class="item-iii">III</li>
    </ul>
```

我们可以传入一个DOM元素，作为上下文，规定在它下面查找匹配元素。

```js
var listItemII = document.getElementById( "ii" );
$( "li.item-a" )
  .closest( "ul", listItemII )
  .css( "background-color", "red" );
$( "li.item-a" )
  .closest( "#one", listItemII )
  .css( "background-color", "green" );
```

`level-2 <ul>`将会变色，因为它是`list item II`的后代。但不会改变`level-1 <ul>`，以为内它不是`list item II`的后代。

例2。Pass a jQuery object to closest. The closest list element toggles a yellow background when it or its descendent is clicked.

```
    <ul>
      <li><b>Click me!</b></li>
      <li>You can also <b>Click me!</b></li>
    </ul>
```
```js
var listElements = $( "li" ).css( "color", "blue" );
$( document ).on( "click", function( event ) {
  $( event.target ).closest( listElements ).toggleClass( "hilight" );
});
```

### .next()

Returns: jQuery

Get the immediately following sibling of each element in the set of matched elements. If a selector is provided, it retrieves the next sibling only if it matches that selector.

```
.next( [selector ] )
```

- `selector`：字符串，选择符表达式

例子：

```
    <ul>
      <li>list item 1</li>
      <li>list item 2</li>
      <li class="third-item">list item 3</li>
      <li>list item 4</li>
      <li>list item 5</li>
    </ul>
```

If we begin at the third item, we can find the element which comes just after it:

```js
$( "li.third-item" ).next().css( "background-color", "red" );
```

将给第四个元素添加样式。

Since we do not supply a selector expression, this following element is unequivocally included as part of the object. If we had supplied one, the element would be tested for a match **before** it was included.

### .nextAll()

Returns: jQuery

Get all following siblings of each element in the set of matched elements, optionally filtered by a selector.

```
.nextAll( [selector ] )
```

- `selector`：A string containing a selector expression to match elements against.

### .nextUntil()

Returns: jQuery

获取集合中每个元素的所有后续兄妹，直到（不包括）匹配选择符、DOM节点或jQuery对象的元素。

```js
.nextUntil( [selector ] [, filter ] )
.nextUntil( [element ] [, filter ] )
```

- `selector`：一个字符串，包含选择器表达式，停止匹配的条件。
- `filter`：一个字符串，包含选择器表达式，元素需满足该匹配。
- `element`：类型：Element 或 jQuery。A DOM node or jQuery object indicating where to stop matching following sibling elements.

Given a selector expression that represents a set of DOM elements, the `.nextUntil()` method searches through the successors of these elements in the DOM tree, stopping when it reaches an element matched by the method's argument. The new jQuery object that is returned contains all following siblings up to but not including the one matched by the `.nextUntil()` argument.

若未提供选择符（或没匹配上），则返回所有的兄妹；此时，结果与`.nextAll()`相同（若没有指定筛选器的话）。

As of jQuery 1.6, A DOM node or jQuery object, instead of a selector, may be passed to the `.nextUntil()` method.

### .prev()

Returns: jQuery

Get the **immediately** preceding sibling of each element in the set of matched elements, optionally filtered by a selector.

```
.prev( [selector ] )
```

- `selector`：A string containing a selector expression to match elements against.

Given a jQuery object that represents a set of DOM elements, the `.prev()` method searches for the predecessor of each of these elements in the DOM tree and constructs a new jQuery object from the matching elements.

If no previous sibling exists, or if the previous sibling element does not match a supplied selector, an empty jQuery object is returned.

### .prevAll()

Returns: jQuery

Get all preceding siblings of each element in the set of matched elements, optionally filtered by a selector.

```
.prevAll( [selector ] )
```

- `selector`：A string containing a selector expression to match elements against.

Given a jQuery object that represents a set of DOM elements, the .prevAll() method searches through the predecessors of these elements in the DOM tree and construct a new jQuery object from the matching elements; the elements are returned in order beginning with the closest sibling.

### .prevUntil()

Returns: jQuery

Get all preceding siblings of each element up to but not including the element matched by the selector, DOM node, or jQuery object.

```
.prevUntil( [selector ] [, filter ] )
.prevUntil( [element ] [, filter ] )
```

- selector：A string containing a selector expression to indicate where to stop matching preceding sibling elements.
- filter：A string containing a selector expression to match elements against.
- element：Type: Element or jQuery。A DOM node or jQuery object indicating where to stop matching preceding sibling elements.

Given a selector expression that represents a set of DOM elements, the `.prevUntil()` method searches through the predecessors of these elements in the DOM tree, stopping when it reaches an element matched by the method's argument. The new jQuery object that is returned contains all previous siblings up to but not including the one matched by the `.prevUntil()` selector; the elements are returned in order from the closest sibling to the farthest.

If the selector is not matched or is not supplied, all previous siblings will be selected; in these cases it selects the same elements as the `.prevAll()` method does when no filter selector is provided.

As of jQuery 1.6, A DOM node or jQuery object, instead of a selector, may be used for the first .prevUntil() argument.

The method optionally accepts a selector expression for its second argument. If this argument is supplied, the elements will be filtered by testing whether they match it.

### .siblings()

Returns: jQuery

Get the siblings of each element in the set of matched elements, optionally filtered by a selector.

```
.siblings( [selector ] )
```

- `selector`：A string containing a selector expression to match elements against.

Given a jQuery object that represents a set of DOM elements, the `.siblings()` method allows us to search through the siblings of these elements in the DOM tree and construct a new jQuery object from the matching elements.

Consider a page with a simple list on it:

```
    <ul>
      <li>list item 1</li>
      <li>list item 2</li>
      <li class="third-item">list item 3</li>
      <li>list item 4</li>
      <li>list item 5</li>
    </ul>
```

If we begin at the third item, we can find its siblings:

```js
$( "li.third-item" ).siblings().css( "background-color", "red" );
```

The result of this call is a red background behind items 1, 2, 4, and 5.

The original element is not included among the siblings, which is important to remember when we wish to find all elements at a particular level of the DOM tree. However, if the original collection contains more than one element, they might be mutual siblings and will both be found. If you need an exclusive list of siblings, use `$collection.siblings().not($collection)`.




