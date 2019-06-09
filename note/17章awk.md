## *17章awk*

### *awk介绍*
- awk：*Aho, Weinberger, Kernighan*，报告生成器，格式化文本输出
有多种版本：New awk（nawk），GNU awk（ gawk）
- gawk：模式扫描和处理语言

action里面字符串必须使用*双引号*, 否则认为是变量



### awk*工作原理*
    第一步: 执行BEGIN{action;… }语句块中的语句

    第二步：从文件或标准输入(stdin)读取一行，然后执行pattern{ action;… }语句块，
    它逐行扫描文件，从第一行到最后一行重复这个过程，直到文件全部被读取完毕。

    第三步：当读至输入流末尾时，执行END{action;…}语句块

    BEGIN语句块在awk开始从输入流中读取行之前被执行，这是一个可选的语句块，
    比如变量初始化、打印输出表格的表头等语句通常可以写在BEGIN语句块中

    END语句块在awk从输入流中读取完所有的行之后即被执行，比如打印所有行的
    分析结果这类信息汇总都是在END语句块中完成，它也是一个可选语句块

    pattern语句块中的通用命令是最重要的部分，也是可选的。如果没有提供
    pattern语句块，则默认执行{ print }，即打印每一个读取到的行，awk读取的每
    一行都会执行该语句块

### *基本用法*：
awk [options] 'program' var=value file…
awk [options] -f programfile var=value file…
awk [options] 'BEGIN{action;… }pattern{action;… }END{action;… }' file ...
awk 程序可由：BEGIN语句块、能够使用模式匹配的通用语句块、END语句
块，共3部分组成
program 通常是被放在单引号中

选项：
-F “分隔符” 指明输入时用到的字段分隔符  
    以:为分隔符,取出passwd文本的第1列和第3列
    [root@CentOS7 ~]#awk -F: '{print $1, $3}' /etc/passwd
    root 0
    bin 1

-v var=value 变量赋值

$0 域 field 一列 代表整行 


****	
awk**变量**  
	-F 分隔符   awk -F:  '{print $1, $3}' /etc/passwd       
	FS field separator 读取文本时, 所使用字段分隔符(输入自动分隔符,默认为空白字符)     
	OFS 输出自动分隔符, 默认为空白字符 
	RS Record separator 记录的分隔符, 输入文本信息时, 所使用的换行符 OFS Output Filed Separator 输出记录的分隔符
	ORS: Output Row Separteor:输出记录分隔符,输出时用指定符号代替换行符
	NF 字段数量  (带$NF表示调用最后一各字段, $(NF-1)...)
	NR 记录的数量  (默认记录一行) 行号
	FNR: 各文件分别计数,记录号;
	FILENAME: 当前文件名;
	ARGC: 命令行参数的个数
	ARGV: 数组,保存的时命令行所给定的各参数
	
	FNR：各文件分别计数,记录号
	awk '{print FNR}' /etc/fstab /etc/inittab
	FILENAME：当前文件名
	awk '{print FILENAME}’ /etc/fstab
	ARGC：命令行参数的个数
	awk '{print ARGC}’ /etc/fstab /etc/inittab
	awk ‘BEGIN {print ARGC}’ /etc/fstab /etc/inittab
	ARGV：数组，保存的是命令行所给定的各参数
	awk ‘BEGIN {print ARGV[0]}’ /etc/fstab /etc/inittab
	awk ‘BEGIN {print ARGV[1]}’ /etc/fstab /etc/inittab

自定义变量(区分字符大小写)      
	(1) -v var=value        
	(2) 在program中直接定义     
	
	示例：
	awk -v test='hello gawk' '{print test}' /etc/fstab
	awk -v test='hello gawk' 'BEGIN{print test}'
	awk 'BEGIN{test="hello,gawk";print test}'
	awk -F:‘{sex=“male”;print $1,sex,age;age=18}’ /etc/passwd
	cat awkscript
	{print script,$1,$2}
	awk -F: -f awkscript script=“awk” /etc/passwd
***
## printf命令(格式化输出)  
格式化输出：printf “FORMAT”, item1, item2, ...  
	(1) 必须指定FORMAT  
	(2) 不会自动换行，需要显式给出换行控制符，\n  
	(3) FORMAT中需要分别为后面每个item指定格式符   
格式符：与item一一对应  
	%c：显示字符的ASCII码
	%d, %i：显示十进制整数
	%e, %E：显示科学计数法数值
	%f：显示为浮点数
	%g, %G：以科学计数法或浮点形式显示数值
	%s：显示字符串
	%u：无符号整数
	%%：显示%自身

*printf实例*   
	awk -F: ‘{printf "%s",$1}’ /etc/passwd  
	awk -F: ‘{printf "%s\n",$1}’ /etc/passwd  
	awk -F: '{printf "%-20s %10d\n",$1,$3}' /etc/passwd  
	awk -F:‘ {printf "Username: %s\n",$1}’ /etc/passwd  
	awk -F: ‘{printf “Username: %s,UID:%d\n",$1,$3}’ /etc/passwd  
	awk -F: ‘{printf "Username: %15s,UID:%d\n",$1,$3}’ /etc/passwd  
	awk -F: ‘{printf "Username: %-15s,UID:%d\n",$1,$3}’ /etc/passwd  

*修饰符*  
	#[.#] 第一个数字控制显示的宽度;第二个#表示小数点后精度，%3.1f  
	- 左对齐（默认右对齐） %-15s
	+ 显示数值的正负符号 %+d
****
*操作符*        
比较操作符：  
==, !=, >, >=, <, <=  
模式匹配符：  
	~：左边是否和右边匹配，包含  
	!~：是否不匹配  

逻辑*操作符*：与 **&&**，或 **||**，非 **!**  
	示例：  
	awk -F: '$3>=0 && $3<=1000 {print $1}' /etc/passwd  
	awk -F: '$3==0 || $3>=1000 {print $1}' /etc/passwd  
	awk -F: ‘!($3==0) {print $1}' /etc/passwd  
	awk -F: ‘!($3>=500) {print $3}’ /etc/passwd  

条件表达式（*三目*表达式）      
	selector?if-true-expression:if-false-expression  
	示例：  
	awk -F: '{$3>=1000?usertype="Common User":usertype="  
	SysUser";printf "%15s:%-s\n",$1,usertype}' /etc/passwd   

PATTERN:根据*pattern*条件，过滤匹配的行，再做处理  
	1. 如果未指定：空模式，匹配每一行  
	2.  /regular expression/：仅处理能够模式匹配到的行，需要用/ /括起来  
	 awk '/^UUID/{print $1}' /etc/fstab  
	 awk '!/^UUID/{print $1}' /etc/fstab  
	3.  relational expression: 关系表达式，结果为“真”才会被处理  
	真：结果为非0值，非空字符串  
	假：结果为空字符串或0值  

	0 和 空为假
	其他任意字符为真  1 
***
## *awk控制语句*     
### *if-else*  
语法：```if(condition){statement;…}[else statement]  
if(condition1){statement1}else if(condition2){statement2}else{statement3}  ```  
使用场景：对awk取得的整行或某个字段做条件判断   

示例：      
    awk -F: '{if($3>=1000)print $1,$3}' /etc/passwd  
	awk -F: '{if($NF=="/bin/bash") print $1}' /etc/passwd
	awk '{if(NF>5) print $0}' /etc/fstab


### while*循环*  
语法：while(condition){statement;…}  
	条件“真”，进入循环；条件“假”，退出循环  
使用场景：  
	对一行内的多个字段逐一类似处理时使用  
	对数组中的各元素逐一处理时使用  
示例：  
awk '/^[[:space:]]*linux16/{i=1;while(i<=NF)  
{print $i,length($i); i++}}' /etc/grub2.cfg  
awk ‘/^[[:space:]]*linux16/{i=1;while(i<=NF) {if(length($i)>=10) {print $i,length($i)}; i++}}’ /etc/grub2.cfg  

### *for*循环
	语法：for(expr1;expr2;expr3) {statement;…}
常见用法：
	for(variable assignment;condition;iteration process){for-body}

特殊用法：能够遍历数组中的元素  
	语法：for(var in array) {for-body}  
示例：  
	awk '/^[[:space:]]*linux16/{for(i=1;i<=NF;i++) {print $i,length($i)}}'
	/etc/grub2.cfg

switch语句
	语法：switch(expression) {case VALUE1 or /REGEXP/: statement1; case
	VALUE2 or /REGEXP2/: statement2; ...; default: statementn}

	break和continue
	 awk ‘BEGIN{sum=0;for(i=1;i<=100;i++){if(i%2==0)continue;sum+=i}print sum}'
	 awk ‘BEGIN{sum=0;for(i=1;i<=100;i++){if(i==66)break;sum+=i}print sum}'

netx:  
	next 提前结束本行的处理, 直接进入下一行的处理(awk自身循环)  
	awk -F: '{if($3%2!=0) next; print $1,$3}' /etc/passwd  
***
## *awk函数*  
数值处理：  
	rand()：返回0和1之间一个随机数  
	awk 'BEGIN{srand(); for (i=1;i<=10;i++)print int(rand()*100) }'  

字符串处理：  
	length([s])：返回指定字符串的长度  
sub(r,s,[t])：  
  对t字符串搜索r表示模式匹配的内容，并将第一个匹配内容替换为s  
	echo "2008:08:08 08:08:08" | awk 'sub(/:/,“-",$1)'  

gsub(r,s,[t])：  
对t字符串进行搜索r表示的模式匹配的内容，并全部替换为s所表示的内容  
	echo "2008:08:08 08:08:08" | awk ‘gsub(/:/,“-",$0)'  

split(s,array,[r])：  
以r为分隔符，切割字符串s，并将切割后的结果保存至array所表示的数组中，第一个索引值为1,第二个索引值为2,…  
	netstat -tn | awk '/^tcp\>/{split($5,ip,":");count[ip[1]]++}END{for (i in count) {print i,count[i]}}’

	rand() 随机数

	split !! 

### *awk中调用shell命令*
system命令
空格是awk中的字符串连接符，如果system中需要使用awk中的变量可以使用空格分隔，或者说除了awk的变量外其他一律用""引用起来
	awk 'BEGIN{system("hostname") }'
	awk 'BEGIN{score=100; system("echo your score is " score) }'

### *awk脚本*     
将awk程序写成脚本，直接调用或执行       
示例：      
	 cat f1.awk     
	{if($3>=1000)print $1,$3}       
	 awk -F: -f f1.awk /etc/passwd      
	 cat f2.awk     
	#!/bin/awk –f       
	#this is a awk script       
	{if($3>=1000)print $1,$3}       
	 chmod +x f2.awk        
	 f2.awk –F: /etc/passwd     

向*awk*脚本传递参数  
	格式：```awkfile var=value var2=value2... Inputfile``` 
**注意**：在BEGIN过程中不可用。直到首行输入完成以后，变量才可用。可以通
过-v 参数，让awk在执行BEGIN之前得到变量的值。命令行中每一个指定的变
量都需要一个-v参数  
示例：
```bash
cat test.awk
    #!/bin/awk –f
    {if($3 >=min && $3<=max)print $1,$3}
```
chmod +x test.awk  
test.awk -F: min=100 max=200 /etc/passwd    











































