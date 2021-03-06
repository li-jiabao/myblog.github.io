---
title: Java进程
author: lijiabao
date: 2021-03-17  20:00
categories:  Java并发
tags: Java并发
---

主要介绍了一下进程的简单创建和进程构建器的一些方法

## Java进程

通过线程的学习，我们已经了解到了Java怎么在同一个程序的不同线程执行Java代码。但有时候还需要执行另外一个程序，可以通过使用`Process`和`ProcessBuilder`类。`Process`类在一个单独的操作系统进程中执行一个命令,允许我们与标准输入输出和错误流交互.`ProcessBuilder`(可以取代`Runtime.exec`使用,且更加的灵活)允许我们配置`Process`对象

### 创建一个进程

首先指定你想要执行的命令,可以提供一个`List<String>`,或者直接提供命令字符串.

```java
var builder = ProcessBuilder("gcc", "myapp.c");
var builder = ProcessBuilder("/bin/ls", "/home");
// 注意！！！第一个字符串必须是一个可执行的命令,而不是shell内置命令.如Windows的cmd.exe
// 使用dir打印,需要提供"cmd.exe""/C"和"dir"
var builder = ProcessBuilder("cmd.exe","/C","dir");
```

每个进程都有一个工作目录,用来解析相对目录.默认情况下,进程的工作目录与虚拟机一样,通常是启动Java程序的那个目录。这个目录可以使用directory方法来改变。

```java
var builder new ProcessBuilder("/bin/ls", "-l");
builder = builder.derectory(path.toFile());
// 由于ProcessBuilder的各方法都返回自身，因此可以把命令串起来。
var builder = new ProcessBuilder("/bin/ls","-l").dirctory(path.toFile());
```

指定如何处理标准输入输出和错误流：默认情况下，他们分别是一个管道

```java
// 获取输入输出和错误流的对象
OutputStream out = p.getOutputStream();
InputStream in = p.getInputStream();
InputStream err = p.getErrorStream();

// 进程的输入是jvm的输出流，进程输入流是jvm的输出流
// 指定进程的输入、输出和错误流与jvm一样，通过下面的方法
builder.redirectIO();
// 如果只想对特定的流这样，可以把值ProcessBuilder.Redirect.INHERIT传入
// redirectInput、redirectOutput、redirectError三种方法中
builder.redirectInput(ProcessBuilder.Redirect.INHERIT);
// 若是将File对象传入这三种方法，将会重定向到文件中
builder.redirectInput(new File());

// 合并输出流和错误流通常很有用，这样可以按进程生成这些消息的顺序显示输出和错误信息
builder.redirectErrorStream(true);   // 合并输出流和错误流
// 合并了输出和错误流之后，不再可以使用redirectError，也不能在process调用getErrorStream了

// 还可以修改环境变量,首先需要得到构建起的环境变量(运行jvm的进程的环境变量初始化)然后再加入或者移除环境条目
Map<String, string> env = builder.environment();
evc.put("LANG","zh_CN");
env.remove("JAVA_HOME");
Process p = builder.start();

// 将一个进程的输出当成另外一个进程的输入,类似shell中的`|`管道操作符
// Java9中提供了一个startPipeline方法,可以传入一个进程构建起的列表,并从最后一个进程获取结果
List<Process> processes = ProcessBuilder.startPipeline(List.of(
    new ProcessBuilder("find", "/opt/jdk-9"),
    new ProcessBuilder("grep", "-o", "\\.[^./}*$"),
    new ProcessBuilder("sort"),
    new ProcessBuilder("uniq")
));
Process last = processes.get(processes.size() - 1);
var result = new String(last.getOutputStream().readAllBytes());
```

### 运行一个进程

配置好构建起之后,需要使用他的start()方法启动进程.如果把输入、输出和错误流配置为管道，就可以在start之后，写输入流，读取输出流和错误流；

```java
Process p = new ProcessBuilder("/bin/ls", "-l")
    .directory(Path.of("/tmp").toFile())
    .start();
try (var in = new Scanner(p.getInputStream())) {
    while (in.hasNext()) System.out.println(in.nextLine());
}
// 进程流的缓冲空间是有限的，不能够下入太多输入，而且需要及时读取输出。
// 如果有大量的输入和输出,可能需要再单独的线程中生产和消费这些输出输入
```

要等待进程完成,可以调用`waitFor()`方法:`int result = p.waitFor()`

如果不想无限期的等待,你可以使用:

```java
long delay = ...;
if (process.waitFor(delay, TimeUnit.SECONDS)) {
    ...;
}else {
    process.destroyForcibly();
}
```

最后会在进程完成时，接收到一个异步通知。滴哦用`process.onExit()`会得到一个`CompletableFuture<Process>`,可以用来调度任何动作。

```java
process.onExit().thenAccept(
p -> System.out.println("EXIT value" + p.exitValue()));
```

### 进程句柄

要获取程序启动的一个进程的更多信息，或者想更多地了解你的计算机上正在运行的任何其他进程，可以使用`ProcessHandle`接口，有四种方法可以获取：

- 给定一个Process对象p，`p.toHandle()`会生成它的`ProcessHandle`
- 给定一个long类型的操作系统进程ID，`ProcessHandle.of(id)`可以生成这个进程的句柄
- `Process.current()`是运行这个Java虚拟机的进程的句柄
- `ProcessHandle.allProcess()`可以生成对当前进程可见的所有操作系统进程的`Stream<ProcessHandle>`

给定一个进程句柄，可以得到他的进程ID、父进程、子进程和后代进程：

```java
long pid = handle.pid();
Optional<ProcessHandle> parent = handle.parent();
Stream<ProcessHandle> children = handle.children();
Stream<ProcessHandle> descendants = handle.descendants();
```

`java.lang.ProcessBuilder`类：

- `ProcessBuilder(string... command)`
- `ProcessBuilder(List<String> command)`用给定的命令和参数生成一个进程构建器
- `ProcessBuilder directory(File directory)`设置进程的工作目录

Modifier and Type | Method and Description
:---: | :---: 
`List<String>` | `command()` 返回操作系统进程构建器的进程 
`ProcessBuilder` | `command(List<String> command)`设置当前参数构成的线程构建器 
`ProcessBuilder` | `command(String... command)`设置当前参数组成的线程构建器 
`File` | `directory()`返回线程构建器的工作目录 
`ProcessBuilder` | `directory(File directory)`设置进程构建器的工作目录 
`Map<String,String>` | `environment()`返回当前进程构建器的环境变量的映射 
`ProcessBuilder` | `inheritIO()`设置进程的输入输出和当前Java进程一样 
`ProcessBuilder.Redirect` | `redirectError()`返回当前进程构建器的错误流去向 
`ProcessBuilder` | `redirectError(File file)`设置当前进程构建器的错误流到文件 
`ProcessBuilder` | `redirectError(ProcessBuilder.Redirect destination)`设置当前进程构建器的错误流去向 
`boolean` | `redirectErrorStream()`告诉进程构建器是否合并错误流和输出流 
`ProcessBuilder` | `redirectErrorStream(boolean redirectErrorStream)`设置当前进程构建器的重定向错误流特性 
`ProcessBuilder.Redirect` | `redirectInput()`返回当前进程构建器输入流来源 
`ProcessBuilder` | `redirectInput(File file)`设置进程构建器的输入流来源为文件对象 
`ProcessBuilder` | `redirectInput(ProcessBuilder.Redirect source)`设置进程构建器的输入流来源 
`ProcessBuilder.Redirect` | `redirectOutput()`返回当前进程构建器的输出流去向 
`ProcessBuilder` | `redirectOutput(File file)`设置当前进程构建器的输出流到文件 
`ProcessBuilder` | `redirectOutput(ProcessBuilder.Redirect destination)`设置当前进程构建器的输出流去向 
`Process` | `start()`开启当前进程构建器代表的进程 













