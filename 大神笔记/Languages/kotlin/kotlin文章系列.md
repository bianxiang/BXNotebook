[toc]

# Kotlin 集合

http://antonioleiva.com/collection-operations-kotlin/

Although we can just use Java collections, Kotlin provides a good set of native interfaces you will want to use:

- `Iterable`: Any classes that inherit from this interface represent a sequence of elements we can iterate over.
- `MutableIterable`: 支持迭代过程中删除元素。
- `Collection`: This class represents a generic collection of elements. We get access to functions that return the size of the collection, whether the collection is empty, contains an item or a set of items. 只支持读取数据，因为 collections 是不可变的。
- `MutableCollection`: a Collection that supports adding and removing elements. It provides extra functions such as add, remove or clear among others.
- `List`: Probably the most used collection. It represents a generic ordered collection of elements. As it’s ordered, we can request an item by its position, using the get function.
- `MutableList`: a List that supports adding and removing elements.
- `Set`: an unordered collection of elements that doesn’t support duplicate elements.
- `MutableSet`: a Set that supports adding and removing elements.
- `Map`: a collection of key-value pairs. The keys in a map are unique, which means we cannot have two pairs with the same key in a map.
- `MutableMap`: a Map that supports adding and removing elements.


All this content and much more can be found in Kotlin for Android Developers book.

## 18.1 聚合操作

### any

Returns true if at least one element matches the given predicate.

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertTrue(list.any { it % 2 == 0 })
assertFalse(list.any { it > 10 })
```

### all

Returns true if all the elements match the given predicate.

```kotlin
assertTrue(list.all { it < 10 })
assertFalse(list.all { it % 2 == 0 })
```

### count

Returns the number of elements matching the given predicate.

```kotlin
assertEquals(3, list.count { it % 2 == 0 })
```

### fold

Accumulates the value starting with an initial value and applying an operation from the first to the last element in a collection.

```kotlin
assertEquals(25, list.fold(4) { total, next -> total + next })
```

### foldRight

Same as fold, but it goes from the last element to first.

```kotlin
assertEquals(25, list.foldRight(4) { total, next -> total + next })
```

### forEach

Performs the given operation to each element.

```kotlin
list forEach { println(it) }
```

### forEachIndexed

Same as forEach, though we also get the index of the element.

```kotlin
list forEachIndexed { index, value 
       -> println("position $index contains a $value") }
```

### max

Returns the largest element or null if there are no elements.

```kotlin
assertEquals(6, list.max())
```

### maxBy

Returns the first element yielding the largest value of the given function or null if there are no elements.

```kotlin
// The element whose negative is greater
assertEquals(1, list.maxBy { -it })
```

### min

Returns the smallest element or null if there are no elements.

```kotlin
assertEquals(1, list.min())
```

### minBy

Returns the first element yielding the smallest value of the given function or null if there are no elements.

```kotlin
// The element whose negative is smaller
assertEquals(6, list.minBy { -it })
```

### none

Returns true if no elements match the given predicate.

```kotlin
// No elements are divisible by 7
assertTrue(list.none { it % 7 == 0 })
```

### reduce

Same as fold, but it doesn’t use an initial value. It accumulates the value applying an operation from the first to the last element in a collection.

```kotlin
assertEquals(21, list.reduce { total, next -> total + next })
```

### reduceRight

Same as reduce, but it goes from the last element to first.

```kotlin
assertEquals(21, list.reduceRight { total, next -> total + next })
```

### sumBy

Returns the sum of all values produced by the transform function from the elements in the collection.

```kotlin
assertEquals(3, list.sumBy { it % 2 })
```

## 18.2 过滤操作

### drop

Returns a list containing all elements except first n elements.

```kotlin
assertEquals(listOf(5, 6), list.drop(4))
```

### dropWhile

Returns a list containing all elements except first elements that satisfy the given predicate.

```kotlin
assertEquals(listOf(3, 4, 5, 6), list.dropWhile { it < 3 })
```

### dropLastWhile

Returns a list containing all elements except last elements that satisfy the given predicate.

```kotlin
assertEquals(listOf(1, 2, 3, 4), list.dropLastWhile { it > 4 })
```

### filter

Returns a list containing all elements matching the given predicate.

```kotlin
assertEquals(listOf(2, 4, 6), list.filter { it % 2 == 0 })
```

### filterNot

Returns a list containing all elements not matching the given predicate.

```kotlin
assertEquals(listOf(1, 3, 5), list.filterNot { it % 2 == 0 })
```

### filterNotNull

Returns a list containing all elements that are not null.

```kotlin
assertEquals(listOf(1, 2, 3, 4), listWithNull.filterNotNull())
```

### slice

Returns a list containing elements at specified indices.

```kotlin
assertEquals(listOf(2, 4, 5), list.slice(listOf(1, 3, 4)))
```

### take

Returns a list containing first n elements.

```kotlin
assertEquals(listOf(1, 2), list.take(2))
```

### takeLast

Returns a list containing last n elements.

```kotlin
assertEquals(listOf(5, 6), list.takeLast(2))
```

### takeWhile

Returns a list containing first elements satisfying the given predicate.

```kotlin
assertEquals(listOf(1, 2), list.takeWhile { it < 3 })
```

## 18.3 Mapping operations

### flatMap

Iterates over the elements creating a new collection for each one, and finally flattens all the collections into a unique list containing all the elements.

```kotlin
assertEquals(listOf(1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7), list.flatMap { listOf(it, it + 1) })
```

### groupBy

Returns a map of the elements in original collection grouped by the result of given function

```kotlin
assertEquals(mapOf("odd" to listOf(1, 3, 5), "even" to listOf(2, 4, 6)),
            list.groupBy { if (it % 2 == 0) "even" else "odd" })
```
    
### map

Returns a list containing the results of applying the given transform function to each element of the original collection.

```kotlin
assertEquals(listOf(2, 4, 6, 8, 10, 12), list.map { it * 2 })
```

### mapIndexed

Returns a list containing the results of applying the given transform function to each element and its index of the original collection.

```kotlin
assertEquals(listOf (0, 2, 6, 12, 20, 30), list.mapIndexed { index, it
        -> index * it })
```

### mapNotNull

Returns a list containing the results of applying the given transform function to each non-null element of the original collection.

```kotlin
assertEquals(listOf(2, 4, 6, 8), listWithNull mapNotNull { it * 2 })
```

## 18.4 元素操作

### contains

Returns true if the element is found in the collection.

```kotlin
assertTrue(list.contains(2))
```

### elementAt

Returns an element at the given index or throws an IndexOutOfBoundsException if the index is out of bounds of this collection.

```kotlin
assertEquals(2, list.elementAt(1))
```

### elementAtOrElse

Returns an element at the given index or the result of calling the default function if the index is out of bounds of this collection.

```kotlin
assertEquals(20, list.elementAtOrElse(10, { 2 * it }))
```

### elementAtOrNull

Returns an element at the given index or null if the index is out of bounds of this collection.

```kotlin
assertNull(list.elementAtOrNull(10))
```

### first

Returns the first element matching the given predicate

```kotlin
assertEquals(2, list.first { it % 2 == 0 })
```

### firstOrNull

Returns the first element matching the given predicate, or null if no element was found.

```kotlin
assertNull(list.firstOrNull { it % 7 == 0 })
```

### indexOf

Returns the first index of element, or `-1` if the collection does not contain element.

```kotlin
assertEquals(3, list.indexOf(4))
```

### indexOfFirst

Returns index of the first element matching the given predicate, or -1 if the collection does not contain such element.

```kotlin
assertEquals(1, list.indexOfFirst { it % 2 == 0 })
```

### indexOfLast

Returns index of the last element matching the given predicate, or -1 if the collection does not contain such element.

```kotlin
assertEquals(5, list.indexOfLast { it % 2 == 0 })
```

### last

Returns the last element matching the given predicate.

```kotlin
assertEquals(6, list.last { it % 2 == 0 })
```

### lastIndexOf

Returns last index of element, or -1 if the collection does not contain element.

```kotlin
val listRepeated = listOf(2, 2, 3, 4, 5, 5, 6)
assertEquals(5, listRepeated.lastIndexOf(5))
```

### lastOrNull

Returns the last element matching the given predicate, or null if no such element was found.

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertNull(list.lastOrNull { it % 7 == 0 })
```

### single

Returns the single element matching the given predicate, or throws exception if there is no or more than one matching element.

```kotlin
assertEquals(5, list.single { it % 5 == 0 })
```

### singleOrNull

Returns the single element matching the given predicate, or null if element was not found or more than one element was found.

```kotlin
assertNull(list.singleOrNull { it % 7 == 0 })
```

## 18.5 Generation operations

### merge

Returns a list of values built from elements of both collections with same indexes using the provided transform function. The list has the length of shortest collection.

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
val listRepeated = listOf(2, 2, 3, 4, 5, 5, 6)
assertEquals(listOf(3, 4, 6, 8, 10, 11), list.merge(listRepeated) { it1, it2 -> 
        it1 + it2 })
```

### partition

Splits original collection into pair of collections, where the first collection contains elements for which the predicate returned true, while the second collection contains elements for which the predicate returned false.

```kotlin
assertEquals(Pair(listOf(2, 4, 6), listOf(1, 3, 5)), 
        list.partition { it % 2 == 0 })
```

### plus

Returns a list containing all elements of the original collection and then all elements of the given collection. Because of the name of the function, we can use the ‘+’ operator with it.

```kotlin
assertEquals(listOf(1, 2, 3, 4, 5, 6, 7, 8), list + listOf(7, 8))
```

### zip

Returns a list of pairs built from the elements of both collections with the same indexes. The list has the length of the shortest collection.

```kotlin
assertEquals(listOf(Pair(1, 7), Pair(2, 8)), list.zip(listOf(7, 8)))
```

## 18.6 顺序操作

### reverse

Returns a list with elements in reversed order.

```kotlin
val unsortedList = listOf(3, 2, 7, 5)
assertEquals(listOf(5, 7, 2, 3), unsortedList.reverse())
```

## sort

Returns a sorted list of all elements.

```kotlin
assertEquals(listOf(2, 3, 5, 7), unsortedList.sort())
```

### sortBy

Returns a list of all elements, sorted by the specified comparator.

```kotlin
assertEquals(listOf(3, 7, 2, 5), unsortedList.sortBy { it % 3 })
```

### sortDescending

Returns a sorted list of all elements, in descending order.

```kotlin
assertEquals(listOf(7, 5, 3, 2), unsortedList.sortDescending())
```

### sortDescendingBy

Returns a sorted list of all elements, in descending order by the results of the specified order function.

```kotlin
assertEquals(listOf(2, 5, 7, 3), unsortedList.sortDescendingBy { it % 3 })
```
