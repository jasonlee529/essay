---
title: Java如何调用shell脚本的
date: 2018-05-08 11:59:42
categories:
- Java
tags:
- Java
- shell
---
有些时候会碰到这样的场景：java的功能里面要嵌入一个功能点，这个功能是通过是shell脚本实现的。这种时候就需要Java对脚本调用的支持了。

## 测试环境
Ubuntu16.04 i3-6100,12GB
## Hello World
来看一个基本的例子
```java
    Process exec = Runtime.getRuntime().exec(new String[] { "uname" ,"-a"});
    exec.waitFor();
    BufferedReader reader =
            new BufferedReader(new InputStreamReader(exec.getInputStream()));
    System.out.println(reader.readLine());

Linux jason-Inspiron-3650 4.4.0-121-generic #145-Ubuntu SMP Fri Apr 13 13:47:23 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
## 解读Process
java.lang.Process类提供了获取输入、输出、等待执行和销毁进程的方法。
Process类可通过ProcessBuilder.start() 和 Runtime.exec 创建实例，从Java1.5开始，ProcessBuilder.start()是更推荐的做法，但网上的教程更多推荐用Runtime.exec()方法。

| Modifier and Type      	| Method               	| Description                                                                                                              	|
|------------------------	|----------------------	|--------------------------------------------------------------------------------------------------------------------------	|
| abstract void          	| destroy ()           	| Kills the subprocess.                                                                                                    	|
| abstract int           	| exitValue ()         	| Returns the exit value for the subprocess.                                                                               	|
| abstract  InputStream  	| getErrorStream ()    	| Returns the input stream connected to the error output of the subprocess.                                                	|
| abstract  InputStream  	|  getInputStream ()   	| Returns the input stream connected to the normal output of the subprocess.                                               	|
| abstract  OutputStream 	|  getOutputStream ()  	| Returns the output stream connected to the normal input of the subprocess.                                               	|
| abstract int           	| waitFor ()           	| Causes the current thread to wait, if necessary, until the process represented by this Process object has terminated.    	|

继承体系上面，Process的实现类是JDK内置的，linux版本的jdk中只带有一个实现类UnixProcess。

## 与脚本交互
Process不但可以执行进程，还可以获取进程的返回结果。
```java
    private String executeCommand(String command) {
        StringBuffer output = new StringBuffer();
        Process p;
        try {
            p = Runtime.getRuntime().exec(command);
            int exitCode = p.waitFor();
            System.out.println(exitCode);
            BufferedReader reader =
                    new BufferedReader(new InputStreamReader(p.getInputStream()));
            String line = "";
            while ((line = reader.readLine()) != null) {
                output.append(line + "\n");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(output.toString());
        return output.toString();
    }

PING www.a.shifen.com (111.13.100.91) 56(84) bytes of data.
64 bytes from localhost (111.13.100.91): icmp_seq=1 ttl=52 time=7.66 ms
64 bytes from localhost (111.13.100.91): icmp_seq=2 ttl=52 time=7.90 ms
64 bytes from localhost (111.13.100.91): icmp_seq=3 ttl=52 time=14.0 ms

--- www.a.shifen.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 7.668/9.861/14.013/2.937 ms

```
## 总结
Java 执行脚本的方式其实类似用直接在bash里面执行脚本，区别在于环境有些变动，执行的效果和bash基本一致。