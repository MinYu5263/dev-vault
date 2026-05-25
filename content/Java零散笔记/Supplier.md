在Java中，`Supplier<T>`是一个函数式接口，它的核心作用是**延迟提供或生成一个特定类型的值**
### **核心特性**

- ​**无参方法**：`Supplier` 的唯一抽象方法是 `T get()`，它不需要参数，直接返回一个 `T` 类型的值。
- ​**延迟执行**：通过封装获取值的逻辑，`Supplier` 允许在需要时才计算或生成值，而不是立即执行。
- ​**无异常抛出**：与 `Callable` 不同，`Supplier` 的 `get()` 方法**不会抛出检查异常**，适合非阻塞操作。
### **典型应用场景**

#### 1. ​**延迟计算与资源优化**

- 当创建对象或计算开销较大时，使用 `Supplier` 可以推迟操作，直到真正需要结果。

```java
Supplier<ExpensiveObject> supplier = () -> new ExpensiveObject();
// 直到调用 get() 时才实例化
ExpensiveObject obj = supplier.get();
```

#### 2. ​**结合 `Optional` 处理空值**

- `Optional.orElseGet(Supplier)` 在值为空时，通过 `Supplier` 动态生成默认值，避免提前计算浪费资源。

```java
Optional<String> optional = Optional.empty();
String value = optional.orElseGet(() -> "Default Value");
```

#### 3. ​**生成无限流（Stream API）​**

- 与 `Stream.generate()` 配合，用 `Supplier` 持续提供元素，创建无限流。

```java
Stream<String> infiniteStream = Stream.generate(() -> "Element");
```

#### 4. ​**工厂模式与依赖注入**

- 通过传递 `Supplier`，解耦对象创建逻辑，使代码更灵活。

```java
public void process(Supplier<Connection> connectionSupplier) {
    Connection conn = connectionSupplier.get();
    // 使用连接
}
// 调用时传入不同的 Supplier 实现
process(() -> createDatabaseConnection());
```
### **注意事项**

- ​**线程安全**：若在多线程环境中共享 `Supplier`，需确保 `get()` 的实现是线程安全的。
- ​**避免副作用**：理想情况下，`Supplier` 应该是无副作用的，多次调用 `get()` 的结果可能相同也可能不同，取决于具体实现。