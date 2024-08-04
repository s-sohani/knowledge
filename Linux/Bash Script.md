
# Shebang
Tells the system which interpreter to use.
```
#!/bin/bash
```

# Comments
```
# This is a single-line comment 

: ' 
This is a 
multi-line comment 
'
```

# Variables
```
# Define a variable
NAME="John"

# Use a variable
echo "Hello, $NAME"

```

# Special Variables
```
$0  # Script name
$1  # First argument
$2  # Second argument
$#  # Number of arguments
$@  # All arguments
$?  # Exit status of the last command
$$  # Process ID of the current shell
```

# Echo
```
echo "Hello, World!"
```

# Read
```
echo "Enter your name:"
read NAME
echo "Hello, $NAME"
```

# Redirecting Output
```
# Redirect standard output to a file
echo "Hello, World!" > output.txt

# Append standard output to a file
echo "Hello, again!" >> output.txt

# Redirect standard error to a file
ls non_existent_file 2> error.txt

# Redirect both standard output and error to a file
ls non_existent_file > all_output.txt 2>&1
```

# Pipes
```
# Send the output of one command as input to another
ls | grep "myfile"
```

# If-Else
```
if [ "$NAME" == "John" ]; then
    echo "Hello, John"
else
    echo "You are not John"
fi
```
