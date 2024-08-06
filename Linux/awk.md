`awk` is a powerful text-processing language that is used for pattern scanning and processing. It is named after its creators: Alfred Aho, Peter Weinberger, and Brian Kernighan.

## Basic Syntax

bash

Copy code

`awk 'pattern { action }' input_file`

- `pattern` specifies a condition.
- `action` specifies what to do when the pattern matches.

## Running `awk`

### From a File

bash

Copy code

`awk 'pattern { action }' file.txt`

### From a Command

bash

Copy code

`echo "text" | awk 'pattern { action }'`

## Basic Examples

### Print Entire File

bash

Copy code

`awk '{ print }' file.txt`

### Print Specific Column

Print the first column of each line:

bash

Copy code

`awk '{ print $1 }' file.txt`

Print the first and third columns:

bash

Copy code

`awk '{ print $1, $3 }' file.txt`

### Print Lines Matching a Pattern

Print lines containing "pattern":

bash

Copy code

`awk '/pattern/ { print }' file.txt`

Print lines where the first column matches "pattern":

bash

Copy code

`awk '$1 == "pattern" { print }' file.txt`

## Field Separators

### Default Separator (Whitespace)

bash

Copy code

`awk '{ print $1 }' file.txt`

### Custom Separator

Using a comma as a field separator:

bash

Copy code

`awk -F, '{ print $1 }' file.csv`

## Built-in Variables

- `NR`: Current record number (line number)
- `NF`: Number of fields in the current record
- `$0`: Entire input record
- `$1, $2, ...`: Individual fields

### Examples

Print line number and line:

bash

Copy code

`awk '{ print NR, $0 }' file.txt`

Print the number of fields in each line:

bash

Copy code

`awk '{ print NF }' file.txt`

## Arithmetic Operations

### Basic Arithmetic

Add two columns:

bash

Copy code

`awk '{ print $1 + $2 }' file.txt`

Multiply two columns:

bash

Copy code

`awk '{ print $1 * $2 }' file.txt`

## Conditional Statements

### If-Else

bash

Copy code

`awk '{ if ($1 > 10) print $1; else print "Too small" }' file.txt`

### Multiple Conditions

bash

Copy code

`awk '{ if ($1 > 10 && $2 < 20) print $1, $2 }' file.txt`

## Loops

### While Loop

bash

Copy code

`awk '{ i = 1; while (i <= NF) { print $i; i++ } }' file.txt`

### For Loop

bash

Copy code

`awk '{ for (i = 1; i <= NF; i++) print $i }' file.txt`

## Functions

### Length of a String

bash

Copy code

`awk '{ print length($1) }' file.txt`

### Substring

bash

Copy code

`awk '{ print substr($1, 2, 3) }' file.txt`

### String Concatenation

bash

Copy code

`awk '{ print $1 " " $2 }' file.txt`

### Mathematical Functions

bash

Copy code

`awk '{ print sqrt($1), sin($2) }' file.txt`

## Using BEGIN and END Blocks

### BEGIN Block

bash

Copy code

`awk 'BEGIN { print "Start Processing" } { print $1 }' file.txt`

### END Block

bash

Copy code

`awk '{ print $1 } END { print "End Processing" }' file.txt`

### Combined Example

bash

Copy code

`awk 'BEGIN { print "Start" } { print $1 } END { print "End" }' file.txt`

## Redirecting Output

### To a File

bash

Copy code

`awk '{ print $1 }' file.txt > output.txt`

### Append to a File

bash

Copy code

`awk '{ print $1 }' file.txt >> output.txt`

## Miscellaneous

### Ignore Case Sensitivity

bash

Copy code

`awk 'BEGIN { IGNORECASE = 1 } /pattern/ { print }' file.txt`

### Count Lines Matching a Pattern

bash

Copy code

`awk '/pattern/ { count++ } END { print count }' file.txt`

### Print Specific Lines

Print lines 1 to 3:

bash

Copy code

`awk 'NR==1, NR==3 { print }' file.txt`

### Print Last Line

bash

Copy code

`awk 'END { print }' file.txt`