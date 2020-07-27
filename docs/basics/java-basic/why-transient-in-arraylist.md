

`ArrayList`使用了`transient`关键字进行存储优化，而`Vector`没有这样做，为什么？

### ArrayList

    /** 
         * Save the state of the <tt>ArrayList</tt> instance to a stream (that 
         * is, serialize it). 
         * 
         * @serialData The length of the array backing the <tt>ArrayList</tt> 
         *             instance is emitted (int), followed by all of its elements 
         *             (each an <tt>Object</tt>) in the proper order. 
         */  
        private void writeObject(java.io.ObjectOutputStream s)  
            throws java.io.IOException{  
            // Write out element count, and any hidden stuff  
            int expectedModCount = modCount;  
            s.defaultWriteObject();  
    
            // Write out array length  
            s.writeInt(elementData.length);  
    
            // Write out all elements in the proper order.  
            for (int i=0; i<size; i++)  
                s.writeObject(elementData[i]);  
    
            if (modCount != expectedModCount) {  
                throw new ConcurrentModificationException();  
            }  
    
        }  
    

ArrayList实现了writeObject方法，可以看到只保存了非null的数组位置上的数据。即list的size个数的elementData。需要额外注意的一点是，ArrayList的实现，提供了fast-fail机制，可以提供弱一致性。

### Vector

    /**
         * Save the state of the {@code Vector} instance to a stream (that
         * is, serialize it).
         * This method performs synchronization to ensure the consistency
         * of the serialized data.
         */
        private void writeObject(java.io.ObjectOutputStream s)
                throws java.io.IOException {
            final java.io.ObjectOutputStream.PutField fields = s.putFields();
            final Object[] data;
            synchronized (this) {
                fields.put("capacityIncrement", capacityIncrement);
                fields.put("elementCount", elementCount);
                data = elementData.clone();
            }
            fields.put("elementData", data);
            s.writeFields();
        }
    

Vector也实现了writeObject方法，但方法并没有像ArrayList一样进行优化存储，实现语句是

`data = elementData.clone();`

clone()的时候会把null值也拷贝。所以保存相同内容的Vector与ArrayList，Vector的占用的字节比ArrayList要多。

可以测试一下，序列化存储相同内容的Vector与ArrayList，分别到一个文本文件中去。* Vector需要243字节* ArrayList需要135字节 分析：

ArrayList是非同步实现的一个单线程下较为高效的数据结构（相比Vector来说）。 ArrayList只通过一个修改记录字段提供弱一致性，主要用在迭代器里。没有同步方法。 即上面提到的Fast-fail机制.ArrayList的存储结构定义为transient，重写writeObject来实现自定义的序列化，优化了存储。

Vector是多线程环境下更为可靠的数据结构，所有方法都实现了同步。

### 区别

> 同步处理：Vector同步，ArrayList非同步 Vector缺省情况下增长原来一倍的数组长度，ArrayList是0.5倍. ArrayList: int newCapacity = oldCapacity + (oldCapacity >> 1); ArrayList自动扩大容量为原来的1.5倍（实现的时候，方法会传入一个期望的最小容量，若扩容后容量仍然小于最小容量，那么容量就为传入的最小容量。扩容的时候使用的Arrays.copyOf方法最终调用native方法进行新数组创建和数据拷贝）
>  
> Vector: int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
>  
> Vector指定了`initialCapacity，capacityIncrement`来初始化的时候，每次增长`capacityIncrement`