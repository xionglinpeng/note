# Java序列化

Java序列化在JDK 1.1中引入，是Java内核的重要特性之一。

对象序列化方式：实现`Serializable`接口。

`Serializable`是一个标记接口，不需要实现任何字段和方法。

序列化处理是通过`ObjectInputStream`和`ObjectOutputStream`实现。

缺点：

1. 只支持Java语言，不支持其他语言。
2. 性能差，序列化后的码流大，对于引用过深的对象序列化容易引起`OOM`异常。

相关特性

1. 被序列化对象必须实现`serializable`口，否则会报错`java.io.NotSerializableException`。

2. 静态属性不会被序列化。

3. 被标记为`transient`属性不会被序列化。

4. 被序列化对象尽可能声明`serialVersionUID`如果不声明`serialVersionUID`，那么`JDK`将会为其默认声明，但是在反序列化是可能发生属性变化，此时将不能被反序列化，会报错`java.io.NotSerializableException`。

   > **关于serialVersionUID描述**
   >
   > 序列化运行时使用一个称为`serialVersionUID`的版本号与每个可序列化的类型相关联，该序列号在序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。如果接收者加载的该对象的类的`serialVersionUID`与对应的发送者的类的版本号不同，则反序列化将会导致`InvalidClassException`。可序列化类可以通过声明`serialVersionUID`字段显式的声明自己的`serialVersionUID`。
   >
   > 例如：
   >
   > ```java
   > private static final long serialVersionUID = 1L;
   > ```
   >
   > 如果可序列化类未显式声明`serialVersionUID`，则序列化运行时将基于该类的各个方面计算该类的默认`serialVersionUID`值，如"Java(TM)对象序列化规范"中所述。不过，强烈建议所有可序列化类都显式声明`serialVersionUID`值，原因是计算默认的`serialVersionUID`对类的详细信息具有较高的敏感性，根据编译器实现的不同可能千差万别，这样在反序列化过程中可能会导致意外的`InvalidClassException`。因此，为保证`serialVersionUID`值跨不同java编译器实现的一致性，反序列化类必须声明一个明确的`serialVersionUID`值。还强烈建议使用`private`修饰符显式声明`serialVersionUID`，原因是这种声明仅应用于直接声明类 -- `serialVersionUID`字段作为继承成员没有用处。数组类不能声明一个明确的`serialVersionUID`，因此它们总是具有默认的计算值。但是数组类没有匹配`serialVersionUID`值得要求。

5. 根据上述说明，被序列化对象中的对象属性也必须实现`Serializable`接口，并尽可能声明`serialVersionUID`。



*java序列化和反序列化代码示例：*

1. 定义序列化接口

   ```java
   /**
    * <p>序列化接口</p>
    *
    * @author xlp
    * @date 2020/6/18 下午5:29
    * @since 1.0.0
    */
   public interface ISerializer {
   
       /**
        * 对象序列化
        */
       <T> byte[] serialize(T obj);
   
       /**
        * 对象反序列化
        */
       <T> T deserialize(byte[] data,Class<T> clazz);
   
       /**
        * 将对象序列化到文件中
        */
       <T> void serializeToFile(T obj,String fileName);
   
       /**
        * 从文件中反序列化成对象
        */
       <T> T deserializeFromFile(String fileName,Class<T> clazz);
   }
   ```

   

2. 实现序列化接口

   ```java
   public class JavaSerializer implements ISerializer {
   
       @Override
       public <T> byte[] serialize(T obj) {
           ByteArrayOutputStream baos = new ByteArrayOutputStream();
           try (ObjectOutputStream oos = new ObjectOutputStream(baos)){
               oos.writeObject(obj);
           } catch (IOException e) {
               throw new JavaSerializerException(e);
           }
           return baos.toByteArray();
       }
   
       @Override
       public <T> T deserialize(byte[] data, Class<T> clazz) {
           ByteArrayInputStream bais = new ByteArrayInputStream(data);
           try (ObjectInputStream ois = new ObjectInputStream(bais)){
               return clazz.cast(ois.readObject());
           } catch (Exception e) {
               throw new JavaSerializerException(e);
           }
       }
   
       @Override
       public <T> void serializeToFile(T obj, String fileName) {
           try (FileOutputStream fos = new FileOutputStream(fileName);
                ObjectOutputStream oos = new ObjectOutputStream(fos)){
               oos.writeObject(obj);
           } catch (Exception e) {
               throw new JavaSerializerException(e);
           }
       }
   
       @Override
       public <T> T deserializeFromFile(String fileName, Class<T> clazz) {
           try (FileInputStream fis = new FileInputStream(fileName);
                ObjectInputStream ois = new ObjectInputStream(fis)){
               return clazz.cast(ois.readObject());
           } catch (Exception e) {
               throw new JavaSerializerException(e);
           }
       }
   }
   ```

3. 定义被序列化实体

   ```java
   @Data
   public class User implements Serializable {
   
       private static final long serialVersionUID = 899163613462543355L;
       private static final String FILE = "/opt/xlp/user.out";
   
       private byte b;
       private short s;
       private int i;
       private long l;
       private double d;
       private float f;
       private boolean bool;
       private char c;
       private String str;
       private static String iStatic = "xavier";
       private transient String iTransient = "transient";
       
       //此属性用于验证：如果不声明serialVersionUID，在序列化此对象之后，在打开此属性，在反序列化报错NotSerializableException
       //private String name;
   
       private static User getInstance(){
           User user = new User();
           user.setB(Byte.parseByte("1"));
           user.setS(Short.parseShort("2"));
           user.setI(3);
           user.setL(4L);
           user.setD(1.1d);
           user.setF(2.1f);
           user.setBool(true);
           user.setC('a');
           user.setStr("str");
           return user;
       }
   
       @Override
       public String toString() {
           return "User{" + "b=" + b + ", s=" + s + ", i=" + i + ", l=" + l +
                   ", d=" + d + ", f=" + f + ", bool=" + bool + ", c=" + c +
                   ", str='" + str + "', iTransient='" + iTransient + 
                   "', iStatic='" + iStatic + "'}";
       }
   }
   ```

   

4. 测试序列化

   ```java
   public static void main(String[] args) throws Exception {
       ISerializer serializer = new JavaSerializer();
       //序列化
       serializer.serializeToFile(User.getInstance(),FILE);
       //更改静态属性的值，以验证静态属性不会被序列化
       iStatic = "name";
       //反序列化
       User user = serializer.deserializeFromFile(FILE, User.class);
       //输出反序列化结果
       System.out.println(user);
       //反序列化输出结果
       //User{b=1, s=2, i=3, l=4, d=1.1, f=2.1, bool=true, c=a, str='str', iTransient='null', iStatic='name'}
   }
   ```

   

最后会在`/opt/xlp/`目录生成`user.out`文件，打开此文件，其十六进制内容如下：

```
00000000: aced 0005 7372 001c 636f 6d2e 7669 6f6c  ....sr..com.viol
00000010: 6574 2e73 6572 6961 6c69 7a61 626c 652e  et.serializable.
00000020: 5573 6572 0c7a 788a df64 3bfb 0200 0942  User.zx..d;....B
00000030: 0001 625a 0004 626f 6f6c 4300 0163 4400  ..bZ..boolC..cD.
00000040: 0164 4600 0166 4900 0169 4a00 016c 5300  .dF..fI..iJ..lS.
00000050: 0173 4c00 0373 7472 7400 124c 6a61 7661  .sL..strt..Ljava
00000060: 2f6c 616e 672f 5374 7269 6e67 3b78 7001  /lang/String;xp.
00000070: 0100 613f f199 9999 9999 9a40 0666 6600  ..a?.......@.ff.
00000080: 0000 0300 0000 0000 0000 0400 0274 0003  .............t..
00000090: 7374 720a                                str.
```

> 使用`vim`打开此文件，然后输入`:%!xxd`即可显示十六进制内容。

序列化二进制文件内容分析：

- `ACED`：stream_magic 声明使用了序列化协议。
- `0005`：序列化协议版本。
- `73`：TC_OBJECT 声明这是一个新的对象。
- `72`：TC_CLASSDESC 声明这里开始一个新的class。
- `001c`：class名字的长度为28。



序列化的类的描述