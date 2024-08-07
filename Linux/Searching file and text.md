#### Using `find`

The `find` command searches for files and directories within the file system.

- **Basic Syntax**
    
    sh
    
    Copy code
    
    `find [path] [options] [expression]`
    
- **Find files by name**
    
    sh
    
    Copy code
    
    `find /path/to/search -name "filename"`
    
- **Find files by extension**
    
    sh
    
    Copy code
    
    `find /path/to/search -name "*.txt"`
    
- **Find files by size**
    
    sh
    
    Copy code
    
    `find /path/to/search -size +100M   # Files larger than 100 MB find /path/to/search -size -100k   # Files smaller than 100 KB`
    
- **Find files by type**
    
    sh
    
    Copy code
    
    `find /path/to/search -type f       # Regular files find /path/to/search -type d       # Directories find /path/to/search -type l       # Symbolic links`
    
- **Find files by modification time**
    
    sh
    
    Copy code
    
    `find /path/to/search -mtime -7     # Modified in the last 7 days find /path/to/search -mtime +30    # Modified more than 30 days ago`
    
- **Find files by access time**
    
    sh
    
    Copy code
    
    `find /path/to/search -atime -7     # Accessed in the last 7 days find /path/to/search -atime +30    # Accessed more than 30 days ago`
    
- **Find files by permissions**
    
    sh
    
    Copy code
    
    `find /path/to/search -perm 644     # Files with 644 permissions find /path/to/search -perm /u+x    # Files executable by the owner`
    
- **Execute a command on found files**
    
    sh
    
    Copy code
    
    `find /path/to/search -name "*.log" -exec rm {} \;    # Delete all .log files find /path/to/search -name "*.sh" -exec chmod +x {} \;  # Make all .sh files executable`
    

#### Using `locate`

The `locate` command searches for files in a prebuilt database, making it faster than `find`.

- **Basic Syntax**
    
    sh
    
    Copy code
    
    `locate [options] pattern`
    
- **Find files by name**
    
    sh
    
    Copy code
    
    `locate filename`
    
- **Find files by partial name or extension**
    
    sh
    
    Copy code
    
    `locate "*.txt"`
    
- **Update the database (run as root or with sudo)**
    
    sh
    
    Copy code
    
    `updatedb`
    

#### Using `which`

The `which` command shows the path of the executable for a given command.

- **Find the path of a command**
    
    sh
    
    Copy code
    
    `which command`
    

#### Using `whereis`

The `whereis` command locates the binary, source, and manual page files for a command.

- **Find the binary, source, and man page for a command**
    
    sh
    
    Copy code
    
    `whereis command`
    

#### Using `type`

The `type` command describes how a given command would be interpreted if used.

- **Find information about a command**
    
    sh
    
    Copy code
    
    `type command`
    

#### Using `grep`

The `grep` command searches inside files for a given pattern.

- **Basic Syntax**
    
    sh
    
    Copy code
    
    `grep [options] pattern [file...]`
    
- **Search for a pattern within files**
    
    sh
    
    Copy code
    
    `grep "pattern" file.txt`
    
- **Search recursively within a directory**
    
    sh
    
    Copy code
    
    `grep -r "pattern" /path/to/dir`
    
- **Search for a pattern and display line numbers**
    
    sh
    
    Copy code
    
    `grep -n "pattern" file.txt`
    
- **Search for a pattern, ignoring case**
    
    sh
    
    Copy code
    
    `grep -i "pattern" file.txt`
    
- **Search for a whole word**
    
    sh
    
    Copy code
    
    `grep -w "word" file.txt`
    
- **Search for multiple patterns**
    
    sh
    
    Copy code
    
    `grep -e "pattern1" -e "pattern2" file.txt`
    

#### Using `awk`

The `awk` command is a powerful text processing tool that can be used for searching as well.

- **Search for a pattern and print the matching lines**
    
    sh
    
    Copy code
    
    `awk '/pattern/ {print $0}' file.txt`
    
- **Search for a pattern and print specific fields**
    
    sh
    
    Copy code
    
    `awk '/pattern/ {print $1, $3}' file.txt`
    

#### Practical Examples

- **Find all `.conf` files modified in the last 7 days in `/etc`**
    
    sh
    
    Copy code
    
    `find /etc -name "*.conf" -mtime -7`
    
- **Locate the `bash` binary**
    
    sh
    
    Copy code
    
    `which bash`
    
- **Find the binary, source, and man page for `ls`**
    
    sh
    
    Copy code
    
    `whereis ls`
    
- **Search for the string "error" in all `.log` files within `/var/log`**
    
    sh
    
    Copy code
    
    `grep "error" /var/log/*.log`
    
- **List all files containing the word "config" in their name**
    
    sh
    
    Copy code
    
    `locate config`