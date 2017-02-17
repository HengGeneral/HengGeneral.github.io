---
layout: post
title: Java 集合框架之List
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "java 集合框架"
---

## Collection 

### List

List, 有序的集合接口。使用该集合时, 用户可以控制元素插入到集合中的位置, 也可以使用元素的索引来访问该元素。
和 Set 不同的是, List允许集合中有重复的元素。 当然, 若是想实现不包含重复元素的list也是可以的, 但还是希望尽量不要这样做。
接口中的方法如下:

```
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean addAll(int index, Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    ...
```

常用的子类有LinkedList, ArrayList 和 Vector等, 如图所示:

![ListTree](/images/java/listTree.png)

***

**注意**: List可以在集合中插入它本身作为集合的某个元素, 尤其要小心 equals() 方法和 hashcode() 方法可能就不好使了。

***

#### ArrayList

基于List接口的可变长数组的实现, 该类和 Vector 几乎相当, 除了 ArrayList 是 unsynchronized。

其中, size, isEmpty, get, set, iterator 和 listIterator操作的时间复杂度是O(1),
add操作是均摊常量时间(amortized constant time), 其它操作的时间复杂度是O(n)。

每个 ArrayList 对象都有 capacity 属性, 用于表示存放 list 中元素(可以是null)的数组的容量。
同时, capacity至少与 list.size() 一样大。

由于 ArrayList 是 unsynchronized, 所以需要对 list 的操作进行封装, 然后对某个对象进行同步控制。
当然, 也可以使用 Collections.synchronizedList() 方法对该 ArrayList 对象进行包装, 做法如下:

```
    List list = Collections.synchronizedList(new ArrayList(...));
```
 
另外, iterator()和 listIterator()方法都是 fail-fast(快速失败): 在 iterator 创建之后, 任何对 list 的结构化修改时(如add(), remove()方法),
iterator 都会抛出 ConcurrentModificationException。因此, 在遇到并发修改时, 迭代器为了避免有风险以及不确定地行为的出现, 都会快速失败。
注意，迭代器的快速失败行为无法得到保证。一般来说，由于 list 是 unsynchronized, 所以无法避免任何不同步的并发修改,
而快速失败迭代器只是会尽最大努力抛出 ConcurrentModificationException。
迭代器的快速失败行为应该仅用于检测 bug。因此，依赖于这种做法是无法给程序正确性提供保证的。

ArrayList常见的方法(增、删、改、查)如下:

#### add()方法

```
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     */
    transient Object[] elementData; // non-private to simplify nested class access
    
    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
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

#### contains()方法

```
    /**
     * @return true if this list contains the specified element
     */
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }

    /**
     * Formally, returns the lowest index i such that o==null?get(i)==null:o.equals(get(i)),
     * or -1 if there is no such index.
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

#### remove()方法

```
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

    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

#### set()方法

```
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```    

#### fail-fast机制

迭代器的快速失败机制(fail-fast)是在迭代器初始时获取 modCount,并存入 expectedModCount 变量中。
然后再迭代器的next(), add(), set() 和 remove()方法前检查 modCount 和 expectedModCount 是否相等(ArrayList表结构的增删操作都会将modCount加1)。
如果不等, 则抛出 ConcurrentModificationException 异常。


```
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
        
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
```        

#### LinkedList

LinkedList就是所说的双向链表(双向链表的顺序实现是ArrayDeque)。和 ArrayList 一样, LinkedList也是 unsynchronized。
如果多个线程并发地对链表进行操作, 并且至少一个线程可以操作成功, 它必须在外部进行同步: 对新对象同步或者使用 Collections.synchronizedList()方法进行包装。
同时, iterator也支持 fail-fast 机制。

常用的方法如下:

#### add()方法

```
    transient int size = 0;

    transient Node<E> first;

    transient Node<E> last;
    
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
    
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    public void addFirst(E e) {
        linkFirst(e);
    }

    public void addLast(E e) {
        linkLast(e);
    }
    
    private void linkFirst(E e) {
        final Node<E> f = first;
        final Node<E> newNode = new Node<>(null, e, f);
        first = newNode;
        if (f == null)
            last = newNode;
        else
            f.prev = newNode;
        size++;
        modCount++;
    }
    
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

    /**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

#### remove()方法

```
    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }

    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }

    E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
    
    public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

#### contains()方法

```
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }

    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

#### get()方法

```
    public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
    
    public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
    
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }

    Node<E> node(int index) {
        // assert isElementIndex(index);

        //如果靠近前半部分, 则从头开始检索; 否则, 从尾开始检索;
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

#### Vector

Vector数据存储和ArrayList几乎一样, 区别是增删等方法是synchronized。
另外, iterator 也是 fail-fast(快速失败): 在 iterator 创建之后, 任何对 list 的结构化修改时(如add(), remove()方法),
iterator 都会抛出 ConcurrentModificationException。因此, 在遇到并发修改时, 迭代器为了避免有风险以及不确定地行为的出现, 都会快速失败。
注意，迭代器的快速失败行为无法得到保证。由于迭代器中的方法并不完全是同步的, 也无法避免任何不同步的并发修改,
所以快速失败迭代器也只是会尽最大努力抛出 ConcurrentModificationException。
所以, fail-fast机制并不是针对synchronized和unsynchronized, 而是为迭代器准备的。为什么呢? 
因为创建了迭代器时, expectCount赋值为modCount, 当一个执行next()时返回元素E, 之后vector又直接调用add()在该位置添加一个元素, 
T1下次next()方法会再次返回E元素, fail-fast就是为了避免这种错误。
另外, 迭代器下面的方法并不是完全同步的, 变量也不是volatile, 若有并发地操作, 还是会出现不一致的情况, 所以fail-fast也只是尽最大努力抛出 ConcurrentModificationException。

#### add()方法
```
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }
    
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
       
    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        ensureCapacityHelper(elementCount + 1);
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        elementData[index] = obj;
        elementCount++;
    }
           
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }           
```

#### remove()方法
```
    protected int elementCount;
    
    public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    }
    
    public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj);
        if (i >= 0) {
            removeElementAt(i);
            return true;
        }
        return false;
    }
```

#### get()方法

```
    public synchronized int lastIndexOf(Object o, int index) {
        if (index >= elementCount)
            throw new IndexOutOfBoundsException(index + " >= "+ elementCount);

        if (o == null) {
            for (int i = index; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    public synchronized E elementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }

        return elementData(index);
    }

    public synchronized E firstElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(0);
    }

    public synchronized E lastElement() {
        if (elementCount == 0) {
            throw new NoSuchElementException();
        }
        return elementData(elementCount - 1);
    }

    //并不是线程安全的
    public Enumeration<E> elements() {
        return new Enumeration<E>() {
            int count = 0;

            public boolean hasMoreElements() {
                return count < elementCount;
            }

            public E nextElement() {
                synchronized (Vector.this) {
                    if (count < elementCount) {
                        return elementData(count++);
                    }
                }
                throw new NoSuchElementException("Vector Enumeration");
            }
        };
    }
```

#### set()方法
```
    public synchronized void setElementAt(E obj, int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        elementData[index] = obj;
    }

    public synchronized E set(int index, E element) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

### Stack

stack(栈), 后进先出的数据结构, 继承了Vector类并扩展了5个方法, push、pop、peek、empty和search方法, 代码如下:

```
    public E push(E item) {
        addElement(item); //参见Vector类addElement()方法

        return item;
    }
    
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1); //参见Vector类removeElementAt()方法

        return obj;
    }
  
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1); //参见Vector类elementAt()方法
    }
        
    public boolean empty() {
        return size() == 0;
    }
            
    public synchronized int search(Object o) {
        int i = lastIndexOf(o); //参见Vector类lastIndexOf()方法

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }            
```    

### PriorityQueue

优先级队列, 也是数据结构中的小顶堆。队列中的元素存放在数组中, 按照自然序或者自定义的比较器来存放的, 若没有提供自定义的比较器时, non-comparable
的对象时不允许插入。和ArrayList, LinkedList不一样的是, 优先级队列并不允许null元素。

#### add()方法

```
    public boolean add(E e) {
        return offer(e);
    }

    /**
     * Inserts the specified element into this priority queue.
     *
     * @return {@code true} (as specified by {@link Queue#offer})
     * @throws ClassCastException if the specified element cannot be
     *         compared with elements currently in this priority queue
     *         according to the priority queue's ordering
     * @throws NullPointerException if the specified element is null
     */
    public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        modCount++;
        int i = size;
        if (i >= queue.length)
            grow(i + 1);
        size = i + 1;
        if (i == 0)
            queue[0] = e;
        else
            siftUp(i, e);
        return true;
    }
    
    private void grow(int minCapacity) {
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50%
        int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                         (oldCapacity + 2) :
                                         (oldCapacity >> 1));
        // overflow-conscious code
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        queue = Arrays.copyOf(queue, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
    
    private void siftUp(int k, E x) {
        if (comparator != null)
            siftUpUsingComparator(k, x);
        else
            siftUpComparable(k, x);
    }

    @SuppressWarnings("unchecked")
    private void siftUpComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>) x;
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            if (key.compareTo((E) e) >= 0)
                break;
            queue[k] = e;
            k = parent;
        }
        queue[k] = key;
    }

    @SuppressWarnings("unchecked")
    private void siftUpUsingComparator(int k, E x) {
        while (k > 0) {
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            if (comparator.compare(x, (E) e) >= 0)
                break;
            queue[k] = e;
            k = parent;
        }
        queue[k] = x;
    }
    
    public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        modCount++;
        int i = size;
        if (i >= queue.length)
            grow(i + 1);
        size = i + 1;
        if (i == 0)
            queue[0] = e;
        else
            siftUp(i, e);
        return true;
    }    
```

#### query()方法

```
    public E peek() {
        return (size == 0) ? null : (E) queue[0];
    }

    private int indexOf(Object o) {
        if (o != null) {
            for (int i = 0; i < size; i++)
                if (o.equals(queue[i]))
                    return i;
        }
        return -1;
    }
    
    
```

#### remove()方法
```
    public boolean remove(Object o) {
        int i = indexOf(o);
        if (i == -1)
            return false;
        else {
            removeAt(i);
            return true;
        }
    }

    boolean removeEq(Object o) {
        for (int i = 0; i < size; i++) {
            if (o == queue[i]) {
                removeAt(i);
                return true;
            }
        }
        return false;
    }
    
    private E removeAt(int i) {
        // assert i >= 0 && i < size;
        modCount++;
        int s = --size;
        if (s == i) // removed last element
            queue[i] = null;
        else {
            E moved = (E) queue[s];
            queue[s] = null;
            siftDown(i, moved);
            if (queue[i] == moved) {
                siftUp(i, moved);
                if (queue[i] != moved)
                    return moved;
            }
        }
        return null;
    }
        
    private void siftDown(int k, E x) {
        if (comparator != null)
            siftDownUsingComparator(k, x);
        else
            siftDownComparable(k, x);
    }

    @SuppressWarnings("unchecked")
    private void siftDownComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>)x;
        int half = size >>> 1;        // loop while a non-leaf
        while (k < half) {
            int child = (k << 1) + 1; // assume left child is least
            Object c = queue[child];
            int right = child + 1;
            if (right < size &&
                ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
                c = queue[child = right];
            if (key.compareTo((E) c) <= 0)
                break;
            queue[k] = c;
            k = child;
        }
        queue[k] = key;
    }

    @SuppressWarnings("unchecked")
    private void siftDownUsingComparator(int k, E x) {
        int half = size >>> 1;
        while (k < half) {
            int child = (k << 1) + 1;
            Object c = queue[child];
            int right = child + 1;
            if (right < size &&
                comparator.compare((E) c, (E) queue[right]) > 0)
                c = queue[child = right];
            if (comparator.compare(x, (E) c) <= 0)
                break;
            queue[k] = c;
            k = child;
        }
        queue[k] = x;
    }        
```

### 参考文献:
1. https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html
2. http://stackoverflow.com/questions/200384/constant-amortized-time/
3. http://stackoverflow.com/questions/4479554/why-vector-methods-iterator-and-listiterator-are-fail-fast
4. https://www.ibm.com/developerworks/cn/java/j-jtp09263/