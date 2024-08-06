The pipe is a powerful tool in Unix-like operating systems, allowing you to pass the output of one command as the input to another.

# Basic Usage
```bash
command1 | command2
```

```bash
ls -l | grep "filename"
```

# View Long Outputs
```bash
ls -l | less
```

# Sorting Output
```bash
cat file.txt | sort
```

# Removing Duplicate Lines
```bash
cat file.txt | sort | uniq
```

# Counting Lines, Words, and Characters
```bash
cat file.txt | wc -l   # Count lines
cat file.txt | wc -w   # Count words
cat file.txt | wc -c   # Count characters
```

# Extracting Columns
```bash
cat file.txt | cut -d' ' -f1,2
```

# Replacing Text
```bash
echo "Hello World" | sed 's/World/Ubuntu/'
```

# Combining Multiple Commands
```bash
cat file.txt | grep "pattern" | sort | uniq -c | sort -nr
```

# Combining with xargs
When using large dataset or using complex script
```bash
cat file_list.txt | xargs -I {} cp {} /destination_directory/
```

```bash
cat pids.txt | xargs kill -9
```


