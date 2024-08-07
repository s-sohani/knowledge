`grep` is a powerful command-line utility in Ubuntu (and other Unix-like systems) for searching plain-text data for lines that match a regular expression.
#### Basic Usage
`grep [OPTIONS] PATTERN [FILE...]`

#### Basic Examples
`grep "search_string" file.txt       # Search for "search_string" in file.txt grep "search_string" file1 file2    # Search for "search_string" in multiple files echo "search_string" | grep "pattern"  # Search for "pattern" in piped input`

#### Common Options

- **-i**: Ignore case (case-insensitive)
    `grep -i "search_string" file.txt`
    
- **-v**: Invert match (select non-matching lines)
    `grep -v "search_string" file.txt`
    
- **-c**: Count the number of matching lines
    
    sh
    
    Copy code
    
    `grep -c "search_string" file.txt`
    
- **-l**: Show files containing the search string
    
    sh
    
    Copy code
    
    `grep -l "search_string" file1 file2`
    
- **-L**: Show files that do not contain the search string
    
    sh
    
    Copy code
    
    `grep -L "search_string" file1 file2`
    
- **-n**: Show line numbers of matching lines
    
    sh
    
    Copy code
    
    `grep -n "search_string" file.txt`
    
- **-H**: Print the filename for each match (default when multiple files are searched)
    
    sh
    
    Copy code
    
    `grep -H "search_string" file1 file2`
    
- **-r** or **-R**: Recursively search directories
    
    sh
    
    Copy code
    
    `grep -r "search_string" /path/to/dir`
    
- **-w**: Match whole words only
    
    sh
    
    Copy code
    
    `grep -w "search_string" file.txt`
    
- **-x**: Match whole lines only
    
    sh
    
    Copy code
    
    `grep -x "search_string" file.txt`
    
- **--color**: Highlight matching strings
    
    sh
    
    Copy code
    
    `grep --color "search_string" file.txt`
    

#### Regular Expressions

- **.**: Match any single character
    
    sh
    
    Copy code
    
    `grep "s.rch" file.txt  # Matches "search", "sirch", etc.`
    
- *****: Match zero or more of the preceding character
    
    sh
    
    Copy code
    
    `grep "s.*ch" file.txt  # Matches "sch", "search", "saabbch", etc.`
    
- **^**: Match the start of a line
    
    sh
    
    Copy code
    
    `grep "^search" file.txt  # Matches lines that start with "search"`
    
- **$**: Match the end of a line
    
    sh
    
    Copy code
    
    `grep "search$" file.txt  # Matches lines that end with "search"`
    
- **[ ]**: Match any one of the enclosed characters
    
    sh
    
    Copy code
    
    `grep "s[aeiou]rch" file.txt  # Matches "sarch", "serch", "sirch", etc.`
    
- **[^ ]**: Match any character not enclosed
    
    sh
    
    Copy code
    
    `grep "s[^aeiou]rch" file.txt  # Matches "sbarch", "srch", but not "sarch"`
    
- **{n,m}**: Match the preceding element at least n times but not more than m times
    
    sh
    
    Copy code
    
    `grep "a\{2,4\}" file.txt  # Matches "aa", "aaa", "aaaa"`
    

#### Advanced Usage

- **-A NUM**: Print NUM lines of trailing context after matching lines
    
    sh
    
    Copy code
    
    `grep -A 3 "search_string" file.txt`
    
- **-B NUM**: Print NUM lines of leading context before matching lines
    
    sh
    
    Copy code
    
    `grep -B 3 "search_string" file.txt`
    
- **-C NUM**: Print NUM lines of context around matching lines
    
    sh
    
    Copy code
    
    `grep -C 3 "search_string" file.txt`
    
- **--include=*PATTERN***: Search only files matching PATTERN
    
    sh
    
    Copy code
    
    `grep --include=\*.txt "search_string" /path/to/dir/*`
    
- **--exclude=*PATTERN***: Exclude files matching PATTERN from the search
    
    sh
    
    Copy code
    
    `grep --exclude=\*.log "search_string" /path/to/dir/*`
    
- **--exclude-dir=*PATTERN***: Exclude directories matching PATTERN from the search
    
    sh
    
    Copy code
    
    `grep --exclude-dir=\*.git "search_string" /path/to/dir/*`
    
- **-e**: Specify multiple patterns
    
    sh
    
    Copy code
    
    `grep -e "pattern1" -e "pattern2" file.txt`
    
- **-f FILE**: Get patterns from a file
    
    sh
    
    Copy code
    
    `grep -f patterns.txt file.txt`
    
- **-q**: Quiet mode (suppress output, only return exit status)
    
    sh
    
    Copy code
    
    `grep -q "search_string" file.txt`
    
- **-s**: Suppress error messages about nonexistent or unreadable files
    
    sh
    
    Copy code
    
    `grep -s "search_string" file.txt`
    

#### Practical Examples

- Search for a string in all `.txt` files in the current directory:
    
    sh
    
    Copy code
    
    `grep "search_string" *.txt`
    
- Search recursively for a string in all files in a directory, ignoring case:
    
    sh
    
    Copy code
    
    `grep -ri "search_string" /path/to/dir`
    
- Find all lines in a file that do not contain a specific string:
    
    sh
    
    Copy code
    
    `grep -v "search_string" file.txt`
    
- Count the occurrences of a string in a file:
    
    sh
    
    Copy code
    
    `grep -o "search_string" file.txt | wc -l`
    
- Find lines matching a regex pattern in all `.log` files, showing 2 lines of context:
    
    sh
    
    Copy code
    
    `grep -C 2 "error.*failed" /var/log/*.log`