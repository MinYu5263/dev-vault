`Files.createDirectories()` 是 Java NIO（New I/O）库中一个用于创建目录的实用方法，属于 `java.nio.file.Files` 类。它的核心功能是 **递归创建目录** ，包括所有不存在的父目录。与 `Files.createDirectory()` 不同，如果目录已经存在，它不会抛出异常，这使得它更安全且适合处理不确定目录是否存在的情况。

---

### 方法签名

```java
java复制public static Path createDirectories(Path dir, FileAttribute<?>... attrs) throws IOException
```
- **参数**:
	- `dir`: 要创建的目录的 `Path` 对象（可以是多层嵌套路径）。
	- `attrs` (可选): 指定目录的属性（例如权限），可变参数，可为空。
- **返回值**: 返回创建的目录的 `Path` 对象。
- **异常**:
	- 如果目录创建失败（如权限不足），抛出 `IOException` 。
	- 如果 `dir` 是 `null` ，抛出 `NullPointerException` 。

---

### 核心行为

1. **递归创建目录**:
	- 如果目标路径中的任何父目录不存在，会自动创建所有缺失的目录。
	- 例如： `dir1/dir2/dir3` ，如果 `dir1` 和 `dir2` 不存在，会先创建它们，最后创建 `dir3` 。
2. **幂等性**:
	- 如果目录已存在，直接返回该目录的 `Path` ，不会抛出异常。
	- 如果路径已经存在但不是一个目录（比如是一个文件），抛出 `FileAlreadyExistsException` 。
3. **原子性**:
	- 尽可能保证操作的原子性（例如，避免部分目录创建后失败导致中间状态）。

---

### 对比其他方法

| 方法名 | 行为 |
| --- | --- |
| `Files.createDirectory()` | 只能创建单层目录，父目录必须存在，否则抛异常。 |
| `Files.createDirectories()` | 自动创建所有缺失的父目录，目录存在时静默返回。 |
| `File.mkdir()` | 单层目录，父目录不存在返回 `false` 。 |
| `File.mkdirs()` | 类似 `createDirectories` ，但返回 `boolean` 而非 `Path` 。 |

---

### 代码示例

#### 示例 1：基本用法

```java
java复制import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class CreateDirectoriesExample {
    public static void main(String[] args) {
        Path path = Paths.get("myapp", "data", "logs"); // 对应路径 myapp/data/logs
        try {
            Path createdDir = Files.createDirectories(path);
            System.out.println("目录已创建: " + createdDir);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
- 如果 `myapp/data/logs` 不存在，会递归创建所有目录。
- 如果目录已存在，静默返回路径。

#### 示例 2：指定目录权限（Linux/Unix）

```java
java复制import java.nio.file.attribute.PosixFilePermission;
import java.nio.file.attribute.PosixFilePermissions;
import java.util.Set;

Path dir = Paths.get("secure-dir");
Set<PosixFilePermission> perms = PosixFilePermissions.fromString("rwxr-x---");
FileAttribute<Set<PosixFilePermission>> attr = PosixFilePermissions.asFileAttribute(perms);
Files.createDirectories(dir, attr);
```
- 为目录设置权限 `rwxr-x---` （仅限所有者读写执行，组只读执行）。

---

### 异常处理场景

1. **权限不足**:
	```java
	java复制Path restrictedPath = Paths.get("/etc/myapp");
	try {
	    Files.createDirectories(restrictedPath);
	} catch (IOException e) {
	    System.err.println("权限不足: " + e.getMessage());
	}
	```
2. **路径被文件占用**:
	```java
	java复制// 假设 myapp/data 是一个文件而不是目录
	Path invalidPath = Paths.get("myapp/data/logs");
	try {
	    Files.createDirectories(invalidPath);
	} catch (FileAlreadyExistsException e) {
	    System.err.println("路径被文件占用: " + e.getFile());
	}
	```

---

### 适用场景

- 需要确保某个目录存在（如日志目录、临时文件目录）。
- 部署应用时初始化目录结构。
- 处理用户上传文件时动态创建存储路径。

---

### 总结

`Files.createDirectories()` 是处理目录创建的最佳实践方法，尤其适合需要递归创建多层目录的场景。它的幂等性（存在时不报错）和安全性（自动处理父目录）使其比传统方法更可靠。结合 `FileAttribute` 还可以实现跨平台的权限控制。