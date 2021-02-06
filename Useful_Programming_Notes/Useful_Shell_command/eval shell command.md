# eval shell command

[TOC]



eval将会首先扫描命令行进行所有的替换，再执行命令。该命令使用于那些一次扫描无法实现其功能的变量。该命令对变量进行两次扫描。这些需要进行两次扫描的变量有时候被称为复杂变量。



## double time scan 

test.txt内容：hello shell world!

```bash
myfile="cat test.txt"
echo $myfile　　#result:cat test.txt
eval echo $myfile　　#result:hello shell world!

# Assign a string value into the variable, $str1
str1="Shell Script"

# Assign the variable name, “str1” to the variable $str2
str2=str1
#Store the command into the variable, $command
command="echo"

# `eval` command will execute the `echo` command and print the value of the variable
# that contains in another variable
eval $command \${$str2} # this is same as using double quote
eval '$command \${$str2}' # only 1 times evaluation 
eval "$command \${$str2}" 
eval "$command \$$str2"  # double time evaluation 


# eval $command ${$str2} #bad substitution
eval $command $str2 # output #1 
eval $command $$str2

# they are the same
eval str3=\"\$\{$str2\}\"
eval str3=\"\${$str2}\" 

# following are the same
echo "$str2"
echo $str2 

# if you want to print " you need to use \
echo \"$str2\" 
```

```bash
# Sometimes you really need \" to work 

str1="Shell Script"

# Assign the variable name, “str1” to the variable $str2
# str2=str1
#Store the command into the variable, $command
command="echo"

# `eval` command will execute the `echo` command and print the value of the variable
# that contains in another variable
eval str2=\"$str1\"
eval str2=$str1  # ouptut Script: command not found, it means it changes to eval str2=Shell Script
echo $str2 #output Shell Script
```



## prepend function analysis 

```bash
export PATH=/opt/myapp/bin:$PATH
export LD_LIBRARY_PATH=/opt/myapp/lib; $LD_LIBRARY_PATH

# PATH 和 LD_LIBRARY_PATH 现在看起来应该像这样:

PATH=/opt/myapp/bin:/usr/bin:/bin
LD_LIBRARY_PATH=/opt/myapp/lib:/usr/lib; /lib

prepend() {
    [ -d "$2" ] && eval $1=\"$2\$\{$1:+':'\$$1\}\" && export $1 ;
}

prepend PATH /opt/myapp/bin
prepend LD_LIBRARY_PATH /opt/myapp/lib

```

### 分析

#### [ -d "$2" ] 

[] 等于 test， 是 一个command

-d 查看是不是directory

$2 第二个 parameter 

#### &&

run the next command if the previous command is true 

#### eval 

```bash
eval $1=\"$2\$\{$1:+':'\$$1\}\"
$1=\"$2\$\{$1:+':'\$$1\}\"
PATH=\"$2\$\{$1:+':'\$$1\}\"  # $1 change to PATH
PATH=$2\$\{$1:+':'\$$1\}  # \" safely wraping the nested command
PATH=$2\$\{$1:+':'PATHexpand\} # \$$1 => \$PATH => $PATH=> PATHexpand
# noted that for second evaluation, we use \$ \{ \} etc 
PATH=Direxpand\$\{PATH:+   ':'PATHexpand\}



${parameter:+word} # 当变量没有被设定，或是为空值的时候，则不会展开任何东西。否则会展开word的内容。

```





## exporting function 

```bash
export -f function_name
```

