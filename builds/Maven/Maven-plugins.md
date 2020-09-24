



## docker-maven-plugin

```xml
<!--使用docker-maven-plugin插件-->
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.0.0</version>

    <!--将插件绑定在某个phase执行-->
    <executions>
        <execution>
            <id>build-image</id>
            <!--将插件绑定在package这个phase上。也就是说，
                用户只需执行mvn package ，就会自动执行mvn docker:build-->
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>

    <configuration>
        <!--指定生成的镜像名,这里是我们的项目名-->
        <imageName>${project.artifactId}</imageName>
        <!--指定标签 这里指定的是镜像的版本，我们默认版本是latest-->
        <imageTags>
            <imageTag>latest</imageTag>
        </imageTags>
        <!-- 指定我们项目中Dockerfile文件的路径-->
        <dockerDirectory>${project.basedir}/src/main/resources</dockerDirectory>

        <!--指定远程docker 地址-->
        <dockerHost>http://127.0.0.1:2375</dockerHost>

        <!-- 这里是复制 jar 包到 docker 容器指定目录配置 -->
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <!--jar包所在的路径  此处配置的即对应项目中target目录-->
                <directory>${project.build.directory}</directory>
                <!-- 需要包含的 jar包 ，这里对应的是 Dockerfile中添加的文件名　-->
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```