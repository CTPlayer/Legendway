### 自动装箱
它允许程序员将基本类型和装箱基本类型混用，按需自动装箱和拆箱。它们之间的差别变得模糊，但是并没有完全的消除。除了语意上有微妙的差别，在性能上也有着明显的差别。
```java
public static void main(String[] args) {
    Long sum = 0L;
    for (int i = 0;i < Integer.MAX_VALUE;i ++) {
        sum += i;
    }
    System.out.println(sum);   
}
```
变量sum被声明成Long而不是long，意味着程序每次往Long sum中增加long时构造一个实例。所以结论很明显：要优先使用基本类型而不是装箱基本类型，要当心无意识的自动装箱。

### equals与hashcode
默认情况下，类的实例只与它自生相等。  
Object的hashCode方法是本地方法，也就是用c语言或c++实现的，该方法通常用来将对象的内存地址转换为整数之后返回。

**为什么重写equals时必须重写hashCode方法？** 如果两个对象相等，则hashCode一定也是相同的。hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写
hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。