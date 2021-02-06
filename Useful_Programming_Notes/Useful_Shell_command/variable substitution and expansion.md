# variable substitution and expansion 

```bash
${parameter} 变量的内容会被展开

${parameter:-word} 当变量没有被设定，或是为空值的时候，word就会被展开。否则，变量被展开。

${parameter:=word} 当变量没有被设定，或是为空值的时候，word就会被展开，而且这个值会被代入到变量里面。若不是，则变量内容被展开。

${parameter:?word} 当变量没有被设定，或是为空值的时候，会将word显示在标准错误输出，并结束。若不是的话，则变量的内容会被展开。

${parameter:+word} 当变量没有被设定，或是为空值的时候，则不会展开任何东西。否则会展开word的内容。
```

