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