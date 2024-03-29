
>Refrences
>https://advancedweb.hu/new-language-features-since-java-8-to-21/
>https://advancedweb.hu/a-categorized-list-of-all-java-and-jvm-features-since-jdk-8-to-21/


## JKD 9
#### Allow private methods in interfaces

It just for ==tidy up== the interface and increase encapsulation and resusability of code.

```java
public interface SumInterface {  
    default int sum(int i, int j) {  
        return doSum(i, j);  
    }  
  
    private int doSum(int i, int j) {  
        return i + j;  
    }  
}

```


#### Diamond operator for anonymous inner classes

###### What is a diamond operator?
- new feature in java SE 7
-  The purpose of diamond operator is to avoid redundant code by leaving the generic type in the right side of the expression.
```java
List<String> myList = new ArrayList<String>();
List<String> myList = new ArrayList<>();
```
 
###### Problem with the diamond operator

 it didn’t allow us to use them in anonymous inner classes.
 ```java
 abstract class MyClass<T>{  
    abstract T add(T num, T num2);  
}  
public class JavaExample {  
    public static void main(String[] args) {  
        MyClass<Integer> obj = new MyClass<>() {  
            Integer add(Integer x, Integer y) {  
                return x+y;   
            }  
        };    
        Integer sum = obj.add(100,101);  
        System.out.println(sum);  
    }  
}
```

```
JavaExample.java:7: error: cannot infer type arguments for MyClass
        MyClass obj = new MyClass<>() {
```

###### Java 9 – Diamond operator enhancements

Java 9 improved the use of diamond operator and allows us to use the diamond operator with anonymous inner classes.

#### Allow effectively-final variables to be used as resources in try-with-resources statements

Despite `try-with-resources` has power, it had a few ==shortcomings== that Java 9 addressed.
- Handle multiple resources in try block, make the code harder to read.
- If you already have a variable that you want to handle with this construct, you had to introduce a dummy variable.

To mitigate these criticisms in Java9, `try-with-resources` was enhanced to handle final or effectively final local variables in addition to newly created ones:

```java
BufferedReader br1 = new BufferedReader(...);
BufferedReader br2 = new BufferedReader(...);
try (br1; br2) {
    System.out.println(br1.readLine() + br2.readLine());
}
```

#### Underscore is no longer a valid identifier name
This block of code is invalid in Java9, but allowed and convey special meaning in Java21.

```java
int _ = 10;
```


## JDK 11

#### Type Inference

Use `var` instead of explicit type, It makes this piece of code less redundant, thus, easier to read.
```java
var greetingMessage = "Hello!";
```
The type of the declared variables is **inferred at compile time**

##### Maybe use `var` has backfire
- Reduce readability
  For example `.getDayOfWeek` in Java 8's Date/Time return `java.time.DayOfWeek` but in `Joda Time` return `int` so use `var` in below example made confusing and hard to determind the the type.
 ```java
var dayOfWeek = date.getDayOfWeek();
```
- Problem in Diamond Operator
  The Diamond operator is a nice feature to remove some verbosity from the right-hand side of an expression. But when using `var`  it iference `Map<Object, Object>` that It Is not valid even emit without a compiler warning. 
```java
Map<String, String> myMap = new HashMap<>();
var myMap = new HashMap<>(); //Map<Object, Object>
```


## JDK 17

#### Switch Expressions
Populate variable with Switch Expression.
```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    default      -> {
        String s = day.toString();
        int result = s.length();
        yield result;
    }
};
```

##### Differences between Switch Statement and Switch Expression
- Switch expressions **cases don't fall-through**. So no more subtle bugs caused by missing `breaks`. To avoid the need for fall-through, **multiple constants can be specified for each case** in a comma separated list.
- Each **case has its own scope**. use `yield` instead of `return`. 
```java
String s = switch (k) {
    case  1 -> {
        String temp = "one";
        yield temp;
    }
    case  2 -> {
        String temp = "two";
        yield temp;
    }
    default -> "many";
}
```
- **Cases of a switch expression are exhaustive**. This means that for String, primitive types and their wrappers the `default` case always has to be defined.
  For `enums` either a `default` case has to be present, or all cases have to be explicitly covered.
```java
int k = 3;
String s = switch (k) {
    case  1 -> "one";
    case  2 -> "two";
    default -> "many";
}

// Enum example
enum Day {
   MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

Day day = Day.TUESDAY;
switch (day) {
    case  MONDAY -> ":(";
    case  TUESDAY, WEDNESDAY, THURSDAY -> ":|";
    case  FRIDAY -> ":)";
    case  SATURDAY, SUNDAY -> ":D";
}

// --- or ---

enum Day {
   MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

Day day = Day.TUESDAY;
switch (day) {
    case  MONDAY -> ":(";
    case  default -> ":D";
}
```

#### Helpful NullPointerExceptions
Log more details in NullPointExceptions. For example:
```java
// In JDK < 14
Exception in thread "main" java.lang.NullPointerException
        at Unlucky.method(Unlucky.java:83)
// In Jdk >= 15
Exception in thread "main" java.lang.NullPointerException:
  Cannot invoke "org.w3c.dom.Node.getChildNodes()" because
  the return value of "org.w3c.dom.NodeList.item(int)" is null
        at Unlucky.method(Unlucky.java:83)
```

