这是 Java 16+ 中引入的 Record Class（记录类）的写法，结合了接口实现和方法重写。这种语法糖主要用于创建不可变的数据载体类
**Record Class 声明**  
一个专门用于描述 **内容不可变的，记录数据的类**，它可以自动生成 对 定义数据 进行创建、设置、访问、比较等代码，所以也被称作“数据类”。记录类的对象在初始化的时候就保存了数据，以后这里面的数据是不能修改的了。

`public record ElfWeapon(WeaponType weaponType)` 是 record 的核心语法：

1. 本质就是一个类
2. 类名后面的()中填写属性列表，编译后，里面的属性会被自动赋上“private final”的修饰符
3. record类只会产生声明的构造，默认提供带参构造，且在构造中把参数赋值给对应的属性。
4. 记录类会根据类名()中的属性列表，自动生成属性的get方法。所有的方法直接使用属性名做为方法名
5. 记录类自动重写toString方法
6. 自动重写了equals方法，判断了两个记录对象里面的内容是否相等
7. 根据Java的比较规范要求，equals方法和hashCode方法必须遵守统一原则。所以，也重写hashCode方法。
### 原代码
```java
public record ElfWeapon(WeaponType weaponType) implements Weapon {

  @Override
  public String toString() {
    return "an elven " + weaponType;
  }
}
```
### 等价普通类写法
```java
public final class ElfWeapon implements Weapon {
    private final WeaponType weaponType;
    
    public ElfWeapon(WeaponType weaponType) {
        this.weaponType = weaponType;
    }
    
    @Override
    public String toString() {
        return "an elven " + weaponType;
    }
    
    // 自动生成的 equals/hashCode/accessor 方法...
}
```
