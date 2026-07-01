# Linux Cheat Sheet

## Viewing
```bash
cat file
cat -n file
more file
less file
head -n 20 file
tail -n 20 file
tail -f file
```

## grep
```bash
grep text file
grep -i text file
grep -n text file
grep -v text file
grep -c text file
grep -w text file
grep -x "line" file
grep -o pattern file
grep -l pattern *.log
grep -L pattern *.log
grep -A 2 pattern file
grep -B 2 pattern file
grep -C 2 pattern file
grep -r pattern .
```

## wc
```bash
wc file
wc -l file
wc -w file
wc -m file
wc -c file
```

## sort & uniq
```bash
sort file
sort -r file
sort -n file
sort -k2 file
sort -u file
sort -h file
uniq file
uniq -c file
uniq -d file
uniq -u file
```

## diff
```bash
diff old new
diff -u old new
diff -y old new
cmp a b
```

## Redirection & Pipes
```bash
command > file
command >> file
command < file
command 2> err.log
command 2>> err.log
command > out.log 2>&1
command &> out.log
command > /dev/null
command 2> /dev/null
command &> /dev/null
command | tee file
command | tee -a file

ls | wc -l
ps -ef | grep ssh
grep ERROR app.log | wc -l
sort file | uniq -c | sort -nr
```
