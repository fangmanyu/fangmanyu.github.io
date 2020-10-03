# JDK1.8 ArrayList源码分析
- 构造函数
- 扩容机制
- fail-fast机制
- remove在特定方式下的区别

__ArrayList 核心__：
```java
    transient Object[] elementData; 
```
```java
    private int size;
```

## 构造函数
``` java
    private static final int DEFAULT_CAPACITY = 10;

    private static final Object[] EMPTY_ELEMENTDATA = {};

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

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

    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

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

### 无参构造
<code>elementData</code> 被赋值为 <code>DEFAULTCAPACITY_EMPTY_ELEMENTDATA</code> ，此时数组长度为0。

### 初始化容量
当容量大于0时为<code>elementData</code>创建相应长度的Object数组；
当容量等于0时将 <code>elementData</code>置为 <code>EMPTY_ELEMENTDATA</code>;
当容量小于0时抛出<code>IllegalArgumentException</code>异常。

### 初始化集合
使用<code>Collection.toArray()</code>方法将集合元素转变为数组，再赋值给<code>elementData</code>。

<code>elementData.getClass() != Object[].class</code>不太理解含义？？

## 动态扩容
```java
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
 
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
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

- 一般扩充的容量是原来的1.5倍。
- 如果扩充之后的容量达不到所需最小容量(<code>minCapacity</code>)，则新容量是<code>minCapacity</code>。
- 如果新容量大于最大数组长度(<code>MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8</code>)，这是就要考虑<code>minCapacity</code>是否超出了<code>Integer</code>的最大长度。如果是则抛<code>OutOfMemoryError</code>异常，否则新容量为<code>Integer.MAX_VALUE</code>。

## fail-fast机制
fail-fast(快速失败)机制是Collection集合中的一种错误机制。当多个线程对同一个集合的结构进行操作(如：add、remove等)时，就会发生Fail-Fast事件。

```java
    @Test
    void test2() {
        List<Integer> list = new ArrayList<>(10);
        Collections.addAll(list, 1, 2, 3, 4, 5, 6, 7);

        new Thread(() -> {
            for (Integer i :
                    list) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(i);
            }
        }).start();

        new Thread(() -> list.add(10)).start();
    }
```

此时会抛出 `java.util.ConcurrentModificationException`异常：当方法检测到对象的并发修改，但不允许这种修改时就会抛出这个异常。
如果单线程违反了规则，也会抛出该异常。

迭代器的快速失败无法得到保证，它不能保证一定会出现该错误，但是会尽最大可能的抛出`java.util.ConcurrentModificationException`这个异常。

### 源码分析
```java
    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

在`ArrayList`内部类`Itr`中，局部变量`expectedModCount`保存`ArrayList`当前的`modCount`值。

迭代器在调用`next` 和 `remove`方法时都会调用`checkForComodification`方法检测`modCount`值是否改变，如果改变则抛出`ConcurrentModificationException`。

`modCount` 是`AbstractList`中的属性，在`ArrayList`的扩容，添加、移除等操作时，都会将`modCount`自增。
```java
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

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

## remove在特定方式下的区别
`ArrayList`的`remove`有两个：
```java
public E remove(int index); 
```
```java
public boolean remove(Object o); 
```
当`ArrayList`类型为`Integer`类型时，使用`remove 方法会有一定的差异。`
```java
    List<Integer> list = new ArrayList<>(10);
    Collections.addAll(list, 1, 2, 3, 4, 5, 6, 7, 128);
    list.remove(5);
    System.out.println(list);
```
此时调用的是```public E remove(int index); ```方法，删除第6个元素。

```java
list.remove(Integer.valueOf(5));
```
如果我们想调用参数是Object的方法，需要将值转换成`Integer`类型。