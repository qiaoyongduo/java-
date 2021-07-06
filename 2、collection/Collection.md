# Collection

## 前言

​	java中集合是基础知识很重要的部分，我们需要了解他的底层原理，扩容等等相关知识集合分为`Collection`和`Map`两种体系。下面先介绍Collection的集合类的继承树如下图所示

![](F:\3.学习文档\学习计划及文档\java-StudyPlan\2、collection\1.png)

## Collection接口介绍

根据上方的继承树可以知道，Collection 接口有 3 种子类型集合: `List`、`Set` 和 `Queue`，AbstractCollection 是 Java 集合框架中 [Collection 接口](http://blog.csdn.net/u011240877/article/details/52773577) 的一个直接实现类，Collection 下的大多数子类都继承 AbstractCollection ，比如 List 的实现类, Set的实现类。

## List

List是Java中比较常用的集合类，关于List接口有很多实现类，本次介绍其中几个重点的实现ArrayList、LinkedList和Vector之间的关系和区别。

List 是一个接口，它继承于Collection的接口，List中元素可以重复，并且是有序的（这里的有序指的是按照放入的顺序进行存储。如按照顺序把1，2，3存入List，那么，从List中遍历出来的顺序也是1，2，3）。

ArrayList、 LinkedList 和 Vector都实现了List接口，是List的三种实现，所以在用法上非常相似。他们之间的主要区别体现在不同操作的性能上。后面会详细分析。

### **ArrayList**

`ArrayList` 的底层是数组队列，相当于动态数组。与 Java 中的数组相比，它的容量能动态增长。在添加大量元素前，应用程序可以使用`ensureCapacity`操作来增加 `ArrayList` 实例的容量。这可以减少递增式再分配的数量。

`ArrayList`继承于 **`AbstractList`** ，实现了 **`List`**, **`RandomAccess`**, **`Cloneable`**, **`java.io.Serializable`** 这些接口。

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{

  }
```

- `RandomAccess` 是一个标志接口，表明实现这个这个接口的 List 集合是支持**快速随机访问**的。在 `ArrayList` 中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。
- `ArrayList` 实现了 **`Cloneable` 接口** ，即覆盖了函数`clone()`，能被克隆。
- `ArrayList` 实现了 `java.io.Serializable `接口，这意味着`ArrayList`支持序列化，能通过序列化去传输。

#### ArrayList扩容机制

以无参数构造方法创建 ArrayList 时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为 10。以下是源码分析扩容机制

add方法

```java
    /**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
   //添加元素之前，先调用ensureCapacityInternal方法
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }
```

**注意** ：JDK11 移除了 `ensureCapacityInternal()` 和 `ensureExplicitCapacity()` 方法

`ensureExplicitCapacity()` 方法

```java
private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
//得到最小扩容量
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 获取默认的容量和传入参数的较大值
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}
```

`ensureExplicitCapacity()` 方法

```java
//判断是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
	modCount++;

	// overflow-conscious code
	if (minCapacity - elementData.length > 0)
		//调用grow方法进行扩容，调用此方法代表已经开始扩容了
		grow(minCapacity);
}
```

- 当我们要 add 进第 1 个元素到 ArrayList 时，elementData.length 为 0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为 10。此时，`minCapacity - elementData.length > 0`成立，所以会进入 `grow(minCapacity)` 方法。
- 当 add 第 2 个元素时，minCapacity 为 2，此时 e lementData.length(容量)在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0` 不成立，所以不会进入 （执行）`grow(minCapacity)` 方法。
- 添加第 3、4···到第 10 个元素时，依然不会执行 grow 方法，数组容量都为 10。

 `grow()` 方法

```java
    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
       // 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
       //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

- 当 add 第 1 个元素时，oldCapacity 为 0，经比较后第一个 if 判断成立，newCapacity = minCapacity(为 10)。但是第二个 if 判断不会成立，即 newCapacity 不比 MAX_ARRAY_SIZE 大，则不会进入 `hugeCapacity` 方法。数组容量为 10，add 方法中 return true,size 增为 1。
- 当 add 第 11 个元素进入 grow 方法时，newCapacity 为 15，比 minCapacity（为 11）大，第一个 if 判断不成立。新容量没有大于数组最大 size，不会进入 hugeCapacity 方法。数组容量扩为 15，add 方法中 return true,size 增为 11。
- 以此类推······

从上面 `grow()` 方法源码我们知道： 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，如果 minCapacity 大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。

```java
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //对minCapacity和MAX_ARRAY_SIZE进行比较
        //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
        //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
        //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

### Vector

Vector实现了AbstractList抽象类和List接口,和ArrayList一样是基于Array存储的，Vector 是线程安全的,在大多数方法上存在synchronized关键字，Vector使用的都是独占锁，独占式锁在同一时刻只有一个线程能够获取，效率太低。并不推荐使用

可以考虑使用：Collections.synchronizedList或者CopyOnWriteArrayList （但也要按需要和了解后使用）

### LinkedList 

LinkedList类是双向链表（如果对链表结构不了解的可以看我另外一篇数据结构文章）,列表中的每个节点都包含了对前一个和后一个元素的引用。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{}
```

LinkedList 是一个继承于AbstractSequentialList的双向链表。
LinkedList 可以被当作堆栈、队列或双端队列进行操作。
LinkedList 实现 List 接口，所以能对它进行队列操作。
LinkedList 实现 Deque 接口，能将LinkedList当作双端队列使用。
LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
LinkedList 实现java.io.Serializable接口，所以LinkedList支持序列化，能通过序列化去传输。
LinkedList 是非同步的。

###  Arraylist 和 Vector 的区别

1. `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[ ]`存储，适用于频繁的查找工作，线程不安全 ；
2. `Vector` 是 `List` 的古老实现类，底层使用 `Object[ ]`存储，线程安全的。

### Arraylist 与 LinkedList 区别?

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都是不同步的，也就是不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构（JDK1.6 之前为循环链表，JDK1.7 取消了循环）
3. **插入和删除是否受元素位置的影响：** ① **`ArrayList` 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList` 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是 O(1)。但是如果要在指定位置 i 插入和删除元素的话（`add(int index, E element)`）时间复杂度就为 O(n-i)。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。 ② **`LinkedList` 采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O(1)，如果是要在指定位置`i`插入和删除元素的话（`(add(int index, E element)`） 时间复杂度近似为`o(n))`因为需要先移动到指定位置再插入。**
4. **是否支持快速随机访问：** `LinkedList` 不支持高效的随机元素访问，而 `ArrayList` 支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于`get(int index)`方法)。
5. **内存空间占用：** `ArrayList` 的空 间浪费主要体现在在 list 列表的结尾会预留一定的容量空间，而 `LinkedList` 的空间花费则体现在它的每一个元素都需要消耗比 `ArrayList` 更多的空间（因为要存放直接后继和直接前驱以及数据）。

## Set

Set继承于Collection接口，是一个不允许出现重复元素，并且无序的集合，主要有HashSet和TreeSet两大实现类。而LinekdHashSet是继承自HashSet，hashset、linkedset可以存储一个null. treeset不能存储null。Set可以理解为集合，非常类似数据概念中的集合，集合三大特征：1、确定性；2、互异性；3、无序性，因此Set实现类也有类似的特征。

### Hashset

HashSet实现Set接口，底层由HashMap来实现，为哈希表结构，HashSet继承自AbstractSet，实现了Set接口，但是其源码非常少，也非常简单。内部使用HashMap来存储数据，数据存储在HashMap的key中，value都是同一个默认值：

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{}
```

下面讲解HashSet的几个常用的方法

1. add

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

HashSet的确定性，也可以理解为唯一性，是通过HashMap的put方法来保证的，往HashMap中put数据时，如果key是一样的，只会替换key对应的value，不会新插入一条数据。

2. remove

```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

从HashMap中移除一条数据。

3. iterator

```java
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
```

可以看出HashSet的add，remove，iterator，size，contains等等的方法都直接委托给了HashMap

### TreeSet

TreeSet也是基于Map来实现，具体实现**TreeMap**，其底层结构为**红黑树**（特殊的二叉查找树）；与HashSet不同的是，TreeSet具有排序功能，分为自然排序(123456)和自定义排序两类，默认是自然排序；在程序中，我们可以按照任意顺序将元素插入到集合中，等到遍历时TreeSet会按照一定顺序输出--倒序或者升序；

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
{}
```

它继承AbstractSet，实现NavigableSet, Cloneable, Serializable接口。

其中AbstractSet提供 Set 接口的骨干实现，从而最大限度地减少了实现此接口所需的工作。

NavigableSet是扩展的 SortedSet，具有了为给定搜索目标报告最接近匹配项的导航方法，这就意味着它支持一系列的导航方法。比如查找与指定目标最匹配项。Cloneable支持克隆，Serializable支持序列化。

（1）与HashSet同理，TreeSet继承AbstractSet类，获得了Set集合基础实现操作；

（2）TreeSet实现NavigableSet接口，而NavigableSet又扩展了SortedSet接口。这两个接口主要定义了搜索元素的能力，例如给定某个元素，查找该集合中比给定元素大于、小于、等于的元素集合，或者比给定元素大于、小于、等于的元素个数；简单地说，实现NavigableSet接口使得TreeSet具备了元素搜索功能；

（3）TreeSet实现Cloneable接口，意味着它也可以被克隆；

（4）TreeSet实现了Serializable接口，可以被序列化，可以使用hessian协议来传输；

具有如下特点：

- 对插入的元素进行排序，是一个有序的集合（主要与HashSet的区别）;
- 底层使用红黑树结构，而不是哈希表结构；
- 不允许插入Null值；
- 不允许插入重复元素；
- 线程不安全；

### LinekdHashSet

LinkedHashSet是HashSet的一个“扩展版本”，HashSet并不管什么顺序，不同的是LinkedHashSet会维护“插入顺序”。HashSet内部使用HashMap对象来存储它的元素，而LinkedHashSet内部使用LinkedHashMap对象来存储和处理它的元素。

LinkedHashSet并没有自己的方法，所有的方法都继承自它的父类HashSet，因此，对LinkedHashSet的所有操作方式就好像对HashSet操作一样。

唯一的不同是内部使用不同的对象去存储元素。在HashSet中，插入的元素是被当做HashMap的键来保存的，而在LinkedHashSet中被看作是LinkedHashMap的键。

### Set集合怎么实现线程安全？

方案一：
和list一样，使用Colletcions这个工具类syn方法类创建个线程安全的set.

Set<String> synSet = Collections.synchronizedSet(new HashSet<>());

方案二：
使用JUC包里面的CopyOnWriteArraySet

Set<String> copySet = new CopyOnWriteArraySet<>();

### Set总结

通过以上特点可以分析出，三者都保证了元素的唯一性，如果无排序要求可以选用HashSet；如果想取出元素的顺序和放入元素的顺序相同，那么可以选用LinkedHashSet。如果想插入、删除立即排序或者按照一定规则排序可以选用TreeSet。

## Queue

队列，队列是一种比较特殊的线性结构。它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作。进行插入操作的端称为队尾，进行删除操作的端称为队头。
队列中最先插入的元素也将最先被删除，对应的最后插入的元素将最后被删除。因此队列又称为“先进先出”（FIFO—first in first out）的线性表，与栈(FILO-first in last out)刚好相反。

### Queue接口中有以下几个常用实现类：

PriorityQueue：非阻塞、非线程安全、无边界，支持优先级队列实现类。
ConcurrentLinkedQueue：非阻塞、线程安全、无边界，基于链接节点的队列实现类。
ArrayBlockingQueue：阻塞、线程安全、有边界，创建的时候指定大小，一旦创建容量不可改变实现类，默认是不保证线程的公平性，不允许向队列中插入null元素。
LinkedBlockingQueue：阻塞、线程安全、可选有边界，一个由链表结构组成的可选有界阻塞队列实现类，如果未指定容量，那么容量将等于Integer.MAX_VALUE。
PriorityBlockingQueue：阻塞、线程安全、无边界，支持优先级排序的无边界阻塞队列实现类。
DelayQueue：阻塞、线程安全、无边界，使用优先级队列实现的无界阻塞队列实现类，只有在延迟期满时才能从中提取元素。
SynchronousQueue：阻塞、线程安全、无数据队列，不存储元素、没有内部容量的阻塞队列实现类。
LinkedBlockingDeque：阻塞、线程安全、无边界，由链表结构组成的可选范围双向阻塞队列实现类，如果未指定容量，那么容量将等于 Integer.MAX_VALUE

### Queue类中方法：

　　**add**    增加一个元索           如果队列已满，则抛出一个IIIegaISlabEepeplian异常
　　**remove**  移除并返回队列头部的元素  如果队列为空，则抛出一个NoSuchElementException异常
　　**element** 返回队列头部的元素       如果队列为空，则抛出一个NoSuchElementException异常
　　**offer**    添加一个元素并返回true    如果队列已满，则返回false
　　**poll**     移除并返问队列头部的元素  如果队列为空，则返回null
　　**peek**    返回队列头部的元素       如果队列为空，则返回null
　　**put**     添加一个元素           如果队列满，则阻塞
　　**take**    移除并返回队列头部的元素   如果队列为空，则阻塞

1.尽量使用offer()方法添加元素，使用poll()方法移除元素。dd()和remove()方法在失败的时候会抛出异常。
2.peek方法不会删除元素， Retrieves, but does not remove, the head of this queue,

## AbstractCollection

AbstractCollection 是 Java 集合框架中 [Collection 接口](http://blog.csdn.net/u011240877/article/details/52773577) 的一个直接实现类， Collection 下的大多数子类都继承 AbstractCollection ，比如 List 的实现类, Set的实现类。

参考github：https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/ArrayList%E6%BA%90%E7%A0%81+%E6%89%A9%E5%AE%B9%E6%9C%BA%E5%88%B6%E5%88%86%E6%9E%90.md

