
>Refrences
>https://advancedweb.hu/new-language-features-since-java-8-to-21/
>https://advancedweb.hu/a-categorized-list-of-all-java-and-jvm-features-since-jdk-8-to-21/


## JKD9
#### Allow private methods in interfaces

It just for tidy up the interface and increase encapsulation and resusability of code.

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

