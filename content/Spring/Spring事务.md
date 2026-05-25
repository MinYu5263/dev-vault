
## 什么是事务？

事务就是一组操作，这些操作要么同时成功，要么同时失败，不能存在部分成功或者部分失败的情况。

## 事务的隔离级别

Spring 事务隔离级别对应数据库隔离级别，默认值为 `Isolation.DEFAULT`（使用数据库默认级别，如 MySQL 默认为 `REPEATABLE_READ`），可选值：

- `READ_UNCOMMITTED`：读未提交（可能出现脏读、不可重复读、幻读）
- `READ_COMMITTED`：读已提交（避免脏读，可能出现不可重复读、幻读）
- `REPEATABLE_READ`：可重复读（避免脏读、不可重复读，可能出现幻读）
- `SERIALIZABLE`：串行化（避免所有问题，性能最差）

## 如何使用事务？

### 编程式事务

SpringBoot中有两个对象：
- `DataSourceTransactionManager`：用来开启事务，提交事务或回滚事务。
- `TransactionDefinition`：事务的属性，在开启事物的时候要将`TransactionDefinition`传递进去来获取一个事务`TransactionStatus`。

```java
@RestController
public class UserController {
    @Resource
    private UserService userService;
    // JDBC 事务管理器
    @Resource
    private DataSourceTransactionManager dataSourceTransactionManager;
    // 定义事务属性
    @Resource
    private TransactionDefinition transactionDefinition;
    @RequestMapping("/sava")
    public Object save(User user) {
        // 开启事务
        TransactionStatus transactionStatus = dataSourceTransactionManager.getTransaction(transactionDefinition);
        // 插⼊数据库
        int result = userService.save(user);
        // 提交事务
        dataSourceTransactionManager.commit(transactionStatus);
        // // 回滚事务
        // dataSourceTransactionManager.rollback(transactionStatus);
        return result;
    }
}
```

这种方式使用较少，一般需要更精细的事务控制可能会用到此方式。

### 编程式事务的简化方式（TransactionTemplate）

除了 `DataSourceTransactionManager`，Spring 提供 `TransactionTemplate` 简化编程式事务：

```java
@Service
public class UserService {
    @Autowired
    private TransactionTemplate transactionTemplate;
    @Autowired
    private UserMapper userMapper;
    
    public void save(User user) {
        transactionTemplate.execute(status -> {
            try {
                userMapper.insert(user);
                return true;
            } catch (Exception e) {
                status.setRollbackOnly(); // 手动回滚
                return false;
            }
        });
    }
}
```

### 声明式事务

在需要进行事务管理的方法上添加`@Transaction`注解即可，无需手动开启和提交事务，进入方法时自动开启事务，方法执行完会自动提交事务，如果中途发生异常会自动回滚事务。

```java
@RequestMapping("/save")
@Transactional
public Object save(User user) {
    int result = userService.save(user);
    return result;
}
```

这是Spring项目最常用的方式，接下来也着重讲解这个方式的一些知识点。

### 作用范围

`@Transactional`可以⽤来修饰⽅法或类:
- 修饰方法时：需要注意只能用在`public`方法上，否则不生效。
- 修饰类时：表明该注解对该类中所有的`public`⽅法都⽣效

### @Transactional 参数

| 参数                     | 作用                                 |
| ---------------------- | ---------------------------------- |
| value                  | 当配置了多个事务管理器时，可以使用该属性指定选择哪个事务管理器    |
| transactionManager     | 当配置了多个事务管理器时，可以使用该属性指定选择哪个事务管理器    |
| propagation            | 事务的传播行为，默认值为`Propagation.REQUIRED` |
| isolation              | 事务的隔离级别，默认值为Isolation.DEFAULT      |
| timeout                | 事务的超时时间，默认值为$-1$，如果超过该时间则自动回滚事务    |
| rollbackFor            | 用于执行能够触发事务回滚的异常类型，可以指定多个异常类型       |
| rollbackForClassName   | 用于执行能够触发事务回滚的异常类型，可以指定多个异常类型       |
| noRollbackFor          | 指定异常类型不回滚事务，可以指定多个                 |
| noRollbackForClassName | 指定异常类型不回滚事务，可以指定多个                 |

## 事务的传播行为

### REQUIRED

#### 释义

这个是Spring事务默认的传播行为，也是最常用的一种。

含义：如果当前存在事务，则加入当前的事务，如果没有则创建一个新的事务

场景：例如方法A调用方法B，若方法A有事务了，则B融入A的事务；如果方法A没有事务，则B会新建一个事务。

注意：刚刚的场景描述中，我用到了融入一词，这样更加贴切。如果B融入了A的事务，那么事务的边界就由A来控制，二者也属于同一个事务了，其中任意一个方法回滚，整体都会进行回滚。

#### 代码示例

```java
@Transactional(rollbackFor = Exception.class)
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

当代码运行到`1/0`时，程序会抛出异常，这时就会回滚，数据库不会新增任何数据

### SUPPORTS

#### 释义

含义：支持当前事务，如果当前存在事务，则加入该事务；如果不在事务，则以非事务的当时执行。

场景：适用于“可选事务”的方法，例如查询操作，有无事务均可，若上层有事务则加入。

> [!question] 既然有无事务均可，为什么还需要进行事务管理？
> - 可以设置事务的隔离级别，避免查询的一些问题，例如脏读、幻读和不可重复读
> - 表面上看，`PROPAGATION_SUPPORTS` 和 “不使用事务注解” 的行为似乎很相似，但是事务不仅仅有传播行为这个属性，还有其它的属性，我们使用这个注解的时候还可以设置其它的属性，例如隔离级别、超时时间、回滚规则。
> - 使用`@Transactional(propagation = SUPPORTS)`注解可以增加代码的可读性，相当于一种非常清晰的注释，告诉后续开发者，这个方法可以在事务中执行，也可以不在事务中执行，这两种情况都被允许。

#### 代码示例

示例一：

```java
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.SUPPORTS)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：程序运行到`1/0`的地方报错，因为`insertUser`方法没有进行事务管理，`insertRole`这个方法指定了事务的传播行为为`SUPPORTS`，所以二者都没有被事务进行管理，所以即使程序报错，数据仍然插入成功

示例二：第二段代码（`RoleService`）不动，在第一段代码上方加上`@Transactional(rollbackFor = Exception.class)`注解

```java
@Transactional(rollbackFor = Exception.class)
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.SUPPORTS)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：程序运行到`1/0`的地方报错，不过这次对`insertUser`方法进行了事务管理，所以`insertRole`也加入了事务，二者都被事务管理，报错后都会滚了，数据没有插入。

### MANDATORY

#### 释义

含义：强制要求当前存在事务，否则会抛出`IllegalTransactionStateException`异常

场景：必须在事务中执行的方法，例如核心业务逻辑的校验步骤，依赖上层事务保证数据的一致性

#### 代码示例

示例一：在`insertUser`方法上面不加`@Transactional`注解

```java
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.MANDATORY)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：由于`insertUser`没有进行事务管理，用户数据插入成功，当执行`insertRole`方法时直接抛出了异常：`org.springframework.transaction.IllegalTransactionStateException: No existing transaction found for transaction marked with propagation 'mandatory'`，这就导致角色数据就没有插入。

示例二：在`insertUser`方法上面加上`@Transactional`注解，其余代码保持不变

```java
@Transactional(rollbackFor = Exception.class)
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.MANDATORY)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：此时`insertRole`方法运行在了事务里，所以不会报错，当运行到`1/0`的地方程序抛出异常，二者都进行了回滚，没有数据插入。

### REQUIRES_NEW

#### 释义

含义：无论当前是否存在事务，都会创建一个新的事务；如果当前存在事务，则将当前事务挂起，知道新的事务完成

特点：新事务与原事务完全独立，二者的提交与回滚互不影响

场景：需要独立执行事务的操作，例如日志记录，即使主事务回滚，日志也要保留

#### 代码示例

```java
@Transactional(rollbackFor = Exception.class)
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.REQUIRES_NEW)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：`insertRole`运行时是一个单独的事务，所以角色数据能插入成功。当程序运行到`int a = 1/0;`处抛出异常只会滚用户的插入数据，所以用户数据没有插入。

### NOT_SUPPORTED

#### 释义

含义：以非事务方式执行操作，如果当前存在事务，则将当前事务挂起

场景：明确不需要事物的操作，例如一些耗时的查询或统计，避免事务长期占用资源

#### 代码示例

```java
@Transactional(rollbackFor = Exception.class)
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.NOT_SUPPORTED)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：用户数据插入失败，角色数据插入成功

### NEVER

#### 释义

含义：以非事务方式执行，如果当前存在事务，则抛出`IllegalTransactionStateException`异常。

场景：严格禁止在事务中执行的操作。

#### 代码示例

示例：

```java
@Transactional(rollbackFor = Exception.class)
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.NEVER)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：运行到`insertRole`方法时直接报错，用户插入数据也会回滚，两条数据都不会插入成功。

### NESTED

#### 释义

含义：如果当前存在事务，则在嵌套事务中执行；如果没有事务则创建新的事务

特点：子事务依赖父事务，父事务回滚子事务也会回滚；但子事务可以独立回滚，且不影响父事务

场景：需要部分回滚的业务，例如批量操作中某条记录执行失败，仅回滚该记录的操作，不影响其它记录。

#### 代码示例

示例一：

```java
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.NESTED)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：程序运行到`int a = 1/0;`会抛出异常，但是两条数据都会插入成功。

示例二：

```java
@Transactional(rollbackFor = Exception.class)
public void insertUser(){
    User user = new User();
    user.setId(UUID.randomUUID().toString());
    user.setUsername("张三");
    user.setPassword("123456");
    userMapper.insert(user);
    roleService.insertRole();
    int a = 1/0;
}
```

```java
@Transactional(rollbackFor = Exception.class,propagation = Propagation.NESTED)
public void insertRole(){
    Role role = new Role();
    role.setId(UUID.randomUUID().toString());
    role.setName("管理员");
    roleMapper.insert(role);
}
```

运行结果：抛出异常，父事务提交（用户数据插入成功），子事务因异常回滚（角色数据未插入）。


## 事务的失效场景

### 事务不生效

#### 方法的访问权限

`@Transaction`注解只有作用于`public`方法上才会生效，若作用于`private`、`protected`和默认权限的方法上时则会失效。这是因为Spring事务是基于AOP动态代理实现的，而 AOP 动态代理默认只会拦截`public`方法，非`public`方法无法被代理增强，所以会导致事务失效。

#### 方法用final修饰

若方法被`final`关键字修饰，事务会失效。这是因为Spring的AOP动态代理需要通过重写目标方法来实现增强，而`final`修饰的方法无法被重写，代理逻辑无法生效，所以就会导致事务失效。

#### 方法内部调用

同一类中，非事务方法调用同类的事务方法时，事务会失效。

**示例**：

```java
@Service
public class UserService {
    // 非事务方法
    public void saveUser() {
        // 调用同类的事务方法
        doSave(); 
    }
    
    @Transactional(rollbackFor = Exception.class)
    public void doSave() {
        // 数据库操作
    }
}
```

**原因**：内部调用时，方法调用未经过 Spring 代理对象，而是直接调用目标对象方法，AOP 增强（事务）无法触发。

**解决办法**：通过 Spring 容器获取当前类的代理对象调用方法，例如：

```java
@Service
public class UserService {
    @Autowired
    private UserService userService; // 自注入代理对象
    
    public void saveUser() {
        userService.doSave(); // 通过代理对象调用
    }
    
    @Transactional(rollbackFor = Exception.class)
    public void doSave() {
        // 数据库操作
    }
}
```

#### 未被Spring容器管理

这个就很简单，Spring 事务依赖容器对 Bean 的管理，未被容器管理的类无法触发事务代理逻辑。所以使用`@Transactional`注解时，需要留意一下类上是否有`@Service`或`@Component`之类的注解。

#### 多线程调用

不同线程之间的事务相互独立，互不影响，所以会失效。

示例：

```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private RoleService roleService;
    
    @Transactional(rollbackFor = Exception.class)
    public void save() {
        // 主线程操作
        userMapper.insert(new User());
        // 子线程调用
        new Thread(() -> {
            roleService.saveRole(); // 子线程事务独立
        }).start();
        int a = 1/0; // 主线程异常
    }
}

@Service
public class RoleService {
    @Transactional(rollbackFor = Exception.class)
    public void saveRole() {
        roleMapper.insert(new Role());
    }
}
```

运行结果：主线程事务回滚（用户数据未插入），但子线程事务独立提交（角色数据插入成功）。

### 事务不回滚

#### 错误的传播特性

若使用了 `NOT_SUPPORTED`、`SUPPORTS`（无外层事务时）等传播行为，事务可能不回滚。

示例：

```java
@Service
public class UserService {
    @Transactional(rollbackFor = Exception.class, propagation = Propagation.NOT_SUPPORTED)
    public void save() {
        userMapper.insert(new User());
        int a = 1/0; // 异常不会触发回滚
    }
}
```

运行结果：数据插入成功，异常未触发回滚。

#### 自己吞了异常

方法内捕获异常但未重新抛出，事务不会回滚。

示例：

```java
@Transactional(rollbackFor = Exception.class)
public void save() {
    try {
        userMapper.insert(new User());
        int a = 1/0;
    } catch (Exception e) {
        // 仅捕获异常未抛出
        log.error("异常", e);
    }
}
```

运行结果：数据插入成功，事务未回滚。

#### 抛了不触发回滚的异常

默认情况下，`@Transactional` 仅对 `RuntimeException` 和 `Error` 触发回滚，若抛出受检异常（如 `IOException`）且未指定 `rollbackFor`，事务不回滚。

示例：

```java
@Transactional
public void save() throws IOException {
    userMapper.insert(new User());
    throw new IOException("异常"); // 受检异常，默认不回滚
}
```

#### 自定义回滚异常

指定的回滚异常与抛出的异常不一致

示例：

```java
@Service
public class OrderServiceImpl implements OrderService {

    @Resource
    private OrderItemServiceImpl orderItemService;

    @Override
    @Transactional(rollbackFor = BusinessException.class)
    public void updateOrder() {
        //TODO
        orderItemService.updateOrderItem();
    }
}

@Service
class OrderItemServiceImpl {
    public void updateOrderItem()  {
        //更新业务
    }
}
```

#### 嵌套事务异常

在嵌套事务中，若子事务设置了传播行为是：`propagation = Propagation.NESTED`，则子事务异常只回滚了子事务，父事务不会回滚。
