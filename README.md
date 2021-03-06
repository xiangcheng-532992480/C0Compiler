# C0Compiler
Java实现的 c0 语言编译器，作者： shianqi@imudges.com

目录
---
* [1.C0语言的语法结构定义](#1)
* [2.假想栈式指令系统表](#2)

<h2 id="1">C0语言的语法结构定义</h2>

```
<程序> -> [<变量定义部分>] {<自定义函数定义部分>} <主函数>
<变量定义部分> -> int id {, id};
<自定义函数定义部分> -> ( int id | void id) '(' ')' <分程序>
<主函数> -> void main'(' ')' <分程序>
<分程序> -> '{' [<变量定义部分>] <语句序列> '}'
<语句序列> -> <语句> {<语句>}
<语句> -> <条件语句> | <循环语句> | '{'<语句序列>'}' | <自定义函数调用语句> | <赋值语句> | <返回语句> | <读语句> | <写语句> | ;
<条件语句> -> if '('<表达式>')' <语句> [else <语句> ]
<循环语句> -> while '(' <表达式>')' <语句>
<自定义函数调用语句> -> <自定义函数调用>;
<赋值语句> -> id = <表达式>;
<返回语句> -> return ['(' <表达式> ')'] ;
<读语句> -> scanf '(' id ')';
<写语句> -> printf '(' [ <表达式>] ')';
<表达式> -> [+｜-] <项> { (+｜-) <项>} 
<项> -> <因子>｛(*｜/) <因子>｝
<因子> -> id｜'(' <表达式>')' | num | <自定义函数调用>
<自定义函数调用> -> id '(' ')'
```

<h3>编译</h3>
```
    java -cp C://javacc-6.0/bin/lib/javacc.jar javacc ./src/com/imudges.C0Compile/JavaCC/Compiler.jj
```

语法名称 | 标识符
-----|----
程序 | A
变量定义部分 | B
自定义函数定义部分 | C
主函数 | D
分程序 | E
语句序列 | F
语句 | G
条件语句 | H
循环语句 | I
自定义函数调用语句 | J 
赋值语句 | K
返回语句 | L
读语句 | M
写语句 | N
表达式 | O
项 | P
因子 | Q
自定义函数调用 | R

```
A -> [B] {C} D
B -> int id {, id};
C -> ( int id | void id) '(' ')' E
D -> void main'(' ')' E
E -> '{' [B] F '}'
F -> G {G}
G -> H | I | '{'F'}' | J | K | L | M | N | ;
H -> if '('O')' | G [else G ]
I -> while '(' O')' G
J -> R;
K -> id = O;
L -> return ['(' O ')'] ;
M -> scanf '(' id ')';
N -> printf '(' [ O] ')';
O -> [+｜-] P { (+｜-) P} 
P -> Q｛(*｜/) Q｝
Q -> id｜'(' O')' | num | R
R -> id '(' ')'
```

```
A -> BCD
B -> int id J; | ε
S -> , id S | , id | ε
C -> HE | HEC | ε
T -> int id '(' ')' | void id '(' ')'
D -> void main'(' ')' E
E -> '{' F '}' | '{' BF '}'
F -> GU
U -> G | GU | ε
G -> H | I | '{'F'}' | J | K | L | M | N | ;
H -> if '('O')' | GV
V -> G | ε
I -> while '(' O')' G
J -> R;
K -> id = O;
L -> return W ;
W -> '(' O ')' | ε
M -> scanf '(' id ')';
N -> printf '(' X ')';
X -> O | ε
O -> Y P Z
Y = + | - | ε
Z = +PZ | -PZ | ε
P -> Q&
& -> *Q& | /Q& | ε
Q -> id｜'(' O')' | num | R
R -> id '(' ')'
```

```
A -> BCD
B -> int id J; | ε
S -> , id S | , id | ε
C -> HE | HEC | ε
T -> int id '(' ')' | void id '(' ')'
D -> void main'(' ')' E
E -> '{' F '}' | '{' BF '}'
  -> '{' F '}' | '{' int id J;F '}' | '{' F '}'
  
```




其中，id代表标识符，num代表整数，其含义及构成方式与C语言相一致；C0源程序中的变量需先定义后使用，其作用域与生存期与C语言相一致；自定义函数可超前使用（调用在前，定义在后）。
根据上面给定的C0文法及其说明和下列定义的假想栈式指令系统，按递归下降分析法设计并实现该C0语言的编译器，生成栈式目标代码；编写栈式指令系统的解释执行程序，输出目标代码的解释执行结果。 
假想的栈式指令系统表


<h2 id="2">假想栈式指令系统表</h2>

dic | t | a | 解释
----|---|---|------------ 
LIT | 0 | a	| 将常数值取到栈顶，a为常数值
LOD | t | a	| 将变量值取到栈顶，a为相对地址，t为层数
STO | t | a	| 将栈顶内容送入某变量单元中，a为相对地址，t为层数
CAL | 0 | a	| 调用函数，a为函数地址
INT | 0 | a	| 在运行栈中为被调用的过程开辟a个单元的数据区
JMP | 0 | a	| 无条件跳转至a地址
JPC | 0 | a	| 条件跳转，当栈顶值为0，则跳转至a地址，否则顺序执行
ADD | 0 | 0	| 次栈顶与栈顶相加，退两个栈元素，结果值进栈
SUB | 0 | 0	| 次栈顶减去栈顶，退两个栈元素，结果值进栈
MUL | 0 | 0	| 次栈顶乘以栈顶，退两个栈元素，结果值进栈
DIV | 0 | 0	| 次栈顶除以栈顶，退两个栈元素，结果值进栈
RED | 0 | 0	| 从命令行读入一个输入置于栈顶
WRT | 0 | 0	| 栈顶值输出至屏幕并换行
RET | 0 | 0	| 函数调用结束后,返回调用点并退栈

