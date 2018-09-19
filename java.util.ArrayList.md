

#  java.util.ArrayList



```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable {}
```



```java
/**
 * 默认初始容量。
 */
private static final int DEFAULT_CAPACITY = 10;

/**
 * 用于空实例的共享空数组实例。
 */
private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * 用于默认大小的空实例的共享空数组实例。我们将其与EMPTY_ELEMENTDATA区分开来，
 * 以了解在添加第一个元素时需要填充多少。
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 存储ArrayList元素的数组缓冲区。
 * ArrayList的容量是这个数组缓冲区的长度。
 * 当添加第一个元素时，任何带有elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * 的空ArrayList都将被扩展到DEFAULT_CAPACITY。
 */
transient Object[] elementData; // 非私有的，以简化嵌套类访问

/**
 * ArrayList的大小（它包含的元素的数量）。
 * @serial
 */
private int size;

/**
 * 要分配的数组的最大大小。
 * 有些VMs在数组中保留一些头字。
 * 试图分配更大的数组可能会导致OutOfMemoryError错误:请求的数组大小超过VM限制
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```



```java
/**
 * 构造具有指定初始容量的空列表。
 *
 * @param  initialCapacity  列表的初始容量
 * @throws IllegalArgumentException 如果指定的初始容量为负
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

/**
 * 构造初始容量为10的空列表。
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * 构造包含指定集合元素的列表，按照集合的迭代器返回它们的顺序。
 *
 * @param c 将其元素放入此列表中的集合
 * @throws NullPointerException 如果指定的集合为空
 */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

## 添加

### add(E e)

add方法首先调用了`private void ensureCapacityInternal(int minCapacity)`用于确定内部容量，如果还有什么容量，就什么也不做，如果元素缓冲区已经满了，则进行扩容操作。然后是元素缓冲区尾部追加元素。

所以可以得出`ArrayList`的一个特点：尽量避免扩容，那么插入元素非常快。

````java
/**
 * 将指定的元素追加到列表的末尾。
 * 
 * @param e 元素添加到此列表中
 * @return <tt>true</tt> (所指定的 {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
````

### set And add

如下的`set`和`add`方法都是在指定索引位置插入元素，不同的是`set`方法是会替换指定索引位置的元素，返回旧元素；而`add`方法是将指定索引及之后的所有元素向右移动，然后在指定索引插入元素，所以它需要确认内部容量，满则扩容。

```java
/**
 * 用指定的元素替换列表中指定位置的元素。
 *
 * @param index 要替换的元素的索引
 * @param element 要存储在指定位置的元素
 * @return 之前位于指定位置的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E set(int index, E element) {
    //进行范围检查，如果超出范围，则抛出数组越界异常
    rangeCheck(index);
	//获取指定索引位的元素
    E oldValue = elementData(index);
    //设置指定索引位新的元素
    elementData[index] = element;
    //返回旧元素
    return oldValue;
}

/**
 * 将指定的元素插入到列表中指定的位置。
 * 将当前位于该位置的元素(如果有的话)和任何后续元素向右移动(向索引中添加一个元素)。
 * @param index 插入指定元素的索引
 * @param element 插入的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    //进行范围检查，如果超出范围，则抛出数组越界异常
    rangeCheckForAdd(index);
	//确认内部容量，满则扩容
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
   	//设置元素
    elementData[index] = element;
    //元素数量+1
    size++;
}
```

### addAll

```java
/**
 * 将指定集合中的所有元素追加到此列表的末尾，按照指定集合的迭代器返回它们的顺序。
 * 如果在操作进行过程中修改了指定的集合，则此操作的行为未定义。
 * (这意味着如果指定的集合是这个列表，并且这个列表不是空的，那么这个调用的行为是未定义的。)
 *
 * @param c 包含要添加到此列表中的元素的集合
 * @return <tt>true</tt> 如果该列表因调用而更改
 * @throws NullPointerException 如果指定的集合为空
 */
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    //将新添加集合的元素拷贝到元素缓冲区尾部
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

/**
 * 从指定位置开始，将指定集合中的所有元素插入到此列表中。将当前位于该
 * 位置的元素(如果有的话)和任何后续元素向右移动(增加它们的索引)。新元
 * 素将按照指定集合的迭代器返回的顺序出现在列表中。
 *
 * @param index 从指定的集合中插入第一个元素的索引
 * @param c 包含要添加到此列表中的元素的集合
 * @return <tt>true</tt> 如果该列表因调用而更改
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException 如果指定的集合为空
 */
public boolean addAll(int index, Collection<? extends E> c) {
    //检查索引是否越界
    rangeCheckForAdd(index);
	//插入的集合转换为一个数组
    Object[] a = c.toArray();
    //插入集合元素数量
    int numNew = a.length;
    //确认内部容量
    ensureCapacityInternal(size + numNew);  // Increments modCount
	//计算要移动的元素数量
    int numMoved = size - index;
    //判断是否有元素需要移动
    if (numMoved > 0)
        //将index索引之后numMoved个元素移动numNew个位置
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);
	//将新的添加的集合元素拷贝到中间空出的位置
    System.arraycopy(a, 0, elementData, index, numNew);
    //变更size
    size += numNew;
    //到了此处，已经添加成功了，判断是否有添加的元素，有就返回true
    return numNew != 0;
}
```

### 扩容计算

1. 调用` calculateCapacity(...)`计算容量，如果当前元素的数组缓冲区等于`DEFAULTCAPACITY_EMPTY_ELEMENTDATA`，则去当前最小容量与默认容量（`DEFAULT_CAPACITY`）去最大值作为容量，否则，去当前最小容量。
2. 调用`ensureExplicitCapacity(...)`确定明确的容量，如果计算的容量已经大于元素数组缓冲区的最大大小时，调用`grow(...)`扩容。
3. 首先扩容50%为新的容量（向右位移1位（`>>1`）,相当于$\div2$）。
4. 如果扩容后的容量比最小容量还小，则以最小容量`minCapacity`为新的容量。
5. 如果新容量大于限制的最大容量`MAX_ARRAY_SIZE`，则使用最小容量`minCapacity`与限制最大容量比较，返回Integer最大值或限制最大容量作为新容量。
6. 使用`Arrays.copyOf(T[] original, int newLength)`将旧元素数组缓冲区拷贝为一个新的数组，并保存至元素数组缓冲区。

```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

//确定内部容量
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
//确定明确的容量
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 * 增加容量，以确保它至少可以容纳最小容量参数指定的元素数量。
 * @param minCapacity 预期的最小容量
 */
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //扩容%50
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //如果最小容量小于扩容容量，则使用最小容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    //拷贝为一个新的数组
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```



## 获取

### get(int index)

`get`方法很简单，就是获取指定索引位置的元素，然后返回即可。

所以可以得出`ArrayList`的一个特点：获取元素非常快。

```java
/**
 * 返回列表中指定位置的元素。
 * @param  index 返回元素的索引
 * @return 位于列表中指定位置的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}
```

## 删除

#### remove(int index)

```java
/**
 * 删除列表中指定位置的元素。
 * 将任何后续元素移到左边(从索引中减去一个)。
 *
 * @param index 要删除的元素的索引
 * @return 从列表中删除的元素
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    //索引数组越界检查
    rangeCheck(index);

    modCount++;
    //获取指定索引的元素
    E oldValue = elementData(index);
	//计算
    int numMoved = size - index - 1;
    //判断是否有元素需要移动
    if (numMoved > 0)
        //index之后的所有元素向前移动1位
        System.arraycopy(elementData, index+1, elementData, index,numMoved);
    //清除最后一位元素，帮助GC
    elementData[--size] = null; // clear to let GC do its work
	//返回被删除的旧值
    return oldValue;
}
```

#### remove(Object o)

```java
/**
 * 移除此列表中首次出现的指定元素（如果存在）。
 * 如果列表不包含此元素，则列表不做改动。
 * 更正式地说，移除满足 (o==null ? get(i)==null : o.equals(get(i))) 
 * 的最低索引的元素（如果存在此类元素）。
 * 如果列表中包含指定的元素，则返回 true（或者等同于这种情况：如果列表由于调
 * 用而发生更改，则返回 true）。
 *
 * @param o 如果存在，则从该列表中删除元素
 * @return <tt>true</tt> 如果这个列表包含指定的元素
 */
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

/*
 * 私有删除方法，跳过边界检查，不返回已删除的值。
 */
private void fastRemove(int index) {
    modCount++;
    //计算需要移动元素的数量
    int numMoved = size - index - 1;
    //判断是否有元素需要移动
    if (numMoved > 0)
        //移动
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //清除最后一位元素，帮助GC
    elementData[--size] = null; // clear to let GC do its work
}
```

#### removeAll(Collection<?> c)

```java
/**
 * 从这个列表中删除指定集合中包含的所有元素。
 *
 * @param c 包含要从列表中删除的元素的集合
 * @return {@code true} 如果该列表因调用而更改
 * @throws ClassCastException 如果此列表中的元素的类与指定的集合不兼容
 * (<a href="Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException 如果该列表包含空元素，并且指定的集合不允许空元素
 * (<a href="Collection.html#optional-restrictions">optional</a>),或者如果指定的集合为空
 * @see Collection#contains(Object)
 */
public boolean removeAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}
```

####  retainAll(Collection<?> c)

```java
/**
 * 仅保留该列表中包含在指定集合中的元素。换句话说，从这个列表中删除指定集合中不包含的所有元素。
 * 即在数组缓冲区中包含在参数c集合中的元素都保留，即求的是交集，即它们都有的元素。
 * 
 * @param c 包含要保留在此列表中的元素的集合
 * @return {@code true} 如果该列表因调用而更改
 * @throws ClassCastException 如果此列表中的元素的类与指定的集合不兼容
 * (<a href="Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException 如果该列表包含空元素，并且指定的集合不允许空元素,
 * 						或者如果指定的集合为空
 * @see Collection#contains(Object)
 */
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}
```

#### batchRemove(Collection<?> c, boolean complement)

```java
private boolean batchRemove(Collection<?> c, boolean complement) {
    //数组缓冲区
    final Object[] elementData = this.elementData;
    int r = 0, w = 0;
    boolean modified = false;
    try {
        //依次迭代数组缓冲区中每一个元素，由参数complement控制是保留包含在集合c中的元素，
        //还是保留不包含在集合c中的元素，true表示保留包含，false表示保留不包含。
        //从数组缓冲区中找到需要保留的元素依次顺序从头放到数组缓存区中
        for (; r < size; r++)
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        // 保留与AbstractCollection的行为兼容性，
        // r在正常情况下是不可能不等于size的，除非c.contains()抛出了依次。
        if (r != size) {
            //将还没有被是否被包含的元素拷贝到数组缓冲区w索引之后
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            //然后增加w索引数
            w += size - r;
        }
        //w!=size，即表示有元素没有被保留，所以讲w索引之后的空格都置空，help gc.
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```

#### removeRange(int fromIndex, int toIndex)

注意：`removeRange`方法是使用`protected`修饰的方法，只能被子类调用

```java
/**
 * 从这个列表中删除所有索引在之间的元素 {@code fromIndex}，包含，和{@code toIndex}，排他
 * Removes from this list all of the elements whose index is between
 * {@code fromIndex}, inclusive, and {@code toIndex}, exclusive.
 * 将任何后续元素移到左边(减少其索引)。
 * 这个调用通过{@code (toIndex - fromIndex)}元素缩短了列表。
 * (如果{@code toIndex==fromIndex}，此操作无效。)
 *
 * @throws IndexOutOfBoundsException if {@code fromIndex} or
 *         {@code toIndex} 超出范围
 *         ({@code fromIndex < 0 ||
 *          fromIndex >= size() ||
 *          toIndex > size() ||
 *          toIndex < fromIndex})
 */
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex,
                     numMoved);

    // clear to let GC do its work
    int newSize = size - (toIndex-fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;
}
```



## range check

```java
/**
 * Checks if the given index is in range.  If not, throws an appropriate
 * runtime exception.  This method does *not* check if the index is
 * negative: It is always used immediately prior to an array access,
 * which throws an ArrayIndexOutOfBoundsException if index is negative.
 */
private void rangeCheck(int index) {
    if (index >= size)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
 * add和addAll使用的rangeCheck版本。
 */
private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

/**
 * Constructs an IndexOutOfBoundsException detail message.
 * Of the many possible refactorings of the error handling code,
 * this "outlining" performs best with both server and client VMs.
 */
private String outOfBoundsMsg(int index) {
    return "Index: "+index+", Size: "+size;
}
```

## enureCapacity

```java
/**
 * 增加这个<tt>ArrayList</tt>实例的容量，
 * 如果有必要，确保它至少可以容纳最小容量参数指定的元素数量。
 *
 * @param minCapacity 预期最小容量
 */
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // 任何大小，如果不是默认元素表
        ? 0
        // 大于默认空表的默认值。它应该是默认大小。
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```

## elementData

```java
@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```

## io

### writeObject(java.io.ObjectOutputStream s)

```java
/**
 * 将<tt>ArrayList</tt>实例的状态保存到流(即序列化)。
 *支持<tt>ArrayList</tt>实例的数组的长度被发出(int)，后面依次是它的所有元素(每个元素都是<tt>对象</tt>)。
 * @serialData The length of the array backing the <tt>ArrayList</tt>
 *             instance is emitted (int), followed by all of its elements
 *             (each an <tt>Object</tt>) in the proper order.
 */
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();

    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

### readObject(java.io.ObjectInputStream s)

```java


/**
 * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
 * deserialize it).
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

## other

###  trimToSize()

```java
/**
 * 将<tt>ArrayList</tt>实例的容量缩减为列表的当前大小。
 * 应用程序可以使用这个操作来最小化<tt>ArrayList</tt>实例的存储。
 */
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```

### size()

```java
/**
 * 返回列表中的元素数量。
 *
 * @return 这个列表中的元素数量
 */
public int size() {
    return size;
}
```

### isEmpty()

```java
/**
 * 如果列表中没有元素，返回<tt>true</tt>。
 * 
 * @return <tt>true</tt>如果这个列表不包含任何元素
 */
public boolean isEmpty() {
    return size == 0;
}
```

### contains(Object o)

```java
/**
 * 如果该列表包含指定的元素，则返回<tt>true</tt>。
 * 更正式地说，当且仅当这个列表包含至少一个元素<tt>e</tt>时，返回<tt>true</tt>
 * <tt>(o==null&nbsp;?&nbsp;e==null&nbsp;:&nbsp;o.equals(e))</tt>.
 *
 * @param o 要测试其在此列表中的存在的元素
 * @return 如果该列表包含指定的元素，则<tt>true</tt>
 */
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}
```

### indexOf(Object o)

```java
/**
 * 返回该列表中指定元素的第一次出现的索引，如果该列表不包含该元素，则返回-1。
 * More formally, returns the lowest index <tt>i</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * 更正式地说，返回最低的索引<tt>i</tt>，使<tt>(o==null&nbsp;?
 * &nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>，如果没有这样的索引，则返回-1。
 */
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

### lastIndexOf(Object o)

```java
/**
 * 返回该列表中指定元素的最后一次出现的索引，如果该列表不包含该元素，则返回-1。
 * More formally, returns the highest index <tt>i</tt> such that
 * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>,
 * or -1 if there is no such index.
 */
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

### clone()

```java
/**
 * 返回这个<tt>ArrayList</tt>实例的浅拷贝。  
 * (元素本身不会被复制。)
 *
 * @return 这个<tt>ArrayList</tt>实例的克隆
 */
public Object clone() {
    try {
        ArrayList<?> v = (ArrayList<?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch (CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```

###  toArray()

```java
/**
 * 返回一个数组，该数组以适当的顺序(从第一个元素到最后一个元素)包含列表中的所有元素。
 *
 * <p>返回的数组将是“安全的(safe)”，因为这个列表不维护对它的引用。
 * (换句话说，这个方法必须分配一个新的数组)。 
 * 因此，调用者可以自由修改返回的数组。
 * <p>此方法充当基于数组和基于集合的api之间的桥梁。
 *
 * @return 一个数组，以适当的顺序包含这个列表中的所有元素
 */
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}
```

### toArray(T[] a)

```java
/**
 * Returns an array containing all of the elements in this list in proper
 * sequence (from first to last element); the runtime type of the returned
 * array is that of the specified array.  If the list fits in the
 * specified array, it is returned therein.  Otherwise, a new array is
 * allocated with the runtime type of the specified array and the size of
 * this list.
 *
 * <p>If the list fits in the specified array with room to spare
 * (i.e., the array has more elements than the list), the element in
 * the array immediately following the end of the collection is set to
 * <tt>null</tt>.  (This is useful in determining the length of the
 * list <i>only</i> if the caller knows that the list does not contain
 * any null elements.)
 *
 * @param a the array into which the elements of the list are to
 *          be stored, if it is big enough; otherwise, a new array of the
 *          same runtime type is allocated for this purpose.
 * @return an array containing the elements of the list
 * @throws ArrayStoreException if the runtime type of the specified array
 *         is not a supertype of the runtime type of every element in
 *         this list
 * @throws NullPointerException if the specified array is null
 */
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

### clear()

```java
/**
 * 从这个列表中删除所有元素。
 * 此调用返回后，列表将为空。
 */
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```



## 1.8新增API

### forEach

forEach是jdk1.8在`public interface Iterable<T>`新增的默认方法，如下：

```java
/**
 * Performs the given action for each element of the {@code Iterable}
 * until all elements have been processed or the action throws an
 * exception.  Unless otherwise specified by the implementing class,
 * actions are performed in the order of iteration (if an iteration order
 * is specified).  Exceptions thrown by the action are relayed to the
 * caller.
 *
 * @implSpec
 * <p>The default implementation behaves as if:
 * <pre>{@code
 *     for (T t : this)
 *         action.accept(t);
 * }</pre>
 *
 * @param action The action to be performed for each element
 * @throws NullPointerException if the specified action is null
 * @since 1.8
 */
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

forEach在ArrayList中的实现

```java
@Override
public void forEach(Consumer<? super E> action) {
    Objects.requireNonNull(action);
    final int expectedModCount = modCount;
    @SuppressWarnings("unchecked")
    //泛型类型推断
    final E[] elementData = (E[]) this.elementData;
    //获取当前集合元素大小
    final int size = this.size;
    //for循环使action消费每一个元素
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        action.accept(elementData[i]);
    }
    //迭代是当前集合不允许发生变化，如果发生变化，则跑出并发修改异常
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```



### removeIf

removeIf是`java.util.Collection`接口1.8新增的默认方法，用于删除断定为真的元素，如下：

```java
/**
 * Removes all of the elements of this collection that satisfy the given
 * predicate.  Errors or runtime exceptions thrown during iteration or by
 * the predicate are relayed to the caller.
 *
 * @implSpec
 * The default implementation traverses all elements of the collection using
 * its {@link #iterator}.  Each matching element is removed using
 * {@link Iterator#remove()}.  If the collection's iterator does not
 * support removal then an {@code UnsupportedOperationException} will be
 * thrown on the first matching element.
 *
 * @param filter a predicate which returns {@code true} for elements to be
 *        removed
 * @return {@code true} if any elements were removed
 * @throws NullPointerException if the specified filter is null
 * @throws UnsupportedOperationException if elements cannot be removed
 *         from this collection.  Implementations may throw this exception if a
 *         matching element cannot be removed or if, in general, removal is not
 *         supported.
 * @since 1.8
 */
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean removed = false;
    final Iterator<E> each = iterator();
    while (each.hasNext()) {
        if (filter.test(each.next())) {
            each.remove();
            removed = true;
        }
    }
    return removed;
}
```

removeIf在ArrayList中的实现

```java
@Override
public boolean removeIf(Predicate<? super E> filter) {
    //验证filter必须不为null
    Objects.requireNonNull(filter);
    // figure out which elements are to be removed
    // any exception thrown from the filter predicate at this stage
    // will leave the collection unmodified
   	//用于统计被删除元素的数量
    int removeCount = 0;
    //创建一个BitSet对象，大小为size
    final BitSet removeSet = new BitSet(si ze);
    final int expectedModCount = modCount;
    //集合元素数量
    final int size = this.size;
    //for循环迭代数组缓冲区中所有元素
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        @SuppressWarnings("unchecked")
        //泛型类型推断转换
        final E element = (E) elementData[i];
        //filter断定，如果断定为真，则将要被删除元素的所有添加到BitSet中
        if (filter.test(element)) {
            //标记指定的索引位位false，对应数组缓冲区要被删除的元素
            removeSet.set(i);
            //被删除的数量加1
            removeCount++;
        }
    }
    //迭代是当前集合不允许发生变化，如果发生变化，则跑出并发修改异常
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }

    // 将剩余的幸存元素移到删除元素所留下的空格上
    final boolean anyToRemove = removeCount > 0;
    if (anyToRemove) {
        //计算被元素被删除后剩下的数量
        final int newSize = size - removeCount;
        //通过BitSet找出所有不被删除的元素，并且从缓冲区中取，然后再依次在缓冲区中顺序保存
        for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
            //返回从索引位i开始第一个为false的索引位，包括i（false是不被删除的）
            i = removeSet.nextClearBit(i);
            //将所有为false索引位的元素顺序保存
            elementData[j] = elementData[i];
        }
        //不被删除的都已经从头开始顺序保存了，后面的所有位都置空，帮助GC
        for (int k=newSize; k < size; k++) {
            elementData[k] = null;  // Let gc do its work
        }
        this.size = newSize;
        //验证迭代时集合没有被改变
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
	//如果有元素被删除，则返回true
    return anyToRemove;
}
```



### replaceAll

`replaceAll`是`java.util.List`接口1.8新增的默认方法，用于一元消费，替换每一个元素，如下：

```java
/**
 * Replaces each element of this list with the result of applying the
 * operator to that element.  Errors or runtime exceptions thrown by
 * the operator are relayed to the caller.
 *
 * @implSpec
 * The default implementation is equivalent to, for this {@code list}:
 * <pre>{@code
 *     final ListIterator<E> li = list.listIterator();
 *     while (li.hasNext()) {
 *         li.set(operator.apply(li.next()));
 *     }
 * }</pre>
 *
 * If the list's list-iterator does not support the {@code set} operation
 * then an {@code UnsupportedOperationException} will be thrown when
 * replacing the first element.
 *
 * @param operator the operator to apply to each element
 * @throws UnsupportedOperationException if this list is unmodifiable.
 *         Implementations may throw this exception if an element
 *         cannot be replaced or if, in general, modification is not
 *         supported
 * @throws NullPointerException if the specified operator is null or
 *         if the operator result is a null value and this list does
 *         not permit null elements
 *         (<a href="Collection.html#optional-restrictions">optional</a>)
 * @since 1.8
 */
default void replaceAll(UnaryOperator<E> operator) {
    Objects.requireNonNull(operator);
    final ListIterator<E> li = this.listIterator();
    while (li.hasNext()) {
        li.set(operator.apply(li.next()));
    }
}
```





```java
@Override
@SuppressWarnings("unchecked")
public void replaceAll(UnaryOperator<E> operator) {
    //验证operator不为null
    Objects.requireNonNull(operator);
    final int expectedModCount = modCount;
    //缓冲区元素数量
    final int size = this.size;
    //迭代数组缓冲区，依次转换每一个元素
    for (int i=0; modCount == expectedModCount && i < size; i++) {
        elementData[i] = operator.apply((E) elementData[i]);
    }
    //验证迭代期间数组缓冲区没有被更改
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}
```



### sort

`sort`是`java.util.List`接口1.8新增的默认方法，，如下：

```java
/**
 * Sorts this list according to the order induced by the specified
 * {@link Comparator}.
 *
 * <p>All elements in this list must be <i>mutually comparable</i> using the
 * specified comparator (that is, {@code c.compare(e1, e2)} must not throw
 * a {@code ClassCastException} for any elements {@code e1} and {@code e2}
 * in the list).
 *
 * <p>If the specified comparator is {@code null} then all elements in this
 * list must implement the {@link Comparable} interface and the elements'
 * {@linkplain Comparable natural ordering} should be used.
 *
 * <p>This list must be modifiable, but need not be resizable.
 *
 * @implSpec
 * The default implementation obtains an array containing all elements in
 * this list, sorts the array, and iterates over this list resetting each
 * element from the corresponding position in the array. (This avoids the
 * n<sup>2</sup> log(n) performance that would result from attempting
 * to sort a linked list in place.)
 *
 * @implNote
 * This implementation is a stable, adaptive, iterative mergesort that
 * requires far fewer than n lg(n) comparisons when the input array is
 * partially sorted, while offering the performance of a traditional
 * mergesort when the input array is randomly ordered.  If the input array
 * is nearly sorted, the implementation requires approximately n
 * comparisons.  Temporary storage requirements vary from a small constant
 * for nearly sorted input arrays to n/2 object references for randomly
 * ordered input arrays.
 *
 * <p>The implementation takes equal advantage of ascending and
 * descending order in its input array, and can take advantage of
 * ascending and descending order in different parts of the same
 * input array.  It is well-suited to merging two or more sorted arrays:
 * simply concatenate the arrays and sort the resulting array.
 *
 * <p>The implementation was adapted from Tim Peters's list sort for Python
 * (<a href="http://svn.python.org/projects/python/trunk/Objects/listsort.txt">
 * TimSort</a>).  It uses techniques from Peter McIlroy's "Optimistic
 * Sorting and Information Theoretic Complexity", in Proceedings of the
 * Fourth Annual ACM-SIAM Symposium on Discrete Algorithms, pp 467-474,
 * January 1993.
 *
 * @param c the {@code Comparator} used to compare list elements.
 *          A {@code null} value indicates that the elements'
 *          {@linkplain Comparable natural ordering} should be used
 * @throws ClassCastException if the list contains elements that are not
 *         <i>mutually comparable</i> using the specified comparator
 * @throws UnsupportedOperationException if the list's list-iterator does
 *         not support the {@code set} operation
 * @throws IllegalArgumentException
 *         (<a href="Collection.html#optional-restrictions">optional</a>)
 *         if the comparator is found to violate the {@link Comparator}
 *         contract
 * @since 1.8
 */
@SuppressWarnings({"unchecked", "rawtypes"})
default void sort(Comparator<? super E> c) {
    Object[] a = this.toArray();
    Arrays.sort(a, (Comparator) c);
    ListIterator<E> i = this.listIterator();
    for (Object e : a) {
        i.next();
        i.set((E) e);
    }
}
```



```java
@Override
@SuppressWarnings("unchecked")
public void sort(Comparator<? super E> c) {
    final int expectedModCount = modCount;
    Arrays.sort((E[]) elementData, 0, size, c);
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
    modCount++;
}
```



## 总结

