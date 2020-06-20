# JAVE

## resources

[jave document](http://www.sauronsoftware.it/projects/jave/manual.php)

[ffmpeg document](http://ffmpeg.org/)

## Installation and requirements

为了在Java应用程序中使用JAVE，必须在应用程序CLASSPATH中添加文件*jave-1.0.jar*。

JAVE在Java运行时环境J2SE v.1.4或更高版本上运行。

## Audio/video encoding

最重要的JAVE类是`it.sauronsoftware.jave.Encoder`对象，*Encoder*对象公开了许多用于多媒体代码转换的方法。为了使用JAVE，你总是必须创建一个*Encoder*实例：

```java
Encoder encoder = new Encoder();
```

一旦创建了实例，就可以开始调用encode()方法进行代码转换：

```java
public void encode(java.io.File source,
                   java.io.File target,
                   it.sauronsoftware.jave.EncodingAttributes attributes)
            throws java.lang.IllegalArgumentException,
                   it.sauronsoftware.jave.InputFormatException,
                   it.sauronsoftware.jave.EncoderException
```

第一个参数`source`表示要解码的源文件。

第二个参数`target`是要创建和编码的目标文件。

属性参数的类型为`it.sauronsoftware.jave.EncodingAttributes`，是一个包含编码器所需信息的数据结构。

请注意，对`encode()`的调用是一个阻塞调用：只有在代码转换操作完成(或失败)后，该方法才会返回。如果您对监视转码操作感兴趣，请参阅“监视转码操作”一节。

## Encoding attributes

要指定关于代码转换操作的首选项，必须向`encode()`调用提供`it.sauronsoftware.jave.EncodingAttributes`实例。您可以创建自己的`EncodingAttributes`实例，并且可以用以下方法填充它：

- `public void setAudioAttributes(it.sauronsoftware.jave.AudioAttributes audioAttributes)`

  它设置音频编码属性。如果从未调用新的EncodingAttributes实例，或者如果给定参数为空，则编码文件中不包含音频流。请参考 "[Audio encoding attributes](http://www.sauronsoftware.it/projects/jave/manual.php#3.1)"。

- `public void setVideoAttributes(it.sauronsoftware.jave.AudioAttributes videoAttributes)`

  它设置视频编码属性。如果从未调用新的EncodingAttributes实例，或者，如果给定参数为空，则编码文件中不包含视频流。请参考 "[Video encoding attributes](http://www.sauronsoftware.it/projects/jave/manual.php#3.2)"。

- `public void setFormat(java.lang.String format)`

  它用于设置新编码文件的streams容器的格式。给定的参数表示格式名称。只有当编码格式名称出现在正在使用的编码器实例的`getSupportedEncodingFormats()`方法返回的列表中时，它才有效并受支持。

- `public void setOffset(java.lang.Float offset)`

  它为转码操作设置偏移量。源文件将从开始时的偏移秒开始重新编码。例如，如果你想剪掉源文件的前五秒，您应该在传递给编码器的`EncodingAttributes`对象上调用`setOffset(5)`。

- `public void setDuration(java.lang.Float duration)`

  它设置了代码转换操作的持续时间。在目标文件中只会重新编码源的持续时间秒。例如，如果你想从源代码中提取并转换30秒的部分代码，您应该在传递给编码器的`EncodingAttributes`对象上调用`setDuration(30)`。

### Audio encoding attributes



### Video encoding attributes

视频编码属性由` it.sauronsoftware.jave.VideoAttributes`类的实例表示。这类对象的可用方法有:

- `public void setCodec(java.lang.String codec)`

  它用于设置视频流转码的编解码器的名称。您必须从当前编码器实例的getVideoEncoders()方法返回的列表中选择一个值。否则你可以传递`VideoAttributes.DIRECT_STREAM_COPY`的特殊值，这需要源文件原始视频流的副本。

- `public void setTag(java.lang.String tag)`

  它设置与重新编码的视频流相关联的tag/fourcc值。如果没有设置任何值，编码器将选择一个默认值。标签值经常被多媒体播放器用来选择在流上运行的视频解码器。例如，带有“DIVX”标签值的mpeg4视频流将使用播放器使用的默认DIVX解码器进行解码。而且，顺便说一下，这就是DivX的真正含义:mpeg4视频流，这正是DivX的真正含义:mpeg4视频流附带“DivX”tag/fourcc的值!

- `public void setBitRate(java.lang.Integer bitRate)`

  它为新的重新编码的视频流设置比特率值。如果没有设置比特率值，编码器将选择一个默认的。这个值应该以比特每秒表示。例如，如果你想要一个360 kb/s的比特率，你应该调用`setBitRate(new Integer(360000))`。

- `public void setFrameRate(java.lang.Integer bitRate)`

  它为新的重新编码的音频流设置帧速率值。如果没有设置比特率帧率，编码器将选择一个默认帧率。该值应该以每秒帧数表示。例如，如果你想要30 f/s帧速率，你应该调用`setFrameRate(new Integer(30))`。

- `public void setSize(it.sauronsoftware.jave.VideoSize size)`

  它设置了视频流中图像的大小和比例。如果没有设置值，编码器将保留原来的大小和比例。否则你可以传递一个`it.sauronsoftware.java.VideoSize`实例，设置喜欢的尺寸。可以使用像素值设置新编码视频的宽度和高度，缩放原始视频。例如，如果你想缩放视频到512px的宽度和384px的高度，你应该调用`setSize(new VideoSize(512, 384))`。

## Monitoring the transcoding operation

监视代码转换操作

您可以使用侦听器监视代码转换操作。JAVE定义了`it.sauronsoftware.jave.EncoderProgressListener`接口。这个接口可以由应用程序实现，具体的`EncoderProgressListener`实例可以传递给编码器。每当发生重大事件时，编码器将调用侦听器方法。要将`EncoderProgressListener`传递给编码器，您应该使用`encode()`方法的定义:

```java
public void encode(java.io.File source,
                   java.io.File target,
                   it.sauronsoftware.jave.EncodingAttributes attributes,
                   it.sauronsoftware.jave.EncoderProgressListener listener)
            throws java.lang.IllegalArgumentException,
                   it.sauronsoftware.jave.InputFormatException,
                   it.sauronsoftware.jave.EncoderException
```

要实现EncoderProgressListener接口，您必须定义以下所有方法:

- `public void sourceInfo(it.sauronsoftware.jave.MultimediaInfo info)`

  在分析源文件之后，编码器调用这个方法。info参数是`it.sauronsoftware.jave.MultimediaInfo`类的一个实例，它表示关于源音频和视频流及其容器的信息。

- `public void progress(int permil)`

  每当编码操作取得进展时，编码器就会调用此方法。permil参数是表示当前操作到达的点的值，其范围从0(刚开始的操作)到1000(完成的操作)。

- `public void message(java.lang.String message)`

  编码器调用此方法来通知有关转码操作的消息(通常消息是警告)。

## Transcoding failures

转码失败

## 获取有关多媒体文件的信息

在对现有的多媒体文件进行编码之前，调用编码器`getInfo()`方法。`getInfo()`方法向您提供关于文件使用的容器以及其封装的音频和视频流的信息:

```java
public it.sauronsoftware.jave.MultimediaInfo getInfo(java.io.File source) throws it.sauronsoftware.jave.InputFormatException,it.sauronsoftware.jave.EncoderException
```

`it.sauronsoftware.jave.MultimediaInfo`对象封装了关于整个多媒体内容及其流的信息，使用`it.sauronsoftware.jave.AudioInfo`和`it.sauronsoftware.jave.VideoInfo`的实例来描述包装的音频和视频。这些对象类似于`EncodingAttributes`, `AudioAttributes`和`VideoAttributes`，但它们在只读模式下工作。查看JAVE API javadoc文档，与JAVE发行版绑定，以获得更多关于它们的详细信息。

## 使用一个备选的ffmpeg可执行文件




## 支持容器格式



## Examples

从一个通用的AVI到一个youtube样的FLV电影，嵌入MP3音频流:

```java
File source = new File("source.avi");
File target = new File("target.flv");
AudioAttributes audio = new AudioAttributes();
audio.setCodec("libmp3lame");
audio.setBitRate(new Integer(64000));
audio.setChannels(new Integer(1));
audio.setSamplingRate(new Integer(22050));
VideoAttributes video = new VideoAttributes();
video.setCodec("flv");
video.setBitRate(new Integer(160000));
video.setFrameRate(new Integer(15));
video.setSize(new VideoSize(400, 300));
EncodingAttributes attrs = new EncodingAttributes();
attrs.setFormat("flv");
attrs.setAudioAttributes(audio);
attrs.setVideoAttributes(video);
Encoder encoder = new Encoder();
encoder.encode(source, target, attrs);
```

接下来的几行从AVI提取音频信息并存储在一个普通的WAV文件中:

```java
File source = new File("source.avi");
File target = new File("target.wav");
AudioAttributes audio = new AudioAttributes();
audio.setCodec("pcm_s16le");
EncodingAttributes attrs = new EncodingAttributes();
attrs.setFormat("wav");
attrs.setAudioAttributes(audio);
Encoder encoder = new Encoder();
encoder.encode(source, target, attrs);
```

下一个例子是一个音频WAV文件，生成一个128 kbit/s，立体声，44100 Hz MP3文件:

```java
File source = new File("source.wav");
File target = new File("target.mp3");
AudioAttributes audio = new AudioAttributes();
audio.setCodec("libmp3lame");
audio.setBitRate(new Integer(128000));
audio.setChannels(new Integer(2));
audio.setSamplingRate(new Integer(44100));
EncodingAttributes attrs = new EncodingAttributes();
attrs.setFormat("mp3");
attrs.setAudioAttributes(audio);
Encoder encoder = new Encoder();
encoder.encode(source, target, attrs);
```

下一个解码一个通用的AVI文件，并创建另一个相同的视频流的源和重新编码的低质量MP3音频流:

```java
File source = new File("source.avi");
File target = new File("target.avi");
AudioAttributes audio = new AudioAttributes();
audio.setCodec("libmp3lame");
audio.setBitRate(new Integer(56000));
audio.setChannels(new Integer(1));
audio.setSamplingRate(new Integer(22050));
VideoAttributes video = new VideoAttributes();
video.setCodec(VideoAttributes.DIRECT_STREAM_COPY);
EncodingAttributes attrs = new EncodingAttributes();
attrs.setFormat("avi");
attrs.setAudioAttributes(audio);
attrs.setVideoAttributes(video);
Encoder encoder = new Encoder();
encoder.encode(source, target, attrs);
```

下一个生成的AVI与mpeg4 /DivX视频和OGG Vorbis音频:

```java
File source = new File("source.avi");
File target = new File("target.avi");
AudioAttributes audio = new AudioAttributes();
audio.setCodec("libvorbis");
VideoAttributes video = new VideoAttributes();
video.setCodec("mpeg4");
video.setTag("DIVX");
video.setBitRate(new Integer(160000));
video.setFrameRate(new Integer(30));
EncodingAttributes attrs = new EncodingAttributes();
attrs.setFormat("mpegvideo");
attrs.setAudioAttributes(audio);
attrs.setVideoAttributes(video);
Encoder encoder = new Encoder();
encoder.encode(source, target, attrs);
```

适合智能手机视频:

```java
File source = new File("source.avi");
File target = new File("target.3gp");
AudioAttributes audio = new AudioAttributes();
audio.setCodec("libfaac");
audio.setBitRate(new Integer(128000));
audio.setSamplingRate(new Integer(44100));
audio.setChannels(new Integer(2));
VideoAttributes video = new VideoAttributes();
video.setCodec("mpeg4");
video.setBitRate(new Integer(160000));
video.setFrameRate(new Integer(15));
video.setSize(new VideoSize(176, 144));
EncodingAttributes attrs = new EncodingAttributes();
attrs.setFormat("3gp");
attrs.setAudioAttributes(audio);
attrs.setVideoAttributes(video);
Encoder encoder = new Encoder();
encoder.encode(source, target, attrs);
```





















