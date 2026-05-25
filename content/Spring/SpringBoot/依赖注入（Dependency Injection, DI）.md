
## 什么是依赖注入？

依赖注入是控制反转（IoC）思想的具体实现，核心是**将对象的依赖由外部容器（如 Spring）负责创建和注入，而非对象自身创建依赖**。通过这种方式，可大幅降低类之间的耦合度，提高代码的可测试性和可维护性。

简单来说：假设类 A 需要用到类 B 的功能，则 B 是 A 的依赖。传统方式中 A 会主动`new B()`创建依赖，而依赖注入中 A 只需声明依赖，由容器负责将 B 的实例 “注入” 到 A 中。

## 注入方式

### 构造器注入（Constructor Injection）

- 通过类的构造方法注入依赖，是 Spring 官方推荐的注入方式
- 能保证注入的依赖不可变（final修饰）且在对象创建时完成初始化，避免 “半初始化” 状态

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired  // Spring 4.3+后，单构造器可省略@Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

**Lombok 简化**：使用`@RequiredArgsConstructor`为`final`字段自动生成构造器

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;
}
```

### Setter 方法注入（Setter Injection）

- 通过 Setter 方法注入依赖，灵活性较高，允许在对象创建后重新设置依赖

```java
@Service
public class UserService {
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### 字段注入（Field Injection）

- 直接在类的字段上使用注解注入，代码简洁但存在一定争议
- 缺点：无法注入 final 修饰的字段，不利于单元测试，依赖关系不直观，过度依赖 Spring 容器

```java
@Service
public class UserService {
	
    @Autowired
    private UserRepository userRepository;
}
```

## 注入注解对比（@Autowired 与 @Resource）

| 特性     | @Autowired（Spring 注解）   | @Resource（JSR-250 规范）  |
| ------ | ----------------------- | ---------------------- |
| 默认注入方式 | 按类型（byType）             | 按名称（byName）            |
| 名称指定方式 | 需配合`@Qualifier("name")` | 直接通过`name`属性指定         |
| 兼容性    | 仅 Spring 环境支持           | 支持多种容器（非 Spring 也可能兼容） |

## 实战推荐

在实际开发中，**最推荐使用构造器注入**，其次是 Setter 注入，对于字段注入，应尽量避免。

## 关键问题解析

### 为什么最推荐构造器注入？

- **保证依赖不可变**：构造器注入的依赖通常用`final`修饰，一旦初始化后就无法修改，符合 "不可变对象更安全" 的设计原则，能避免多线程环境下的潜在问题。
- **确保对象完整性**：依赖在对象创建时就必须注入，避免了 "半初始化" 状态（对象已创建但依赖未注入），调用任何方法时都能保证依赖可用，减少NullPointerException风险。
- **方便测试**：在单元测试中可以通过`new`关键字直接传入模拟对象，无需依赖 Spring 容器，测试更简单。
- **提前暴露循环依赖**：若存在循环依赖（如 A 依赖 B，B 依赖 A），Spring 启动时会直接抛出`BeanCurrentlyInCreationException`，强制开发者解决设计问题。

### 什么时候需要用 Setter 注入？

- **依赖可选**：有这个依赖可以运行，没有这个依赖也能运行（如日志组件）。
- **依赖需要动态变更**：运行时需切换依赖实现（如切换数据源、缓存策略）。


> [!tip] 提示
> 虽然动态变更依赖很灵活，但同时也带来了一定的风险。<br>使用时需要注意：
> - 使用前要保证注入了需要用到的依赖，否则会存在空指针的风险
> - 字段没有使用`final`修饰，依赖可能会被修改，导致不可预期的问题。

### 为什么不推荐字段注入（@Autowired 直接标注字段）？

- 依赖关系不够直观：仅通过类定义无法直观知道它依赖哪些组件，必须查看字段注解才能了解，降低了代码的可读性。
- 无法使用`final`：字段注入的依赖不能用final修饰，意味着依赖可能被随时修改，破坏了对象的不可变性。
- 严重依赖Spring容器：脱离 Spring 容器后，无法通过new关键字创建对象（因为依赖注入由容器完成），导致单元测试必须启动 Spring 容器（如@SpringBootTest），大幅降低测试效率。
- 可能导致循环依赖：字段注入容易掩盖循环依赖问题（如 A 依赖 B，B 依赖 A），而构造器注入会在启动时直接报错，更早暴露问题。

### 构造器注入必须用`final`修饰字段吗？

构造器注入不强制要求依赖字段用`final`修饰，但强烈建议使用`final`。不使用`final`的构造器注入虽然语法上是合法的，Spring也能正常注入依赖，但是极不推荐这样做。<br>
因为这样做，依赖的字段后续可以被修改，破坏了对象的稳定性；再者，无法体现 "依赖是类的必要组成部分" 的设计意图。

### 若`final`字段无需 Spring 注入，如何处理？

- 如果这个字段是常量，直接赋值就可以。
- 如果想要更灵活的控制，可以手动写构造函数。
