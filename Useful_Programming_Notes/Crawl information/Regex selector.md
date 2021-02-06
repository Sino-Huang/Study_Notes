# Regex selector

[TOC]

## Note: raw input for different language 

```python
# python
>>> print(r"\n")
\n
>>> print("\n")



```

![image-20210116190315341](images/image-20210116190315341.png)

```c++
// C++
 1 #include <iostream>
 2 #include <string>
 3 
 4 using namespace std;
 5 
 6 int main()
 7 {
 8     string normal_str="First line.\nSecond line.\nEnd of message.\n";
 9     string raw_str=R"(First line.\nSecond line.\nEnd of message.\n)";
10     cout<<normal_str<<endl;
11     cout<<raw_str<<endl;
12     return(0);
13 }
```

## Potential tutorial

https://www.youtube.com/watch?v=sa-TUpSx1JA



## CheatSheet

Anchors

![image-20210116170416583](images/image-20210116170416583.png)

1. word boundary 
   1. have boundary for the word (like whitespace, tab, newline, first word)

Character Classes 

![image-20210116171904490](images/image-20210116171904490.png)

Quantifier 

![image-20210116171924780](images/image-20210116171924780.png)

Escaping character 

![image-20210116171936996](images/image-20210116171936996.png)

Assertion

![image-20210116171957831](images/image-20210116171957831.png)

Groups and ranges

![image-20210116172009638](images/image-20210116172009638.png)



Pattern Modifier 

![image-20210116172026280](images/image-20210116172026280.png)



String replacement 

![image-20210116172035153](images/image-20210116172035153.png)

