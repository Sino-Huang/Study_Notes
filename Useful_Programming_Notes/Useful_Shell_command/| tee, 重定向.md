# 重定向

```bash
sukai@sukai-MS-7B89:~/Templates$ echo test > test.txt
sukai@sukai-MS-7B89:~/Templates$ cat test.txt 
test
sukai@sukai-MS-7B89:~/Templates$ cat test.txt > test2.txt
sukai@sukai-MS-7B89:~/Templates$ cat test2.txt 
test
sukai@sukai-MS-7B89:~/Templates$ cat test.txt | cat -n > test3.txt
sukai@sukai-MS-7B89:~/Templates$ cat test3.txt 
     1	test

```

