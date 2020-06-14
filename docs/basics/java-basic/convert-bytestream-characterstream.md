
想要实现字符流和字节流之间的相互转换需要用到两个类：

OutputStreamWriter 是字符流通向字节流的桥梁

InputStreamReader 是字节流通向字符流的桥梁

### 字符流转成字节流

```

public static void main(String[] args) throws IOException {
    File f = new File("test.txt");
    
    // OutputStreamWriter 是字符流通向字节流的桥梁,创建了一个字符流通向字节流的对象
    OutputStreamWriter osw = new OutputStreamWriter(new FileOutputStream(f),"UTF-8");
    
    osw.write("我是字符流转换成字节流输出的");
    osw.close();

}

```

### 字节流转成字符流

```
  public static void main(String[] args) throws IOException {
        
        File f = new File("test.txt");
        
        InputStreamReader inr = new InputStreamReader(new FileInputStream(f),"UTF-8");
        
        char[] buf = new char[1024];
        
        int len = inr.read(buf);
        System.out.println(new String(buf,0,len));
        
        inr.close();

    }

```