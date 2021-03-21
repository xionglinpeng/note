# Byte→Hex

byte类型二进制数据与十六进制相互转换。

## 1、格式化：%2X和%2x

 可以使用转换符进行格式化输出，`%2X`输出大写十六进制，`%2x`输出小写十六进制。

```java
String.format("%2X",byte);
System.out.printf("%2X",byte);
```

## 2、Integer.toHexString(int)

Integer对象提供了toHexString方法可以将一个int转换为16进制字符串。

```java
Integer.toHexString(10)
```

## 3、Apache Commons编解码器

Apache Commons提供了编解码器，其中就包含了十六进制转换的操作。

添加依赖

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>${version}</version>
</dependency>
```

类`org.apache.commons.codec.binary.Hex`提供了用于char、byte和String相互转换的静态函数：

```java
byte[] decodeHex(String data);
byte[] decodeHex(char[] data);
char[] encodeHex(byte[] data);
char[] encodeHex(byte[] data, boolean toLowerCase);
char[] encodeHex(ByteBuffer data);
char[] encodeHex(ByteBuffer data, boolean toLowerCase);
String encodeHexString(byte[] data);
String encodeHexString(byte[] data, boolean toLowerCase);
String encodeHexString(ByteBuffer data);
String encodeHexString(ByteBuffer data, boolean toLowerCase);
```

*example：*

```java
var hex = Hex.encodeHexString(new byte[]{4,5,6,10,11});
System.out.println(hex); //0405060a0b
```

## 4、Spring Security crypto

在Spring Security框架中提供了对十六进制与char和byte进行转换的工具类，有两个方式可以提供：

1. 提供Spring Security核心包

   ```xml
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-core</artifactId>
       <version>5.3.2.RELEASE</version>
   </dependency>
   ```

2. 提供Spring Security加密包

   ```xml
   <dependency>
       <groupId>org.springframework.security</groupId>
       <artifactId>spring-security-crypto</artifactId>
       <version>5.3.2.RELEASE</version>
   </dependency>
   ```

类`org.springframework.security.crypto.codec.Hex`是用于十六进制转换的，位于核心包和加密包中，也就是说上述两个包中都有`Hex`类型，它们的代码是一致，引入任意一个都可以。

提供了两个用于相互转换的静态函数：

```java
char[] encode(byte[] bytes);
byte[] decode(CharSequence s);
```

*example：*

```java
var hex = Hex.encode(new byte[]{4,5,6,10,11});
System.out.println(hex); //0405060a0b
```

## 5、位移计算

二进制转十六进制的规则是每4个比特位之和为一个十六进制位，例如二进制11100011，前四个比特位之和为3，后四个比特位为E，组合起来即十六进制0xE3。

```
1110,0011
  ↓   ↓
  E   3
```

因此，只需要分别取出高四位和低四位，并转换成功int，然后从声明的十六进制字符数组中去取对应的字符组合起来即可。

```java
//十六进制字符数组
private static final char[] HEX = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
			'a', 'b', 'c', 'd', 'e', 'f' };
//b是一个byte型数据
char[] result = new char[2];
//取高四位
result[0] = HEX[(0xF0 & b) >>> 4];
//取低四位
result[1] = HEX[(0xF0 & b)];
```

十六进制转二进制byte也是一样的原理，先取十六进制字符，获得对应十进制索引，即二进制4位之和，在通过位移运算组合起来即可。





