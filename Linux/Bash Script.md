
# Shebang
Tells the system which interpreter to use.
```bash
#!/bin/bash
```

# Comments
```bash
# This is a single-line comment 

: ' 
This is a 
multi-line comment 
'
```

# Variables
```bash
# Define a variable
NAME="John"

# Use a variable
echo "Hello, $NAME"

```

# Special Variables
```bash
$0  # Script name
$1  # First argument
$2  # Second argument
$#  # Number of arguments
$@  # All arguments
$?  # Exit status of the last command
$$  # Process ID of the current shell
```

# Echo
```bash
echo "Hello, World!"
```

# Read
```bash
echo "Enter your name:"
read NAME
echo "Hello, $NAME"
```

# Redirecting Output
```bash
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
```bash
# Send the output of one command as input to another
ls | grep "myfile"
```

# If-Else
```bash
if [ "$NAME" == "John" ]; then
    echo "Hello, John"
else
    echo "You are not John"
fi
```

# Elif
```bash
if [ "$NAME" == "John" ]; then
    echo "Hello, John"
elif [ "$NAME" == "Jane" ]; then
    echo "Hello, Jane"
else
    echo "You are not John or Jane"
fi
```

# Case
```bash
case "$NAME" in
  "John")
    echo "Hello, John"
    ;;
  "Jane")
    echo "Hello, Jane"
    ;;
  *)
    echo "You are not John or Jane"
    ;;
esac
```

# For Loop
```bash
for i in 1 2 3 4 5; do
    echo "Number: $i"
done

# C-style for loop
for ((i = 1; i <= 5; i++)); do
    echo "Number: $i"
done
```

# While Loop
```bash
count=1
while [ $count -le 5 ]; do
    echo "Number: $count"
    ((count++))
done
```

# Until Loop
```bash
count=1
until [ $count -gt 5 ]; do
    echo "Number: $count"
    ((count++))
done
```

# Defining and Calling Functions
```bash
# Define a function
greet() {
    echo "Hello, $1"
}

# Call a function
greet "John"
```

# Returning Values
```bash
# Define a function with a return value
add() {
    return $(($1 + $2))
}

# Call the function
add 3 5
result=$?
echo "Result: $result"
```

# Using Local Variables
```bash
my_function() {
    local VAR="local variable"
    echo "$VAR"
}
my_function
echo "$VAR"  # This will be empty because VAR is local to the function
```

# Checking if a File Exists
```bash
if [ -f "filename.txt" ]; then
    echo "File exists"
else
    echo "File does not exist"
fi
```

# Checking if a Directory Exists
```bash
if [ -d "dirname" ]; then
    echo "Directory exists"
else
    echo "Directory does not exist"
fi
```

# Reading a File Line by Line
```bash
while IFS= read -r line; do
    echo "$line"
done < "filename.txt"
```

# Exit on Error
```bash
set -e  # Exit immediately if a command exits with a non-zero status
```

# Trap
```bash
# Run a command when the script exits
trap 'echo "Script exited with status $?"' EXIT
```

# Parsing Arguments
```bash
while [ "$1" != "" ]; do
    case $1 in
        -a | --arg1 )         shift
                              ARG1=$1
                              ;;
        -b | --arg2 )         shift
                              ARG2=$1
                              ;;
        -h | --help )         usage
                              exit
                              ;;
        * )                   usage
                              exit 1
    esac
    shift
done
```

# Usage Function
```bash
usage() {
    echo "Usage: $0 [-a arg1] [-b arg2]"
}
```

# Enabling Debugging
```bash
set -x  # Print commands and their arguments as they are executed
```

# Disabling Debugging
```bash
set +x  # Turn off debugging
```

# Examples
```bash
#!/bin/bash

### Simple Backup Script

SOURCE="/path/to/source"
DEST="/path/to/dest"
DATE=$(date +%Y-%m-%d)
BACKUP_NAME="backup-$DATE.tar.gz"

tar -czf "$DEST/$BACKUP_NAME" "$SOURCE"
echo "Backup completed: $DEST/$BACKUP_NAME"
```

```bash
#!/bin/bash

### User Input and Conditional Logic

echo "Enter your age:"
read AGE

if [ "$AGE" -ge 18 ]; then
    echo "You are an adult."
else
    echo "You are a minor."
fi
```

```bash
#!/bin/bash

### Looping Through Files

for file in /path/to/files/*; do
    echo "Processing $file"
    # Perform some action on $file
done
```

```bash
#!/bin/bash

### Function with Argument Parsing

greet() {
    local NAME=$1
    echo "Hello, $NAME"
}

while [ "$1" != "" ]; do
    case $1 in
        -n | --name )         shift
                              NAME=$1
                              ;;
        * )                   echo "Invalid option"
                              exit 1
    esac
    shift
done

greet "$NAME"
```