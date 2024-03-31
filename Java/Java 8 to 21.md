
>Refrences
>https://advancedweb.hu/new-language-features-since-java-8-to-21/
>https://advancedweb.hu/a-categorized-list-of-all-java-and-jvm-features-since-jdk-8-to-21/


## JKD 11
### Allow private methods in interfaces
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


### Diamond operator for anonymous inner classes
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

### Allow effectively-final variables to be used as resources in try-with-resources statements
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

### Underscore is no longer a valid identifier name
This block of code is invalid in Java9, but allowed and convey special meaning in Java21.

```java
int _ = 10;
```

### Type Inference
Use `var` instead of explicit type, It makes this piece of code less redundant, thus, easier to read.
```java
var greetingMessage = "Hello!";
```
The type of the declared variables is **inferred at compile time**

#### Maybe use `var` has backfire
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

### Switch Expressions
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

#### Differences between Switch Statement and Switch Expression
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

### Helpful NullPointerExceptions
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

### Text Blocks
Use triple quout to write multiple line string. for example:
```java
String html = "";
html += "<html>\n";
html += "  <body>\n";
html += "    <p>Hello, world</p>\n";
html += "  </body>\n";
html += "</html>\n";

// also blow same result
String html = """
          <html>
            <body>
              <p>Hello, world</p>
            </body>
          </html>
          """;
//  remove new line using \
String singleLine = """
          Hello \
          World
          """;
```

The compiler checks the whitespace used for indentation in each line to find the least indented line, and shifts each line to the left by this minimal common indentation.
This means that if the closing `"""` is in a separate line, the indentation can be increased by shifting the closing token to the left.
The opening `"""` does not count for the indentation removal so it's not necessary to line up the text block with it.
```java
String noIndentation = """
          First line
          Second line
          """;

String indentedByToSpaces = """
          First line
          Second line
        """;
        
// Below results are the same
String indentedByToSpaces = """
         First line
         Second line
       """;

String indentedByToSpaces = """
                              First line
                              Second line
                            """;
```

The `String` class also provides some programmatic ways to deal with indentation.
- The `indent` method takes an integer and returns a new string with the specified levels of additional indentation
- `stripIndent` returns the contents of the original string without all the incidental indentation.

#### Some Tips
- Text Block new line contain `\n`, If you want open file include Text Block content, you see single line, so **Make sure you have correct control on characters**. For example replace all `\n` with `\n\r`.
- Preserve trailing space Because in Text Block spaces are ignored. If a line end with spaces or tab, use `\t` or `\s`. 
- Text Block compatible with `String::formatted` or ``String::format`.
```java
var name = "world";
var greeting = """
    hello
    %s
    """.formatted(name);
```

### Pattern Matching for instanceof
If test of `instanceof` be passed, we can declared pattern variable.
```java
// JDK < 16
if (obj instanceof String) {
    String s = (String) obj;
    // use s
}

// JDK >= 16
if (obj instanceof String s) {
    // use s
}

// Pattern Matchin with complex condition
if (obj instanceof String s && s.length() > 5) {
  // use s
}

// User Pattern variable in every where in block.
private static int getLength(Object obj) {
  if (!(obj instanceof String s)) {
    throw new IllegalArgumentException();
  }

  // s is in scope - if the instanceof does not match
  //      the execution will not reach this statement
  return s.length();
}
```


### Record Classes
Define immutable data classes. **Record Classes are only about the data they carry** without providing too much customization options.
```java
public record Point(int x, int y) { }
```
Record Classes **can't extend other classes**, they **can't declare native methods**, and they are **implicitly final** and **can't be abstract**.
Fields of a Record Class are not only `final` by default, it's **not even possible to have any non-final fields**.
**Supplying data** to a record is only possible through its constructor.

```java
public record Point(int x, int y) {
  public Point {
    if (x < 0) {
      throw new IllegalArgumentException("x can't be negative");
    }
    if (y < 0) {
      y = 0;
    }
  }
}
```

#### Tipes
- Use Local Records to model intermediate transformations
  A typical solution was to rely on `Pair` or similar holder classes from a library, or to define your own.
```java
public List<Product> findProductsWithMostSaving(List<Product> products) {
  record ProductWithSaving(Product product, double savingInEur) {}

  products.stream()
    .map(p -> new ProductWithSaving(p, p.basePriceInEur * p.discountPercentage))
    .sorted((p1, p2) -> Double.compare(p2.savingInEur, p1.savingInEur))
    .map(ProductWithSaving::product)
    .limit(5)
    .collect(Collectors.toList());
}
```
- Record class not compatible with bean, spring data and some features in Jackson and other libs so far. 

### Sealed Classes
- Alternative for final class. 
- Restrict which other classes or interfaces may extend or implement them.
- Allowing the authors to explicitly list the subclasses.
```java
public sealed class Shape
    permits Circle, Quadrilateral {...}
```
- Permitted classes must be located in the same package as the superclass.
- Authors are forced to always explicitly define the boundaries of a sealed type hierarchy.
	- `final`: the subclass can not be extended at all
	- `sealed`: the subclass can only be extended by some permitted classes
	- `non-sealed`: the subclass can be freely extended
```java
public sealed class Shape
    permits Circle, Quadrilateral, WeirdShape {...}

public final class Circle extends Shape {...}

public sealed class Quadrilateral extends Shape
    permits Rectangle, Parallelogram {...}
public final class Rectangle extends Quadrilateral {...}
public final class Parallelogram extends Quadrilateral {...}

public non-sealed class WeirdShape extends Shape {...}
```
- Declare all of Classes in the same source file in which case the `permits` clause can be omitted.
```java
public sealed class Shape {
  public final class Circle extends Shape {}

  public sealed class Quadrilateral extends Shape {
    public final class Rectangle extends Quadrilateral {}
    public final class Parallelogram extends Quadrilateral {}
  }

  public non-sealed class WeirdShape extends Shape {}
}
```
- Record classes can also be part of a sealed hierarchy as leafs because they are implicitly final.
- Permitted classes must be located in the same package as the superclass.
- **Sealed classes** offer a nice alternative to _Enum types_ making it possible to use regular classes to model the fixed alternatives.

### Unnamed Class and Instance main method (Preview)
- Define `main` method on unnamed package, module or class.
- Define not `static` `main` method.
- The `string[]` parameter can also be ommited.
Unnamed classes work similarly to _unnamed packages_ and _unnamed modules_. If a class does not have a `package` declaration, it will be part of the unnamed package, in which case they can not be referenced by classes from named packages. If a package is not part of a module, it will be part of the unnamed module, so packages from other modules can't refer them.

## JDK 21

### Record Patterns
Pattern matchin used for `instanceof` and `switch`, now is used for `Record` pattern.
```java
// Pattern matching with a type pattern using instanceof
if (obj instanceof String s) {
  // ... use s ...
}

// Pattern matching with a type pattern using switch
switch (obj) {
    case String s -> // ... use s ...
    // ... other cases ...
};

// Pattern matching with a Record pattern
interface Point { }
record Point2D(int x, int y) implements Point { }
enum Color { RED, GREEN, BLUE }
record ColoredPoint(Point p, Color c) { }

Object r = new ColoredPoint(new Point2D(3, 4), Color.GREEN);

// Nested pattern matching or record pattern
if (r instanceof ColoredPoint(Point2D(int x, int y), Color c)) {
  // work with x, y, and c
}

var length = switch (r) {
	case ColoredPoint(Point2D(int x, int y), Color c) -> Math.sqrt(x*x + y*y);
	case ColoredPoint(Point p, Color c) -> 0;
}
```


