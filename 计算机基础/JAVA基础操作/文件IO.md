### Path

* Path 用来表示文件路径
* Paths 是工具类，用来获取 Path 实例

```java
public static Path get(URI uri){/***/} 

Path source = Paths.get("1.txt"); // 相对路径 使用 user.dir 环境变量来定位 1.txt

Path source = Paths.get("d:\\1.txt"); // 绝对路径 代表了  d:\1.txt
```

其中

* `.` 代表了当前路径
* `..` 代表了上一级路径

### Files

##### 检查文件是否存在

```java
Path path = Paths.get("helloword/data.txt");
System.out.println(Files.exists(path));
```

##### 创建一级目录

```java
Path path = Paths.get("helloword/d1");
Files.createDirectory(path);
```

* 如果目录已存在，会抛异常 FileAlreadyExistsException
* 不能一次创建多级目录，否则会抛异常 NoSuchFileException

##### 创建多级目录用

```java
Path path = Paths.get("helloword/d1/d2");
Files.createDirectories(path);
```

##### 拷贝文件

```java
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/target.txt");

Files.copy(source, target);
```

* 如果文件已存在，会抛异常 FileAlreadyExistsException

如果希望用 source 覆盖掉 target，需要用 StandardCopyOption 来控制

```java
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
```

##### 移动文件

```java
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/data.txt");

Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
```

* StandardCopyOption.ATOMIC_MOVE 保证文件移动的原子性

##### 删除文件

```java
Path target = Paths.get("helloword/target.txt");

Files.delete(target);
```

* 如果文件不存在，会抛异常 NoSuchFileException

##### 删除目录

```java
Path target = Paths.get("helloword/d1");

Files.delete(target);
```

* 如果目录还有内容，会抛异常 DirectoryNotEmptyException

##### 遍历目录文件

```java
//1.
public static Path walkFileTree(Path start, FileVisitor<? super Path> visitor){/***/};
//FileVisitor可传入下面的简单实例，根据需要可重写visitFile()/preVisitDirectory()/postVisitDirectory()
new SimpleFileVisitor<Path>(){}

//2.
public static Stream<Path> walk(Path start, FileVisitOption... options){/***/};
//用流的方式去遍历
```

### 文件I/O

##### 写入文件

```java
//1.
Path path = Paths.get("");
BufferedWriter writer = Files.newBufferedWriter(path, StandardCharsets.UTF_8);
writer.write("Hello");

BufferedWriter writer = Files.newBufferedWriter(path, StandardCharsets.UTF_8,StandardOpenOption.APPEND);//追加写
writer.write("Hello");

//2.
String filename;
PrintWriter writer = new PrintWriter(filename,"UTF-8");
writer.println("..");

//3.
Files.write(path,"Hello".getBytes(StandardCharsets.UTF-8));

//4.
try(BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(new FileOutputStream(filename)))){
    bw.write("Hello");
    bw.flush();
};
```

##### 读取文件

```java
String filename;
//1.
try(Scanner sc = new Scanner(new FileReader(filename))){};

//2.
Stream<String> lines = Files.lines(Paths.getPath(filename));
lines.forEach();
lines.forEachOrdered();//按照读取顺序

//3.
List<String> lines = Files.readAllLines(Paths.getPath(filename));

//4.管道流
try(BufferedReader br = new BufferedReader(new FileReader(filename))){
    String line;
    while((line=br.readLine())!=null){
        System.out.println(line);
    }
}
```

