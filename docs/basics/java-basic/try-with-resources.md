Java里，对于文件操作IO流、数据库连接等开销非常昂贵的资源，用完之后必须及时通过close方法将其关闭，否则资源会一直处于打开状态，可能会导致内存泄露等问题。

关闭资源的常用方式就是在finally块里是释放，即调用close方法。比如，我们经常会写这样的代码：

    public static void main(String[] args) {
        BufferedReader br = null;
        try {
            String line;
            br = new BufferedReader(new FileReader("d:\\hollischuang.xml"));
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            // handle exception
        } finally {
            try {
                if (br != null) {
                    br.close();
                }
            } catch (IOException ex) {
                // handle exception
            }
        }
    }
    
从Java 7开始，jdk提供了一种更好的方式关闭资源，使用try-with-resources语句，改写一下上面的代码，效果如下：

    public static void main(String... args) {
        try (BufferedReader br = new BufferedReader(new FileReader("d:\\ hollischuang.xml"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            // handle exception
        }
    }
    
看，这简直是一大福音啊，虽然我之前一般使用IOUtils去关闭流，并不会使用在finally中写很多代码的方式，但是这种新的语法糖看上去好像优雅很多呢。看下他的背后：
    
    public static transient void main(String args[])
        {
            BufferedReader br;
            Throwable throwable;
            br = new BufferedReader(new FileReader("d:\\ hollischuang.xml"));
            throwable = null;
            String line;
            try
            {
                while((line = br.readLine()) != null)
                    System.out.println(line);
            }
            catch(Throwable throwable2)
            {
                throwable = throwable2;
                throw throwable2;
            }
            if(br != null)
                if(throwable != null)
                    try
                    {
                        br.close();
                    }
                    catch(Throwable throwable1)
                    {
                        throwable.addSuppressed(throwable1);
                    }
                else
                    br.close();
                break MISSING_BLOCK_LABEL_113;
                Exception exception;
                exception;
                if(br != null)
                    if(throwable != null)
                        try
                        {
                            br.close();
                        }
                        catch(Throwable throwable3)
                          {
                            throwable.addSuppressed(throwable3);
                        }
                    else
                        br.close();
            throw exception;
            IOException ioexception;
            ioexception;
        }
    }
    
其实背后的原理也很简单，那些我们没有做的关闭资源的操作，编译器都帮我们做了。所以，再次印证了，语法糖的作用就是方便程序员的使用，但最终还是要转成编译器认识的语言。