## [解决图片压缩出现Unsupported Image Type的问题](https://www.cnblogs.com/ncyhl/p/7681080.html)

在做图片压缩时，遇到如下错误：

```
Unsupported Image Type
```

原因是JDK自带的jpeg解析器不能解析所有jpeg格式的图片，如CMYK（印刷品模式）模式。

图片结果P图之后，保存时默认的保存格式是CMYK格式，而不是RGB格式。

CMYK是彩色印刷时采用的一种套色模式，利用色料的三原色混色原理，加上黑色油墨，共计四中颜色混合叠加，形成所谓的“全彩印刷”。四种标准颜色是：

- C：Cyan = 青色，又称为‘天蓝色’或’湛蓝’。
- M：Magenta = 品红色，又称‘洋红色’；
- Y：Yellow = 黄色；
- K：Key Plate（baack）= 定位套版色（黑色），有些文献解释说这里的K指代Black黑色，且为了避免与RGB的Blue蓝色混淆不用B而改称K。

解决方案：

使用**TwelveMonkeys**就可以解决了。

TwelveMonkeys的使用比较简单，只要把相关的jar包加入到类路径，其他的类我们基本不会用到，只要使用JDK的`javax.imageio.ImageIO`或其上层的接口就可以了。JDK的`javax.imageio.ImageIO`具有自动发现的功能，会自动查找相关的编解码类并使用，而不适用JDK默认的编解码类，所以使用这个库是完全无侵入的。

TwelveMonkeys GitHup：https://github.com/haraldk/TwelveMonkeys

Maven依赖：

```xml
<dependency>
    <groupId>com.twelvemonkeys.imageio</groupId>
    <artifactId>imageio-jpeg</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>com.twelvemonkeys.imageio</groupId>
    <artifactId>imageio-tiff</artifactId>
    <version>3.4.1</version>
</dependency>
```

测试代码：

```java
public static void main(String[] args) throws IOException {
	FileInputStream inputStream = 
        new FileInputStream("C:\\Users\\HP\\Desktop\\images\\Logo.jpg");
	BufferedImage read = ImageIO.read(inputStream);
	//压缩
	BufferedImage image = 
        Thumbnails.of(read).scale(1f).outputQuality(0.8f).asBufferedImage();
	//转换为输入流
	ByteArrayOutputStream bos = new ByteArrayOutputStream();
	ImageIO.write(image, "jpg", bos);
	ByteArrayInputStream byteArrayInputStream = 
        new ByteArrayInputStream(bos.toByteArray());
}
```

