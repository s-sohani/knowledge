The pipe is a powerful tool in Unix-like operating systems, allowing you to pass the output of one command as the input to another.

Basic Usage
```
command1 | command2
```

```
ls -l | grep "filename"
```

View Long Outputs
```
ls -l | less
```

Sorting Output
```
cat file.txt | sort
```

Removing Duplicate Lines
```
cat file.txt | sort | uniq
```

Counting Lines, Words, and Characters
```
cat file.txt | wc -l   # Count lines
cat file.txt | wc -w   # Count words
cat file.txt | wc -c   # Count characters
```

Extracting Columns
```
cat file.txt | cut -d' ' -f1,2
```

Replacing Text
```
echo "Hello World" | sed 's/World/Ubuntu/'
```

Combining Multiple Commands
```
cat file.txt | grep "pattern" | sort | uniq -c | sort -nr
```

Combining with xargs
```
cat file_list.txt | xargs -I {} cp {} /destination_directory/
```

