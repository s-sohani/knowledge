
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
  For example `.getDayOfWeek` in Java 8's Date/Time return `java.time.DayOfWeek` but in `Joda Time` return int so use `var` in below example made conf
