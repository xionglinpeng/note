# java.util.Vector

## Member Variables

### elementData

```java
/**
 * The array buffer into which the components of the vector are
 * stored. The capacity of the vector is the length of this array buffer,
 * and is at least large enough to contain all the vector's elements.
 *
 * <p>Any array elements following the last element in the Vector are null.
 *
 * @serial
 */
protected Object[] elementData;
```



### elementCount

```java
/**
 * The number of valid components in this {@code Vector} object.
 * Components {@code elementData[0]} through
 * {@code elementData[elementCount-1]} are the actual items.
 *
 * @serial
 */
protected int elementCount;
```



### capacityIncrement

```java
/**
 * The amount by which the capacity of the vector is automatically
 * incremented when its size becomes greater than its capacity.  If
 * the capacity increment is less than or equal to zero, the capacity
 * of the vector is doubled each time it needs to grow.
 *
 * @serial
 */
protected int capacityIncrement;
```



### MAX_ARRAY_SIZE

```java
/**
 * The maximum size of array to allocate.
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```



## Constructor

`java.util.Vector`有四个构造函数。

### Vector()



```java
/**
 * Constructs an empty vector so that its internal data array
 * has size {@code 10} and its standard capacity increment is
 * zero.
 */
public Vector() {
    this(10);
}
```


### Vector(int initialCapacity)



```java
/**
 * Constructs an empty vector with the specified initial capacity and
 * with its capacity increment equal to zero.
 *
 * @param   initialCapacity   the initial capacity of the vector
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}
```



### Vector(int initialCapacity, int capacityIncrement)



```java
/**
 * Constructs an empty vector with the specified initial capacity and
 * capacity increment.
 *
 * @param   initialCapacity     the initial capacity of the vector
 * @param   capacityIncrement   the amount by which the capacity is
 *                              increased when the vector overflows
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}
```



### Vector(Collection<? extends E> c)



```java
/**
 * Constructs a vector containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this
 *       vector
 * @throws NullPointerException if the specified collection is null
 * @since   1.2
 */
public Vector(Collection<? extends E> c) {
    elementData = c.toArray();
    elementCount = elementData.length;
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
}
```



















































































































