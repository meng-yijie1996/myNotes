### 前言

1、当我们执行下面语句的时候，底层进行了什么操作

```java
new ArrayList<Integer>();
```

```java
// Constructs an empty list with an initial capacity of ten.
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

- 底层创建了一个空的Object数组，指向DEFAULTCAPACITY_EMPTY_ELEMENTDATA，初始容量为0。
- 首次执行add方法，会扩容为10，并且elementData会指向新的内存空间。

2、当执行add方法后，如果超过了10，那么会进行扩容，扩容的大小为原值的一半，也就是5个，使用下列方法扩容

```java
Arrays.copyOf(elementData, netCapacity);
```

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

### 单线程环境下

单线程环境的ArrayList是不会有问题的

```java
public class ArrayListNotSafeDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("a");
        for(String element : list) {
            System.out.println(element);
        }
    }
}
```

### 多线程环境

为什么ArrayList是线程不安全的？因为在进行写操作的时候，方法上为了保证并发性，是没有添加synchronized修饰，所以并发写的时候，就会出现问题

```java
/**
 * Appends the specified element to the end of this list.
 * @param e element to be appended to this list
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

当我们同时启动30个线程去操作List的时候

```java
public class ArrayListNotSafeDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(list);
            }, String.valueOf(i)).start();
        }
    }
}
```

这个时候出现了错误，也就是java.util.ConcurrentModificationException，即并发修改异常

```java
java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:911)
	at java.util.ArrayList$Itr.next(ArrayList.java:861)
	at java.util.AbstractCollection.toString(AbstractCollection.java:461)
	at java.lang.String.valueOf(String.java:2994)
	at java.io.PrintStream.println(PrintStream.java:821)
	at arrayliat.ArrayListNotSafeDemo.lambda$main$0(ArrayListNotSafeDemo.java:19)
	at java.lang.Thread.run(Thread.java:748)
```

```java
public void run() {
    if (target != null) {
        target.run();
    }
}
```

```java
public void println(Object x) {
    String s = String.valueOf(x);
    synchronized (this) {
        print(s);
        newLine();
    }
}
```

```java
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```

```java
public String toString() {
    Iterator<E> it = iterator();
    if (! it.hasNext())
        return "[]";
    StringBuilder sb = new StringBuilder();
    sb.append('[');
    for (;;) {
        E e = it.next();
        sb.append(e == this ? "(this Collection)" : e);
        if (! it.hasNext())
            return sb.append(']').toString();
        sb.append(',').append(' ');
    }
}
```

```java
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

```java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

### 解决方案

#### 方案一：Vector

第一种方法，就是不用ArrayList这种不安全的List实现类，而采用Vector，线程安全的

Vector实现线程安全：在方法上加了锁，即synchronized。

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}
```

这样就每次只能够一个线程进行操作，所以不会出现线程不安全的问题，但是因为加锁了，导致并发性基于下降

#### 方案二：Collections.synchronized()

```java
List<String> list = Collections.synchronizedList(new ArrayList<>());
```

采用Collections集合工具类，在ArrayList外面包装一层 同步 机制

#### 方案三：采用JUC里面的方法

CopyOnWriteArrayList：写时复制，主要是一种**读写分离**的思想

写时复制，CopyOnWrite容器即写时复制的容器，往一个容器中添加元素的时候，不直接往当前容器Object[]添加，而是先将Object[]进行copy，复制出一个新的容器object[] newElements，然后新的容器Object[] newElements里添加原始，添加元素完后，在将原容器的引用指向新的容器 setArray(newElements)；

这样做的好处是可以对copyOnWrite容器进行并发的读 ，而不需要加锁，因为当前容器不需要添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器

就是写的时候，把ArrayList扩容一个出来，然后把值填写上去，在通知其他的线程，ArrayList的引用指向扩容后的

查看底层add方法源码

```java
    public boolean add(E e) {
        // 首先需要加锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 然后在末尾扩容一个单位
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            // 然后在把扩容后的空间，填写上需要add的内容
            newElements[len] = e;
            // 最后把内容set到Array中
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```



## HashSet线程不安全

### CopyOnWriteArraySet

- 底层还是使用CopyOnWriteArrayList进行实例化

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E> implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;
    private final CopyOnWriteArrayList<E> al;
    // Creates an empty set.
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
    ...
```

### HashSet底层结构

- HashSet的底层结构是HashMap

```java
public HashSet() {
    map = new HashMap<>();
}
```

但是为什么我调用 HashSet.add()的方法，只需要传递一个元素，而HashMap是需要传递key-value键值对？

首先我们查看hashSet的add方法

```java
    private static final Object PRESENT = new Object();
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

我们能发现但我们调用add的时候，存储一个值进入map中，只是作为key进行存储，而value存储的是一个Object类型的常量，也就是说HashSet只关心key，而不关心value.

## HashMap线程不安全

同理HashMap在多线程环境下，也是不安全的

```java
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0, 8));
                System.out.println(map);
            }, String.valueOf(i)).start();
        }
    }
```

### 解决方法

1、使用Collections.synchronizedMap(new HashMap<>());

2、使用 ConcurrentHashMap

```java
Map<String, String> map = new ConcurrentHashMap<>();
```