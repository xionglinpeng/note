# minimal-mistakes

Official：https://mmistakes.github.io/minimal-mistakes/

GitHup：https://github.com/mmistakes/minimal-mistakes

Jekyll主题，用于构建个人站点，博客，项目文档或投资组合。



## 快速入门

克隆minimal-mistakes

```shell
git clone https://github.com/mmistakes/minimal-mistakes.git
```

将以下内容添加到Gemfile

```shell
gem "minimal-mistakes-jekyll"
```

进入到目录minimal-mistakes，运行以下[Bundler](http://bundler.io/)命令拉取和更新bundled gems。

```shell
bundle
```

在你的Jekyll项目的`_config.yml`文件中设置`theme` 。

```shell
theme: minimal-mistakes-jekyll
```

启动博客

```shell
bundle exec jekyll serve
```

在浏览器中打开 [http://localhost:4000](http://localhost:4000/) 网址访问。



## minimal-mistakes-jekyll配置







1. 设置皮肤
2. 设置语言
3. 打开搜索
4. 



## **CUSTOMIZATION**



### Overriding Theme Defaults



### Navigation



### UI Text



### Authors





# 未完成资源

额外的功能：https://www.jianshu.com/p/d19eede28520

额外的功能-例如为文章统计：https://www.jianshu.com/p/9f71e260925d

markdown解析器问题：https://www.jianshu.com/p/558a5d50e077



完美的入门教程：https://blog.csdn.net/zy_281870667/article/details/78907214

Linux安装Jekyll：https://blog.csdn.net/yq_forever/article/details/104148454

修改Gem,Bundler的镜像为国内源：https://www.jianshu.com/p/a56aa38a6403



```shell
D:\program\minimal-mistakes>bundle
Fetching gem metadata from https://gems.ruby-china.com/...........
Fetching gem metadata from https://gems.ruby-china.com/.
Resolving dependencies...
Using public_suffix 4.0.4
Using addressable 2.7.0
Using bundler 2.1.4
Using colorator 1.1.0
Using concurrent-ruby 1.1.6
Using eventmachine 1.2.7 (x64-mingw32)
Using http_parser.rb 0.6.0
Using em-websocket 0.5.1
Fetching multipart-post 2.1.1
Installing multipart-post 2.1.1
Fetching faraday 1.0.1
Installing faraday 1.0.1
Using ffi 1.12.2 (x64-mingw32)
Using forwardable-extended 2.6.0
Using i18n 1.8.2
Using sassc 2.3.0 (x64-mingw32)
Using jekyll-sass-converter 2.1.0
Using rb-fsevent 0.10.3
Using rb-inotify 0.10.1
Using listen 3.2.1
Using jekyll-watch 2.2.1
Using rexml 3.2.4
Using kramdown 2.2.1
Using kramdown-parser-gfm 1.1.0
Using liquid 4.0.3
Using mercenary 0.3.6
Using pathutil 0.16.2
Using rouge 3.18.0
Using safe_yaml 1.0.5
Using unicode-display_width 1.7.0
Using terminal-table 1.8.0
Using jekyll 4.0.0
Using jekyll-feed 0.13.0
Fetching sawyer 0.8.2
Installing sawyer 0.8.2
Fetching octokit 4.18.0
Installing octokit 4.18.0
Fetching jekyll-gist 1.5.0
Installing jekyll-gist 1.5.0
Fetching jekyll-include-cache 0.2.0
Installing jekyll-include-cache 0.2.0
Fetching jekyll-paginate 1.1.0
Installing jekyll-paginate 1.1.0
Fetching jekyll-sitemap 1.4.0
Installing jekyll-sitemap 1.4.0
Fetching minimal-mistakes-jekyll 4.19.1
Installing minimal-mistakes-jekyll 4.19.1
Bundle complete! 1 Gemfile dependency, 38 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```









```shell
D:\program\minimal-mistakes>bundle exec jekyll serve
Configuration file: D:/program/minimal-mistakes/_config.yml
            Source: D:/program/minimal-mistakes
       Destination: D:/program/minimal-mistakes/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/convertible.rb:41: warning: Using the last argument as keyword parameters is deprecated
       Jekyll Feed: Generating feed for posts
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
D:/developer/Ruby27-x64/lib/ruby/gems/2.7.0/gems/jekyll-4.0.0/lib/jekyll/tags/include.rb:179: warning: Using the last argument as keyword parameters is deprecated
                    done in 1.829 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for 'D:/program/minimal-mistakes'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
[2020-04-26 11:30:59] ERROR `/favicon.ico' not found.
```

