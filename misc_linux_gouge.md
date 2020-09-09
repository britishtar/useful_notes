# Derek's Misc Linux Gouge

## Searching
To search for a given filename:
```shell
# To find "myfile.tar.gz":
$ find . -type f -name "myfile.tar.gz"
# Or, using wildcards:
$ find . -type f -name 'myfile*'
```

To search for a pattern in files, recursively starting at current directory:
```shell
$ grep -rl "pattern" .
```
