# sed shell command

https://coolshell.cn/articles/9104.html

TBC

[TOC]

## Change by s command 

```sh
$ cat pets.txt
This is my cat
  my cat's name is betty
This is my dog
  my dog's name is frank
This is my fish
  my fish's name is george
This is my goat
  my goat's name is adam
```

s means change command 

/my means change my

/Hao Chen's means change to this 

/g apply to all 

```sh
$ sed "s/my/Hao Chen's/g" pets.txt
This is Hao Chen's cat
  Hao Chen's cat's name is betty
This is Hao Chen's dog
  Hao Chen's dog's name is frank
This is Hao Chen's fish
  Hao Chen's fish's name is george
This is Hao Chen's goat
  Hao Chen's goat's name is adam
```

注意：如果你要使用单引号，那么你没办法通过\’这样来转义，就有双引号就可以了，在双引号内可以用\”来转义。

再注意：上面的sed并没有对文件的内容改变，只是把处理过后的内容输出，如果你要写回文件，你可以使用重定向，如

```sh
$ sed "s/my/Hao Chen's/g" pets.txt > hao_pets.txt

# or use -i

$ sed -i "s/my/Hao Chen's/g" pets.txt
```

### append or tailing stuff by using s

^ and $ are regex starting and end pattern

```sh
$ sed 's/^/#/g' pets.txt
#This is my cat
#  my cat's name is betty
#This is my dog
#  my dog's name is frank
#This is my fish
#  my fish's name is george
#This is my goat
#  my goat's name is adam

$ sed 's/$/ --- /g' pets.txt
This is my cat ---
  my cat's name is betty ---
This is my dog ---
  my dog's name is frank ---
This is my fish ---
  my fish's name is george ---
This is my goat ---
  my goat's name is adam ---
```

