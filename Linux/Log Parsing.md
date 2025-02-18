# GREP
`GREP` Allow you to search pattern in files, `ZGREP` for GZIP files. 
```bash
grep <pattern> [options] file.log
```
- -n: Number of lines that matches. 
- -i: Case insensitive. 
- -v: Invert match.
- -E: Extended regex. 
- -c: Count number of matches. 
- -l: Find file names that match that pattern. 

# NGREP
`NGREP` is used for analyzing network packets. 
```bash
ngrep -I file.pcap
```
- -d: Specify network interface. 
- -i: Case insensitive. 
- -x: Print in alternate hexdump. 
- -t: print time stamp. 
- -I: Road pcap file. 

# CUT
The `CUT` command is used for parse fields from delimited logs. 
```bash
cut -d ":" -f 2 file.log
```
- -d: Used the field delimiter. 
- -f: The field numbers. 
- -c: Specifies characters position

# SED
`SED` is used to replace string in a file. 
```bash 
sed s/regex/replace/g
```
- s: Search 
- g: Replace
- d: Delete
- w: Append to file

# SORT
`SORT` is used to sort a file. 
```bash 
sort foo.txt
```
- -o: Output to file. 
- -r: Reverse order. 
- -n: Numerical Sort
- -k: Sort by column. 
- -c: Check if ordered. 
- -u: Sort and remove. 
- -f: Ignore case. 
- -h: Human sort. 

# UNIQ
`UNIQ` is used to extract uniq occurrences. 
```bash 
uniq foo.txt
```
- -c: Count the number of duplicate. 
- -d: Print duplicate. 
- -i: Case intensive. 

# DIFF
`DIFF` is used to display difference in files by comparing line by line. 
```bash 
diff foo.log bar.log
```
Output:
- a: Add.
- c: Change.
- d: Delete.
- #: Line number. 
- <: File1.
- >: File2.

# AWK
`AWK` is programming language for manipulating data. 
```bash
awk '{print $2}' foo.log
```
