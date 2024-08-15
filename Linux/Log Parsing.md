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
- -f: the field numbers. 
- -c: Specifies characters position

