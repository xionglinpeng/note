# Jekyll

official：https://www.jekyll.com.cn/

## 安装ruby开发环境

首先需要ruby开发环境，下载ruby安装包。

下载地址：https://rubyinstaller.org/downloads/

这里下载最新版的[Ruby+Devkit 2.7.1-1 (x64)](https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-2.7.1-1/rubyinstaller-devkit-2.7.1-1-x64.exe)。

> 注意，因为Ruby+Devkit 2.7.1-1 (x64）托管在git release上，是需要翻墙的，所以需要想办法下载下来。推荐使用IDM下载。



如下图所示，将Ruby+Devkit安装在D:\developer目录下（根据个人情况设置）。

> 注意：
>
> 1. 在设置安装目录栏的上方有一行英文提示
>
>    ```
>    please avoid any folder name that contains spaces (e.g. Program Files).
>    请避免任何包含空格的文件夹名（例如：Program Files）
>    ```
>
> 2. 勾选Add Ruby executables to your PATH。
>
>    勾选了这个一个选项的话，在成功安装Ruby+Devkit之后会自动设置Ruby的环境变量，这样直接打开cmd命令行窗口，就可以执行了。如果没有勾选也没有关系，可以后续手动配置环境变量，或者使用Ruby命令行的cmd窗口：Start Command Prompt with Ruby（开始-菜单 可以看到）。

![1587818645830](images\1587818645830.png)

上一步的文件夹标识后面有如下说明：

```
setup will install Ruby with MSYS2 into the following folder.click install to continue or click browse to use adifferent one.
安装程序将使用MSYS2将Ruby安装到以下文件夹中。单击install以继续安装，或单击browse以使用不同的安装。
```

Ruby+Devkit相关环境时依赖MSYS2的，所以默认勾选了`MSYS2 development toolchain 2020-04-02`。如果这里不勾选，后续也需要手动安装MSYS2。

直接Next > 

![1587818652904](images\1587818652904.png)

等待提取文件安装完毕。

![1587818793409](images\1587818793409.png)

Finish。

![1587818940178](images\1587818940178.png)

安装完成会后会弹出如下命令行界面，有三个选项：

1. MSYS2基础安装。

   安装时已经勾选了同步安装MSYS2，所以这里选择1是没有意义的，如果非要选择1进行安装，将会如示例*Demonstration.1*所示。

2. MSYS2系统更新（可选）。

   [MSYS2](https://baike.baidu.com/item/MSYS2/17190550?fr=aladdin)是在window上的模拟Linux环境，其包管理工具是pacman，这里的系统更新其实就是pacman的软件源更新。

3. MSYS2和MINGW开发工具链。

这里不做任何选择，直接关闭即可。

![1587827052878](images\1587827052878.png)



*Demonstration.1*:

```shell
 _____       _           _____           _        _ _         ___
|  __ \     | |         |_   _|         | |      | | |       |__ \
| |__) |   _| |__  _   _  | |  _ __  ___| |_ __ _| | | ___ _ __ ) |
|  _  / | | | '_ \| | | | | | | '_ \/ __| __/ _` | | |/ _ \ '__/ /
| | \ \ |_| | |_) | |_| |_| |_| | | \__ \ || (_| | | |  __/ | / /_
|_|  \_\__,_|_.__/ \__, |_____|_| |_|___/\__\__,_|_|_|\___|_||____|
                    __/ |           _
                   |___/          _|_ _  __   | | o __  _| _     _
                                   | (_) |    |^| | | |(_|(_)\^/_>

   1 - MSYS2 base installation
   2 - MSYS2 system update (optional)
   3 - MSYS2 and MINGW development toolchain

Which components shall be installed? If unsure press ENTER [1,3] 1

> sh -lc true
MSYS2 seems to be properly installed

   1 - MSYS2 base installation
   2 - MSYS2 system update (optional)
   3 - MSYS2 and MINGW development toolchain

Which components shall be installed? If unsure press ENTER []
```



> 命令行界面被关闭之后，如果想执行2或者3怎么办，可以打开cmd窗口执行命令`ridk install`开启选择。

## 测试ruby和gem

到目前为止，Ruby环境已经安装完成，因为Ruby+Devkit，所以gem也同时被安装了。

打开“开始”菜单，选择Start Command Prompt with Ruby，开发Ruby命令行窗口，指定如下命令测试安装接口：

```shell
C:\Users\xlp>ruby -v
ruby 2.7.1p83 (2020-03-31 revision a0c7c23c9c) [x64-mingw32]

C:\Users\xlp>gem -v
3.1.2
```

## 配置gem源

后续需要通过gem安装jekyll和bundler，需要一些镜像源上的资源。

配置gem的源为国内的源，至于为什么需要配置，想必不用多说了。

如果使用的默认源也能下载，只是那速度非常感人。

```shell
# 查看当前源
C:\Users\xlp>gem sources -l
*** CURRENT SOURCES ***

https://rubygems.org/
# 删除默认源
C:\Users\xlp>gem sources -r https://rubygems.org/
https://rubygems.org/ removed from sources
# 添加国内源
C:\Users\xlp>gem sources -a https://gems.ruby-china.com
https://gems.ruby-china.com added to sources
# 再次查看当前源
C:\Users\xlp>gem source -l
*** CURRENT SOURCES ***

https://gems.ruby-china.com
```

如果删除旧源的时候提示如下错误：

```shell
C:\Users\xlp>gem sources -a https://rubygems.org/
https://rubygems.org/ is too similar to https://rubygems.org

Do you want to add this source? [yn]  y
source https://rubygems.org/ already present in the cache
```

执行命令``清除缓存即可

```shell
C:\Users\xlp>gem sources -c
*** Removed specs cache ***
```

## 配置bundler源





## 安装jekyll和bundler

执行命令`gem install jekyll bundler`。

```shell
D:\developer\Ruby27-x64\bin>gem install jekyll bundler
Fetching public_suffix-4.0.4.gem
Fetching addressable-2.7.0.gem
Fetching colorator-1.1.0.gem
Fetching http_parser.rb-0.6.0.gem
Fetching eventmachine-1.2.7-x64-mingw32.gem
Fetching concurrent-ruby-1.1.6.gem
Fetching ffi-1.12.2-x64-mingw32.gem
Fetching sassc-2.3.0-x64-mingw32.gem
Fetching jekyll-sass-converter-2.1.0.gem
Fetching rb-fsevent-0.10.3.gem
Fetching rb-inotify-0.10.1.gem
Fetching listen-3.2.1.gem
Fetching jekyll-watch-2.2.1.gem
Fetching kramdown-2.2.1.gem
Fetching kramdown-parser-gfm-1.1.0.gem
Fetching liquid-4.0.3.gem
Fetching mercenary-0.3.6.gem
Fetching forwardable-extended-2.6.0.gem
Fetching pathutil-0.16.2.gem
Fetching rouge-3.18.0.gem
Fetching jekyll-4.0.0.gem
Fetching safe_yaml-1.0.5.gem
Fetching unicode-display_width-1.7.0.gem
Fetching terminal-table-1.8.0.gem
Successfully installed public_suffix-4.0.4
Successfully installed addressable-2.7.0
Successfully installed colorator-1.1.0
Temporarily enhancing PATH for MSYS/MINGW...
Building native extensions. This could take a while...
Successfully installed http_parser.rb-0.6.0
Successfully installed eventmachine-1.2.7-x64-mingw32
Successfully installed em-websocket-0.5.1
Successfully installed concurrent-ruby-1.1.6

HEADS UP! i18n 1.1 changed fallbacks to exclude default locale.
But that may break your application.

If you are upgrading your Rails application from an older version of Rails:

Please check your Rails app for 'config.i18n.fallbacks = true'.
If you're using I18n (>= 1.1.0) and Rails (< 5.2.2), this should be
'config.i18n.fallbacks = [I18n.default_locale]'.
If not, fallbacks will be broken in your app by I18n 1.1.x.

If you are starting a NEW Rails application, you can ignore this notice.

For more info see:
https://github.com/svenfuchs/i18n/releases/tag/v1.1.0

Successfully installed i18n-1.8.2
Successfully installed ffi-1.12.2-x64-mingw32
Successfully installed sassc-2.3.0-x64-mingw32
Successfully installed jekyll-sass-converter-2.1.0
Successfully installed rb-fsevent-0.10.3
Successfully installed rb-inotify-0.10.1
Successfully installed listen-3.2.1
Successfully installed jekyll-watch-2.2.1
Successfully installed kramdown-2.2.1
Successfully installed kramdown-parser-gfm-1.1.0
Successfully installed liquid-4.0.3
Successfully installed mercenary-0.3.6
Successfully installed forwardable-extended-2.6.0
Successfully installed pathutil-0.16.2
Successfully installed rouge-3.18.0
Successfully installed safe_yaml-1.0.5
Successfully installed unicode-display_width-1.7.0
Successfully installed terminal-table-1.8.0
-------------------------------------------------------------------------------------
Jekyll 4.0 comes with some major changes, notably:

  * Our `link` tag now comes with the `relative_url` filter incorporated into it.
    You should no longer prepend `{{ site.baseurl }}` to `{% link foo.md %}`
    For further details: https://github.com/jekyll/jekyll/pull/6727

  * Our `post_url` tag now comes with the `relative_url` filter incorporated into it.
    You shouldn't prepend `{{ site.baseurl }}` to `{% post_url 2019-03-27-hello %}`
    For further details: https://github.com/jekyll/jekyll/pull/7589

  * Support for deprecated configuration options has been removed. We will no longer
    output a warning and gracefully assign their values to the newer counterparts
    internally.
-------------------------------------------------------------------------------------
Successfully installed jekyll-4.0.0
Parsing documentation for public_suffix-4.0.4
Installing ri documentation for public_suffix-4.0.4
Parsing documentation for addressable-2.7.0
Installing ri documentation for addressable-2.7.0
Parsing documentation for colorator-1.1.0
Installing ri documentation for colorator-1.1.0
Parsing documentation for http_parser.rb-0.6.0
unknown encoding name "chunked\r\n\r\n25" for ext/ruby_http_parser/vendor/http-parser-java/tools/parse_tests.rb, skipping
Installing ri documentation for http_parser.rb-0.6.0
Parsing documentation for eventmachine-1.2.7-x64-mingw32
Installing ri documentation for eventmachine-1.2.7-x64-mingw32
Parsing documentation for em-websocket-0.5.1
Installing ri documentation for em-websocket-0.5.1
Parsing documentation for concurrent-ruby-1.1.6
Installing ri documentation for concurrent-ruby-1.1.6
Parsing documentation for i18n-1.8.2
Installing ri documentation for i18n-1.8.2
Parsing documentation for ffi-1.12.2-x64-mingw32
Installing ri documentation for ffi-1.12.2-x64-mingw32
Parsing documentation for sassc-2.3.0-x64-mingw32
Installing ri documentation for sassc-2.3.0-x64-mingw32
Parsing documentation for jekyll-sass-converter-2.1.0
Installing ri documentation for jekyll-sass-converter-2.1.0
Parsing documentation for rb-fsevent-0.10.3
Installing ri documentation for rb-fsevent-0.10.3
Parsing documentation for rb-inotify-0.10.1
Installing ri documentation for rb-inotify-0.10.1
Parsing documentation for listen-3.2.1
Installing ri documentation for listen-3.2.1
Parsing documentation for jekyll-watch-2.2.1
Installing ri documentation for jekyll-watch-2.2.1
Parsing documentation for kramdown-2.2.1
Installing ri documentation for kramdown-2.2.1
Parsing documentation for kramdown-parser-gfm-1.1.0
Installing ri documentation for kramdown-parser-gfm-1.1.0
Parsing documentation for liquid-4.0.3
Installing ri documentation for liquid-4.0.3
Parsing documentation for mercenary-0.3.6
Installing ri documentation for mercenary-0.3.6
Parsing documentation for forwardable-extended-2.6.0
Installing ri documentation for forwardable-extended-2.6.0
Parsing documentation for pathutil-0.16.2
Installing ri documentation for pathutil-0.16.2
Parsing documentation for rouge-3.18.0
Installing ri documentation for rouge-3.18.0
Parsing documentation for safe_yaml-1.0.5
Installing ri documentation for safe_yaml-1.0.5
Parsing documentation for unicode-display_width-1.7.0
Installing ri documentation for unicode-display_width-1.7.0
Parsing documentation for terminal-table-1.8.0
Installing ri documentation for terminal-table-1.8.0
Parsing documentation for jekyll-4.0.0
Installing ri documentation for jekyll-4.0.0
Done installing documentation for public_suffix, addressable, colorator, http_parser.rb, eventmachine, em-websocket, concurrent-ruby, i18n, ffi, sassc, jekyll-sass-converter, rb-fsevent, rb-inotify, listen, jekyll-watch, kramdown, kramdown-parser-gfm, liquid, mercenary, forwardable-extended, pathutil, rouge, safe_yaml, unicode-display_width, terminal-table, jekyll after 65 seconds
Fetching bundler-2.1.4.gem
Successfully installed bundler-2.1.4
Parsing documentation for bundler-2.1.4
Installing ri documentation for bundler-2.1.4
Done installing documentation for bundler after 18 seconds
27 gems installed
```



## 测试jekyll和bundler

```shell
C:\Users\xlp\Desktop>jekyll -v
jekyll 4.0.0

C:\Users\xlp>bundle -v
Bundler version 2.1.4
```



## 搭建博客

1. 在 `./myblog` 目录下创建一个全新的 Jekyll 网站。

   ```shell
   C:\Users\xlp\Desktop>jekyll new myblog
   Running bundle install in C:/Users/xlp/Desktop/myblog...
     Bundler: Fetching gem metadata from https://rubygems.org/............
     Bundler: Fetching gem metadata from https://rubygems.org/..
     Bundler: Resolving dependencies...
     Bundler: Using public_suffix 4.0.4
     Bundler: Using addressable 2.7.0
     Bundler: Using bundler 2.1.4
     Bundler: Using colorator 1.1.0
     Bundler: Using concurrent-ruby 1.1.6
     Bundler: Using eventmachine 1.2.7 (x64-mingw32)
     Bundler: Using http_parser.rb 0.6.0
     Bundler: Using em-websocket 0.5.1
     Bundler: Using ffi 1.12.2 (x64-mingw32)
     Bundler: Using forwardable-extended 2.6.0
     Bundler: Using i18n 1.8.2
     Bundler: Using sassc 2.3.0 (x64-mingw32)
     Bundler: Using jekyll-sass-converter 2.1.0
     Bundler: Using rb-fsevent 0.10.3
     Bundler: Using rb-inotify 0.10.1
     Bundler: Using listen 3.2.1
     Bundler: Using jekyll-watch 2.2.1
     Bundler: Using rexml 3.2.4
     Bundler: Using kramdown 2.2.1
     Bundler: Using kramdown-parser-gfm 1.1.0
     Bundler: Using liquid 4.0.3
     Bundler: Using mercenary 0.3.6
     Bundler: Using pathutil 0.16.2
     Bundler: Using rouge 3.18.0
     Bundler: Using safe_yaml 1.0.5
     Bundler: Using unicode-display_width 1.7.0
     Bundler: Using terminal-table 1.8.0
     Bundler: Using jekyll 4.0.0
     Bundler: Using jekyll-feed 0.13.0
     Bundler: Using jekyll-seo-tag 2.6.1
     Bundler: Using minima 2.5.1
     Bundler: Using thread_safe 0.3.6
     Bundler: Fetching tzinfo 1.2.7
     Bundler: Installing tzinfo 1.2.7
     Bundler: Fetching tzinfo-data 1.2020.1
     Bundler: Installing tzinfo-data 1.2020.1
     Bundler: Fetching wdm 0.1.1
     Bundler: Installing wdm 0.1.1 with native extensions
     Bundler: Bundle complete! 6 Gemfile dependencies, 35 gems now installed.
     Bundler: Use `bundle info [gemname]` to see where a bundled gem is installed.
   New jekyll site installed in C:/Users/xlp/Desktop/myblog.
   ```

2. 进入新创建的目录。

   ```
   cd myblog
   ```

3. 构建网站并启动一个本地服务器。

   ```shell
   C:\Users\xlp\Desktop\myblog>bundle exec jekyll serve
   Configuration file: C:/Users/xlp/Desktop/myblog/_config.yml
               Source: C:/Users/xlp/Desktop/myblog
          Destination: C:/Users/xlp/Desktop/myblog/_site
    Incremental build: disabled. Enable with --incremental
         Generating...
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/document.rb:466: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/document.rb:432: warning: Passing the keyword argument as the last hash parameter is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/document.rb:75: warning: The called method `merge_data!' is defined here
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/document.rb:439: warning: Passing the keyword argument as the last hash parameter is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/document.rb:75: warning: The called method `merge_data!' is defined here
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
          Jekyll Feed: Generating feed for posts
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
   D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
                       done in 1.127 seconds.
    Auto-regeneration: enabled for 'C:/Users/xlp/Desktop/myblog'
       Server address: http://127.0.0.1:4000/
     Server running... press ctrl-c to stop.
   [2020-04-25 22:32:41] ERROR `/favicon.ico' not found.
   [2020-04-25 22:36:09] ERROR `/favicon.ico' not found.
   ```

4. 在浏览器中打开 [http://localhost:4000](http://localhost:4000/) 网址。

   ![1587828417338](images\1587828417338.png)