
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


