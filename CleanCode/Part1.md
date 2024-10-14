## meaning full name


**1. Accurate Naming Conventions:**

- Retain complete information in variable names. For example, use a descriptive list name like `days_of_week` instead of vague names such as `daysinweek`.
- Correct Example: `days_of_week = ['Saturday', 'Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday']`
- Incorrect Example: `nums = list('Saturday', ...)`

**2. Meaningful and Pronounceable Identifiers:**

- Avoid ambiguous or "noisy" words like "data", "info", or "manager" which do not add value.
- Instead of generic terms, use specific descriptors that clearly define the purpose of the variable:
    - `user_record`: This should refer to records retrieved from a database.
    - `user_credentials`: Use this to refer specifically to authentication-related information.
    - `user_attributes`: Use to describe properties or characteristics of a user.
    - `user_profile`: Information that encompasses aspects of a user's profile.

**3. Pronounceable and Understandable Abbreviations:**

- Abbreviations should be intuitive and easily pronounceable to prevent confusion.
- Example: Rename `genYmdHms` to `generate_year_month_day_hour_minute_second` for clarity.
- Avoid ambiguous abbreviations like `prm` which could mean 'parameter', 'premium', or 'preeminent'. Always choose clarity over brevity.

**4. Consistency with Language Conventions:**

- Follow language-specific naming conventions to enhance code readability and maintainability:
    - Python: Use PEP 8 guidelines with `snake_case` for variables and functions, e.g., `user_profile`.
    - .NET: Use `CamelCase` for naming, e.g., `userProfile`.

**5. Class and Method Naming:**

- Class names should be nouns or noun phrases that clearly represent what the class is about, such as `UserSaver` or `UserProfilePoster`.
- Method names should be verbs that describe the action the method performs, like `save_user_records`.

**6. Consistency in Terminology:**

- Use a consistent term for similar concepts throughout your code to avoid confusion.
- Choose a single term for each action: if you use `delete` for removing items, avoid using synonyms like `destroy` or `remove` elsewhere in your code.

**7. Functionality and Method Examples:**

- When creating functions, clearly define what each function does and ensure the naming reflects this action. For instance, a function named `Add()` should not be used ambiguously between adding an instance and incrementing a value.



### Writing Clean and Efficient Functions in Python



## 1. Small Functions

- **Keep functions short**: A function should ideally do one thing and do it well. This makes it easier to read, understand, and maintain.
- **Single Responsibility Principle (SRP)**: A function should have one reason to change, meaning it should focus on a single task.

For example:

```python
def process_order(order):
    validate_order(order)
    calculate_shipping(order)
    send_confirmation_email(order)
```



By splitting responsibilities into smaller functions, you improve clarity and modularity.

## 2. Descriptive Names
Function names should be self-explanatory and reflect what they do. The reader should understand the purpose of the function without needing to look at the code.
For example:

Instead of naming a function post_data, use a more descriptive name like send_entity_data_to_blockchain_service.

## 3. Fewer Arguments
Functions should have few parameters. Ideally, a function should take zero, one, or two arguments. More than three parameters may indicate that a function is doing too much.
If a function requires many parameters, consider wrapping them in a class or data structure.


## 4. Avoid Side Effects
Functions should not have hidden side effects, like modifying global variables or unexpectedly changing the state of an object.
A function should either return a value or modify a state, but not both.
Example of a function with side effects:

```python
# Avoid this: modifying global state
def add_item_to_cart(item):
    cart.append(item)  # 'cart' is a global variable
    ```

A better approach:
```python
def add_item_to_cart(cart, item):
    return cart + [item]
    ```

## 5. Command-Query Separation
A command modifies data, sends a message, or updates the database.
A query answers a question or returns a value without altering the system.
Bad example: Mixing command and query logic.


```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance

    def transfer_funds(self, amount, target_account):
        if self.balance >= amount:  # Query: Checking balance
            self.balance -= amount  # Command: Modifying balance
            target_account.balance += amount  # Command: Modifying target account
            self.log_transaction(amount, target_account)  # Command: Logging
            return True  # Query: Returning status
        return False
        ```


Clean code separates commands and queries:


```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance

    # Query: Only checks if the user has sufficient funds
    def has_sufficient_funds(self, amount):
        return self.balance >= amount

    # Command: Only responsible for transferring funds
    def transfer_funds(self, amount, target_account):
        if not self.has_sufficient_funds(amount):
            raise ValueError("Insufficient funds")
        self.balance -= amount
        target_account.balance += amount

    # Command: Logs the transaction separately
    def log_transaction(self, amount, target_account):
        print(f"Transferred {amount} to {target_account}. New balance: {self.balance}")

def process_transaction(source_account, target_account, amount):
    if source_account.has_sufficient_funds(amount):  # Query
        source_account.transfer_funds(amount, target_account)  # Command
        source_account.log_transaction(amount, target_account)  # Command
    else:
        print("Transaction failed: Insufficient funds")
        ```


## 6. Avoid Boolean Parameters
Boolean parameters often signal that a function is trying to do too much. It's better to split the functionality into separate functions.
Bad example:

```python
def send_message(message, is_urgent):
    if is_urgent:
        send_urgent_message(message)
    else:
        send_normal_message(message)
        ```


Clean code:

```python
def send_urgent_message(message):
    # logic for urgent message

def send_normal_message(message):
    # logic for normal message
    ```

## 7. DRY (Don’t Repeat Yourself)
Avoid code duplication by refactoring repeated logic into a separate function. Duplication leads to harder maintenance.
Bad example:


```python
def calculate_price_with_tax(price):
    return price * 1.2

def calculate_discounted_price_with_tax(price, discount):
    discounted_price = price - (price * discount)
    return discounted_price * 1.2
    ```


Clean code:


```python
def apply_tax(price):
    return price * 1.2

def calculate_discounted_price(price, discount):
    discounted_price = price - (price * discount)
    return apply_tax(discounted_price)
    ```


## 8. Error Handling
Error handling should not obscure the core logic of the function. A function should focus on its main purpose, while error handling can be separated.

Bad example:
```python
def read_file(file_path):
    if not os.path.exists(file_path):
        raise FileNotFoundError("File does not exist")
    with open(file_path, 'r') as file:
        return file.read()
        ```


Clean code:
```python
def read_file(file_path):
    assert_file_exists(file_path)
    with open(file_path, 'r') as file:
        return file.read()

def assert_file_exists(file_path):
    if not os.path.exists(file_path):
        raise FileNotFoundError("File does not exist")
        ```



### comment

### **Avoid positional markers**
استفاده از نشانگرهای موقعیتی مانند «# شروع حلقه» یا «# پایان تابع» معمولاً فقط موجب شلوغی کد می‌شود. بهتر است از نام‌گذاری مناسب توابع و متغیرها، به همراه تورفتگی (indentation) و قالب‌بندی درست استفاده کنید تا ساختار بصری و خوانایی بهتری به کد داده شود.
```python
# Start of function
def calculate_area(radius):
    # Start of calculation
    pi = 3.14159
    area = pi * radius ** 2
    # End of calculation
    return area
# End of function

```
### **Don't leave commented out code in your codebase**
این کار باعث شلوغی و کاهش خوانایی کد می‌شود. اگر بخواهید کدی را در آینده بازیابی کنید، از سیستم کنترل نسخه (Version Control) مانند Git استفاده کنید که تاریخچه‌ی کامل کد شما را نگه می‌دارد.
```python
def calculate_area(radius):
    pi = 3.14159
    # old formula: return 3 * radius * radius
    return pi * radius ** 2

```
### **Don't have journal comments**
استفاده از **کامنت‌های ژورنالی** (Journal Comments) یکی از اشتباهات رایج در برنامه‌نویسی است. این نوع کامنت‌ها معمولاً برای ثبت تغییرات یا پیگیری نسخه‌های قبلی کد استفاده می‌شوند. در حالی که این کار در گذشته مرسوم بود، اما اکنون که ابزارهای کنترل نسخه مانند Git در دسترس هستند، نیازی به نگهداری چنین کامنت‌هایی نیست. تمامی تاریخچه و تغییرات کد شما به طور خودکار در Git ذخیره می‌شود و می‌توانید به راحتی آن‌ها را مشاهده و بازیابی کنید.
```javascript
/**
 * 2018-12-20: Removed monads, didn't understand them (RM)
 * 2017-10-01: Improved using special monads (JP)
 * 2016-02-03: Removed type-checking (LI)
 * 2015-03-14: Added combine with type-checking (JR)
 */
public int Combine(int a,int b)
{
    return a + b;
}
```


### **Only comment things that have business logic complexity**
```javascript
public int HashIt(string data)
{
    // The hash
    var hash = 0;

    // Length of string
    var length = data.length;

    // Loop through every character in data
    for (var i = 0; i < length; i++)
    {
        // Get character code.
        const char = data.charCodeAt(i);
        // Make the hash
        hash = ((hash << 5) - hash) + char;
        // Convert to 32-bit integer
        hash &= hash;
    }
}
```
good example
```javascript
public int Hash(string data)
{
    var hash = 0;
    var length = data.length;

    for (var i = 0; i < length; i++)
    {
        var character = data[i];
        // use of djb2 hash algorithm as it has a good compromise
        // between speed and low collision with a very simple implementation
        hash = ((hash << 5) - hash) + character;

        hash = ConvertTo32BitInt(hash);
    }
    return hash;
}

private int ConvertTo32BitInt(int value)
{
    return value & value;
}
```


#### Formating

```python
import math
def calc_dist(x1,y1,x2,y2): return math.sqrt((x2-x1)**2 + (y2-y1)**2)

```


```python
import math

def calculate_distance(x1, y1, x2, y2):
    """
    Calculate the Euclidean distance between two points (x1, y1) and (x2, y2) in a 2D space.
    """
    distance = math.sqrt((x2 - x1) ** 2 + (y2 - y1) ** 2)
    return distance

```

نمونه دوم با فاصله‌گذاری مناسب بین خطوط و پارامترها، و استفاده از داک‌استرینگ برای شرح عملکرد تابع، خوانایی و درک کد را بسیار بهبود می‌بخشد. تفکیک دستورات و توضیحات کد در نمونه دوم نه تنها آن را قابل فهم‌تر می‌کند، بلکه تسهیل در نگهداری و عیب‌یابی کد را نیز ممکن می‌سازد.