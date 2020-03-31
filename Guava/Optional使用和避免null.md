> "Null sucks." -Doug Lea  
> "I call it my billion-dollar mistake." - Sir C. A. R. Hoare, on his invention of the null reference

轻率地使用null可能会导致很多令人费解的问题。我们发现95%的集合类不接受null值作为元素。我们认为，相比默默地接受null，使用快速失败操作拒绝null值对开发者更有帮助。

null的含糊语义让人很不舒服。null很少可以明确地表示某种语义，例如，Map.get(key)返回null时，可能表示map中的值是null，亦或map中没有key对应的值。null可以表示失败、成功或几乎任何情况。使用Null以外的特定值，会让你的逻辑描述变得更清晰。

null确实也有合适和正确的使用场景，如在性能和速度方面null是廉价的，而且在对象数组中，出现null也是无法避免的。但相对于底层库来说，在应用级别的代码中，null往往是导致混乱，疑难问题和模糊语义的元凶，就如同我们举过的Map.get(key)的例子。最关键的是，null本身没有定义它表达的意
思。

鉴于这些原因，很多Guava工具类对Null值都采用快速失败操作，除非工具类本身提供了针对Null值的因变措施。此外，Guava还提供了很多工具类，让你更方便地用特定值替换Null值。

## Optional

开发人员使用null表明的是某种缺失情形：可能是已经有一个默认值，或没有值，或找不到值。例如，Map.get返回null就表示找不到给定键对应的值。

Guava用Optional<T>表示可能为null的T类型引用。一个Optional实例可能包含非null的引用（我们称之为引用存在），也可能什么也不包括（称之为引用缺失）。它从不说包含的是null值，而是用存在或缺失来表示。但Optional从不会包含null值引用。

com.google.common.base.Optional<T>类的声明

```java
@GwtCompatible(serializable = true)
public abstract class Optional<T> implements Serializable {}
```

### 创建Optional实例

1. Optional.of(T)

    创建指定引用的Optional实例，若引用为null则快速失败

2. Optional.absent()

    创建引用缺失的Optional实例

3. Optional.fromNullable(T)

    创建指定引用的Optional实例，若引用为null则表示缺失

以上3个都是static的方法

### 查询引用

1. boolean isPresent()

    如果Optional包含非null的引用（引用存在），返回true

2. T get()

    返回Optional所包含的引用，若引用缺失，则抛出java.lang.IllegalStateException

3. T or(T)

    返回Optional所包含的引用，若引用缺失，返回指定的值

4. T orNull()

    返回Optional所包含的引用，若引用缺失，返回null

5. Set<T> asSet()

    返回Optional所包含引用的单例不可变集，如果引用存在，返回一个只有单一元素的集合，如果引用缺失，返回一个空集合。

以上5个都是非static的方法

```java
@Test
public void testOption(){
    Optional<Integer> possible = Optional.of(5);
    Optional<Integer> absent = Optional.absent();
    Optional<Integer> nullable = Optional.fromNullable(null);
    Optional<Integer> noNullable = Optional.fromNullable(6);

    System.out.println("possible isPresent:" + possible.isPresent());
    System.out.println("absentOpt isPresent:" + absent.isPresent());
    System.out.println("nullable isPresent:" + nullable.isPresent());
    System.out.println("noNullable isPresent:" + noNullable.isPresent());

    System.out.println("possible value:" + possible.get());
    System.out.println("nullable or : " + nullable.or(6));
    System.out.println("nullable orNull : " + nullable.orNull());

    Set<Integer> set = noNullable.asSet();
    System.out.println("noNullable asSet: " + set.size());
}
```

输出：

```
possible isPresent:true
absentOpt isPresent:false
nullable isPresent:false
noNullable isPresent:true
possible value:5
nullable or : 6
nullable orNull : null
noNullable asSet: 1
```

## Optional的意义

使用Optional除了赋予null语义，增加了可读性，最大的优点在于它是一种傻瓜式的防护。Optional迫使你积极思考引用缺失的情况，因为你必须显式地从Optional获取引用。直接使用null很容易让人忘掉某些情形，尽管FindBugs可以帮助查找null相关的问题，但是我们还是认为它并不能准确地定位问题根源。

和入参一样，方法的返回值也可能是null。和其他人一样，你绝对很可能会忘记别人写的方法method(a,b)会返回一个null，就好像当你实现method(a,b)时，也很可能忘记输入参数a可以为null。将方法的返回类型指定为Optional，也可以迫使调用者思考返回的引用缺失的情形。

## 处理null其他方法

当你需要用一个默认值来替换可能的null，请使用Objects.firstNonNull(T, T) 方法。如果两个值都是null，该方法会抛出NullPointerException。Optional也是一个比较好的替代方案，例如：Optional.of(first).or(second).

还有其它一些方法专门处理null或空字符串：

* emptyToNull(String)
* nullToEmpty(String)
* isNullOrEmpty(String)

我们想要强调的是，这些方法主要用来与混淆null/空的API进行交互。当每次你写下混淆null/空的代码时，Guava团队都泪流满面。

> 好的做法是积极地把null和空区分开，以表示不同的含义，在代码中把null和空同等对待是一种令人不安的坏味道。
