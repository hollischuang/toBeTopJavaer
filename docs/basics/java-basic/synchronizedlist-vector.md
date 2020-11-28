Vector是java.util包中的一个类。 SynchronizedList是java.util.Collections中的一个静态内部类。

在多线程的场景中可以直接使用Vector类，也可以使用Collections.synchronizedList(List<t> list)方法来返回一个线程安全的List。</t>

**那么，到底SynchronizedList和Vector有没有区别，为什么java api要提供这两种线程安全的List的实现方式呢？**

首先，我们知道Vector和Arraylist都是List的子类，他们底层的实现都是一样的。所以这里比较如下两个`list1`和`list2`的区别：

    List<String> list = new ArrayList<String>();
    List list2 =  Collections.synchronizedList(list);
    Vector<String> list1 = new Vector<String>();
    

## 一、比较几个重要的方法。

### 1\.1 add方法

**Vector的实现：**

    public void add(int index, E element) {
        insertElementAt(element, index);
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
    
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    

**synchronizedList的实现：**

    public void add(int index, E element) {
       synchronized (mutex) {
           list.add(index, element);
       }
    }
    

这里，使用同步代码块的方式调用ArrayList的add()方法。ArrayList的add方法内容如下：

    public void add(int index, E element) {
        rangeCheckForAdd(index);
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        ensureExplicitCapacity(minCapacity);
    }
    

从上面两段代码中发现有两处不同： **1\.Vector使用同步方法实现，synchronizedList使用同步代码块实现。 2.两者的扩充数组容量方式不一样（两者的add方法在扩容方面的差别也就是ArrayList和Vector的差别。）**

### 1\.2 remove方法

**synchronizedList的实现：**

    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);}
    }
    

ArrayList类的remove方法内容如下：

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
    

**Vector的实现：**

    public synchronized E remove(int index) {
            modCount++;
            if (index >= elementCount)
                throw new ArrayIndexOutOfBoundsException(index);
            E oldValue = elementData(index);
    
            int numMoved = elementCount - index - 1;
            if (numMoved > 0)
                System.arraycopy(elementData, index+1, elementData, index,
                                 numMoved);
            elementData[--elementCount] = null; // Let gc do its work
    
            return oldValue;
        }
    

**从remove方法中我们发现除了一个使用同步方法，一个使用同步代码块之外几乎无任何区别。**

> 通过比较其他方法，我们发现，SynchronizedList里面实现的方法几乎都是使用同步代码块包上List的方法。如果该List是ArrayList,那么，SynchronizedList和Vector的一个比较明显区别就是一个使用了同步代码块，一个使用了同步方法。

## 三、区别分析

**数据增长区别**

> 从内部实现机制来讲ArrayList和Vector都是使用数组(Array)来控制集合中的对象。当你向这两种类型中增加元素的时候，如果元素的数目超出了内部数组目前的长度它们都需要扩展内部数组的长度，Vector缺省情况下自动增长原来一倍的数组长度，ArrayList是原来的50%,所以最后你获得的这个集合所占的空间总是比你实际需要的要大。所以如果你要在集合中保存大量的数据那么使用Vector有一些优势，因为你可以通过设置集合的初始化大小来避免不必要的资源开销。

**同步代码块和同步方法的区别** 

1.同步代码块在锁定的范围上可能比同步方法要小，一般来说锁的范围大小和性能是成反比的。 

2.同步块可以更加精确的控制锁的作用域（锁的作用域就是从锁被获取到其被释放的时间），同步方法的锁的作用域就是整个方法。

3.同步代码块可以选择对哪个对象加锁，但是静态方法只能给this对象加锁。

**同步代码块和同步方法的区别** 1.同步代码块在锁定的范围上可能比同步方法要小，一般来说锁的范围大小和性能是成反比的。 2.同步块可以更加精确的控制锁的作用域（锁的作用域就是从锁被获取到其被释放的时间），同步方法的锁的作用域就是整个方法。 3.静态代码块可以选择对哪个对象加锁，但是静态方法只能给this对象加锁。

> 因为SynchronizedList只是使用同步代码块包裹了ArrayList的方法，而ArrayList和Vector中同名方法的方法体内容并无太大差异，所以在锁定范围和锁的作用域上两者并无却别。 在锁定的对象区别上，SynchronizedList的同步代码块锁定的是mutex对象，Vector锁定的是this对象。那么mutex对象又是什么呢？ 其实SynchronizedList有一个构造函数可以传入一个Object,如果在调用的时候显示的传入一个对象，那么锁定的就是用户传入的对象。如果没有指定，那么锁定的也是this对象。

所以，SynchronizedList和Vector的区别目前为止有两点： 1.如果使用add方法，那么他们的扩容机制不一样。 2.SynchronizedList可以指定锁定的对象。

但是，凡事都有但是。 SynchronizedList中实现的类并没有都使用synchronized同步代码块。其中有listIterator和listIterator(int index)并没有做同步处理。但是Vector却对该方法加了方法锁。 所以说，在使用SynchronizedList进行遍历的时候要手动加锁。

但是，但是之后还有但是。

之前的比较都是基于我们将ArrayList转成SynchronizedList。那么如果我们想把LinkedList变成线程安全的，或者说我想要方便在中间插入和删除的同步的链表，那么我可以将已有的LinkedList直接转成 SynchronizedList，而不用改变他的底层数据结构。而这一点是Vector无法做到的，因为他的底层结构就是使用数组实现的，这个是无法更改的。

所以，最后，SynchronizedList和Vector最主要的区别： **1\.SynchronizedList有很好的扩展和兼容功能。他可以将所有的List的子类转成线程安全的类。** **2\.使用SynchronizedList的时候，进行遍历时要手动进行同步处理**。 **3\.SynchronizedList可以指定锁定的对象。**
