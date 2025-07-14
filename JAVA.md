# JAVA

## 一、graalvm打包原生app

### 1.maven中引入相关依赖，以及打包时配置

```xml
<dependencies>
        <dependency>
            <groupId>info.picocli</groupId>
            <artifactId>picocli</artifactId>
            <version>4.7.5</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <!-- 打包成可执行 JAR -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.10.1</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.4.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <!-- 配置main函数路径-->
                                    <mainClass>com.sj.CliDemo</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

### 2.编写代码

```java
package com.sj;

import picocli.CommandLine;
import picocli.CommandLine.Command;

import java.io.*;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;

@Command(
        name = "ag",
        mixinStandardHelpOptions = true,
        version = "mytool 1.0",
        description = "这是一个 Java 编写的 CLI 工具示例"
)
public class CliDemo implements Runnable {

    @Override
    public void run() {
        //指定输入的字符集
        Scanner scanner = new Scanner(System.in, StandardCharsets.UTF_8);
        while (true) {
            System.out.print("eg:");
            String userInput = scanner.nextLine();
            if ("exit".equals(userInput)) {
                break;
            }
            else if ("who".equals(userInput)) {
                System.out.println("我是一个AI助手，旨在帮助你解决问题。");
                continue;
            }
        }
    }

    public static void main(String[] args) {
        //指定输出的字符集
        System.setOut(new PrintStream(System.out, true, StandardCharsets.UTF_8));
        new CommandLine(new CliDemo()).execute(args);
    }
}

```

### 3.打出jar包

```
mvn clean package
```



### 4.下载安装visual studio 2022

### 5.下载对应jdk版本的graalVM

地址：`https://github.com/graalvm/graalvm-ce-builds`

### 6.配置graalVM环境变量

### 7.使用命令打包原生可执行文件

命令： `native-image --no-fallback --enable-http --enable-https --enable-all-security-services -cp cli-demo-1.0-SNAPSHOT.jar -H:Name=eg -H:Path=test com.sj.CliDemo`

（如果提示找不到vcvarsall.bat，则在搜索栏搜索`x64 Native Tools Command Prompt for VS 2022` ， 然后cd到打包位置打包，这样会携带上vs2022的环境）

参数含义：

**--no-fallback：** **禁用 fallback image**。表示如果 native 编译失败，就不生成一个依赖 JVM 的后备版本。打出来的可执行文件就是真正的“无需 JVM 的独立原生程序”。

**--enable-http：** 显式启用对 **HTTP 协议（非 HTTPS）** 的支持。如果你的程序通过 `java.net.HttpURLConnection` 或 `HttpClient` 发出 HTTP 请求，这个参数是必要的。

**--enable-https：** 显式启用对 **HTTPS 协议** 的支持。如果你的程序通过 `java.net.HttpURLConnection` 或 `HttpClient` 发出 HTTPS 请求，这个参数是必要的。

**---cp cli-demo-1.0-SNAPSHOT.jar：** 指定编译时的 **classpath**，即包含你的程序和依赖的 `.jar` 文件路径（与 Java 的 `-cp` 一致）。

**-H:Name=eg：** 指定生成的原生可执行文件名为 `eg`（无扩展名，Linux/macOS 下为 `eg`，Windows 下为 `eg.exe`）。

**-H:Path=test：** 指定将生成的 `eg` 文件输出到 `test/` 目录下，即最终路径为：`test/eg`

**com.sj.CliDemo：** 指定要执行的 Java 主类的 **全限定名**（包含包名），也是 native-image 编译的入口类。

### 8.扩展

（1）当代码中使用到反射时（或者依赖中使用到反射时），需要先使用命令执行一次jar包：`java -agentlib:native-image-agent=config-output-dir=./reflect-config  -cp cli-demo-1.0-SNAPSHOT.jar com.sj.CliDemo`，生成反射相关配置，然后使用native-image命令打包时，加上参数

`-H:ConfigurationFileDirectories=/work/reflect-config` 指定反射配置文件，最终的完整命令：`native-image --no-fallback --enable-http --enable-https --enable-all-security-services -cp cli-demo-1.0-SNAPSHOT.jar -H:Name=eg -H:ConfigurationFileDirectories=reflect-config -H:Path=test com.sj.CliDemo`

（2）如果想在命令提示符中使用上下左右等功能，需要使用`jline`相关依赖，在pom.xml中添加以下配置：

```xml
<dependency>
            <groupId>org.jline</groupId>
            <artifactId>jline</artifactId>
            <version>3.30.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.jline</groupId>
                    <artifactId>jline-terminal-jni</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.jline</groupId>
                    <artifactId>jline-terminal-jna</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.jline</groupId>
                    <artifactId>jline-terminal-jansi</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.jline</groupId>
                    <artifactId>jline-terminal-ffm</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.jline</groupId>
                    <artifactId>jline-native</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.jline</groupId>
                    <artifactId>jline-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

然后删除Scanner相关代码，替换成以下代码：

```java
Terminal terminal = null;
try {
     terminal = TerminalBuilder.builder()
                    .system(true)
                    .encoding(StandardCharsets.UTF_8)
                    .build();
     } catch (IOException e) {
            throw new RuntimeException(e);
     }

LineReader lineReader = LineReaderBuilder.builder()
                .terminal(terminal)
                .build();
String userInput = lineReader.readLine();
```

**注意：**在使用jline时，需要在打包后，手动修改`META-INF\native-image\org.jline`目录下所有文件夹中的`native-image.properties`文件的`UnlockExperimentalVMOptions`属性
