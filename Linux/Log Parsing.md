# Grep
`Grep` Allow you to search pattern in files, `ZGrep` for GZIP files. 
```bash
grep <pattern> [options] file.log
```
- -n: Number of lines that matches. 
- -i: Case insensitive. 
- -v: Invert match.
- -E: Extended regex. 
- -c: Count number of matches. 
- -l: Find file names that match that pattern. 

# NGrep
`NGrep` is used for analyzing network packets. 
```bash
ngrep -I file.pcap
```
- -d: Specify network interface. 
- -i: Case insensitive. 
- 