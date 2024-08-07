`sed` (stream editor) is a powerful text-processing tool used to perform basic text transformations on an input stream.
#### Basic Usage
```bash
sed [OPTIONS] 'script' [input-file]
```

#### Basic Commands

- **p**: Print lines
    ```bash
    sed -n 'p' file.txt  # Print all lines 
    sed -n '5p' file.txt  # Print the 5th line 
    sed -n '5,10p' file.txt  # Print lines from 5 to 10
    ```
    
- **d**: Delete lines
    ```bash
    sed 'd' file.txt  # Delete all lines (empty output) 
    sed '5d' file.txt  # Delete the 5th line 
    sed '5,10d' file.txt  # Delete lines from 5 to 10 
    ```
    
- **s**: Substitute/replace text
    ```bash
    sed 's/old/new/' file.txt  # Replace the first occurrence of 'old' with 'new' on each line 
    sed 's/old/new/g' file.txt  # Replace all occurrences of 'old' with 'new' on each line 
    sed 's/old/new/2' file.txt  # Replace the second occurrence of 'old' with 'new' on each line
    ```
    
- **i**: Insert text before a line
    ```bash
    sed '5i\New line of text' file.txt  # Insert text before the 5th line
    ```
    
- **a**: Append text after a line
    ```bash
    sed '5a\New line of text' file.txt  # Append text after the 5th line
    ```
    
- **c**: Change lines
    ```bash
    sed '5c\New line of text' file.txt  # Replace the 5th line with new text
    ```
    

#### Addressing

- **Number**: Specific line number
    ```bash
    sed '3d' file.txt  # Delete the 3rd line
    ```
    
- **$**: Last line
    ```bash
    sed '$d' file.txt  # Delete the last line
    ```
    
- **/pattern/**: Lines matching a pattern
    ```bash
    sed '/pattern/d' file.txt  # Delete lines matching 'pattern'
    ```
    
- **Range**: From one line to another
    ```bash
    sed '3,5d' file.txt  # Delete lines from 3 to 5 sed '/start/,/end/d' file.txt  # Delete lines from 'start' to 'end'
    ```
    
- **First and last occurrence**:
    ```bash
    sed '0,/pattern/d' file.txt  # Delete from start to first occurrence of 'pattern' 
    sed '/pattern/,$d' file.txt  # Delete from first occurrence of 'pattern' to end
    ```
    

#### Flags

- **-n**: Suppress automatic printing of pattern space
    ```bash
    sed -n 'p' file.txt  # Print all lines (same as 'cat file.txt')
    ```
    
- **-e**: Script to be executed
    ```bash
    sed -e 's/old/new/' -e 's/foo/bar/' file.txt  # Multiple scripts
    ```
    
- **-f**: Read script from file
    ```bash
    sed -f script.sed file.txt
    ```
    
- **-i**: Edit files in place
    ```bash
    sed -i 's/old/new/g' file.txt  # Edit file.txt in place 
    sed -i.bak 's/old/new/g' file.txt  # Edit in place and create a backup
    ```
    

#### Advanced Commands

- **y**: Transform (replace characters)
    ```bash
    sed 'y/abc/ABC/' file.txt  # Convert 'a' to 'A', 'b' to 'B', 'c' to 'C'
    ```
    
- **&**: The matched string
    ```bash
    sed 's/pattern/& and more/' file.txt  # Replace 'pattern' with 'pattern and more'
    ```
    
- **\1, \2, ...**: Refer to matched groups
    ```bash
    sed 's/\(pattern\)/\1 and more/' file.txt  # Same as above but with grouping
    ```
    
- **g**: Global replacement
    ```bash
    sed 's/old/new/g' file.txt  # Replace all occurrences on each line
    ```
    
- **w FILE**: Write to a file
    ```bash
    sed -n 's/pattern/&/w output.txt' file.txt  # Write matching lines to output.txt
    ```
    
- **r FILE**: Read from a file
    ```bash
    sed '5r input.txt' file.txt  # Append content of input.txt after 5th line
    ```
    
- **e**: Execute command
    ```bash
    sed 's/old/new/e' file.txt  # Execute command after substitution
    ```
    

#### Practical Examples

- Substitute 'foo' with 'bar' on specific lines:
    ```bash
    sed '2,4s/foo/bar/' file.txt  # Lines 2 to 4 sed '/pattern/s/foo/bar/' file.txt  # Lines matching 'pattern'
    ```
    
- Insert a line after every line matching a pattern:
    ```bash
    sed '/pattern/a\New line of text' file.txt
    ```
    
- Print lines that match a pattern:
    ```bash
    sed -n '/pattern/p' file.txt
    ```
    
- Delete lines that do not match a pattern:
    ```bash
    sed '/pattern/!d' file.txt
    ```
    
- Print only lines containing 'foo':
    ```bash
    sed -n '/foo/p' file.txt
    ```
