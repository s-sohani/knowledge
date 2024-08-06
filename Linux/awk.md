`awk` is a powerful text-processing language that is used for pattern scanning and processing. It is named after its creators: Alfred Aho, Peter Weinberger, and Brian Kernighan.

## Basic Syntax

```
awk 'pattern { action }' input_file
```

- `pattern` specifies a condition.
- `action` specifies what to do when the pattern matches.

## Running `awk`

### From a File
`
awk 'pattern { action }' file.txt
`

### From a Command
`echo "text" | awk 'pattern { action }'`

## Basic Examples

### Print Entire File
`awk '{ print }' file.txt`

### Print Specific Column

Print the first column of each line:
`awk '{ print $1 }' file.txt`

Print the first and third columns:
`awk '{ print $1, $3 }' file.txt`

### Print Lines Matching a Pattern

Print lines containing "pattern":
`awk '/pattern/ { print }' file.txt`

Print lines where the first column matches "pattern":
`awk '$1 == "pattern" { print }' file.txt`

## Field Separators

### Default Separator (Whitespace)
`awk '{ print $1 }' file.txt`

### Custom Separator

Using a comma as a field separator:
`awk -F, '{ print $1 }' file.csv`

## Built-in Variables

- `NR`: Current record number (line number)
- `NF`: Number of fields in the current record
- `$0`: Entire input record
- `$1, $2, ...`: Individual fields

### Examples

Print line number and line:
`awk '{ print NR, $0 }' file.txt`

Print the number of fields in each line:
`awk '{ print NF }' file.txt`

## Arithmetic Operations

### Basic Arithmetic
`awk '{ print $1 + $2 }' file.txt`

Multiply two columns:
`awk '{ print $1 * $2 }' file.txt`

## Conditional Statements

### If-Else
`awk '{ if ($1 > 10) print $1; else print "Too small" }' file.txt`

### Multiple Conditions
`awk '{ if ($1 > 10 && $2 < 20) print $1, $2 }' file.txt`

## Loops

### While Loop
`awk '{ i = 1; while (i <= NF) { print $i; i++ } }' file.txt`

### For Loop
`awk '{ for (i = 1; i <= NF; i++) print $i }' file.txt`

## Functions

### Length of a String
`awk '{ print length($1) }' file.txt`

### Substring
`awk '{ print substr($1, 2, 3) }' file.txt`

### String Concatenation
`awk '{ print $1 " " $2 }' file.txt`

### Mathematical Functions
`awk '{ print sqrt($1), sin($2) }' file.txt`

## Using BEGIN and END Blocks

### BEGIN Block
`awk 'BEGIN { print "Start Processing" } { print $1 }' file.txt`

### END Block
`awk '{ print $1 } END { print "End Processing" }' file.txt`

### Combined Example
`awk 'BEGIN { print "Start" } { print $1 } END { print "End" }' file.txt`

## Redirecting Output

### To a File
`awk '{ print $1 }' file.txt > output.txt`

### Append to a File
`awk '{ print $1 }' file.txt >> output.txt`

## Miscellaneous

### Ignore Case Sensitivity
`awk 'BEGIN { IGNORECASE = 1 } /pattern/ { print }' file.txt`

### Count Lines Matching a Pattern
`awk '/pattern/ { count++ } END { print count }' file.txt`

### Print Specific Lines
Print lines 1 to 3:
`awk 'NR==1, NR==3 { print }' file.txt`

### Print Last Line
`awk 'END { print }' file.txt`