## 输出

```bash
echo 'Hello World'
#输出
参 数：-n 不要在最后自动换行
　　-e 若字符串中出现以下字符，则特别加以处理，而不会将它当成一般文字输出：
　　\a 发出警告声；
　　\b 删除前一个字符；
　　\c 最后不加上换行符号；
　　\f 换行但光标仍旧停留在原来的位置；
　　\n 换行且光标移至行首；
　　\r 光标移至行首，但不换行；
　　\t 插入tab；
　　\v 与\f相同；
　　\\ 插入\字符；
　　\nnn 插入nnn（八进制）所代表的ASCII字符；
　　--help 显示帮助
　　--version 显示版本信息
　　每一个shell支持的echo指令可能都不太相同。
　　-n 不输出最后的\n
　　-e解释转义字符
　　-E不解释转义字符
```


## 变量的定义

```bash
#变量定义
a=12
a=HelloWorld
a='pwd' #‘变量替换符
a=$a:a
#$变量添加
#局部变量
local a=12
```
### 特殊变量

```bash
$0 #获取当前执行shell脚本的文件名，包括路径
$n #获取当前执行脚本的第n个参数
$* #获取当前shell的所有参数，将所有命令行参数视为单个字符，相当于$1$2...
$# #得到执行当前脚本的参数个数
$@ #获取这个程序的所有参数，并保留参数之间的任何留白，相当于$1 $2 $3
```
### 状态变量

```bash
$? #判断上一指令是否成功执行，0为成功
$$ #取当前进程的PID
$! #上一个指令的PID
```

## 自增自减

【方式一】declare -i来声明整数变量


```bash
root@localhost:~# declare -i x=1
root@localhost:~# x+=1
root@localhost:~# echo $x
2
```

【方式二】使用let命令

```bash
root@localhost:~# i=1
root@localhost:~# let i+=1
root@localhost:~# echo $i
2
root@localhost:~# i=1
root@localhost:~# let i=$i+1
root@localhost:~# echo $i
2
root@localhost:~# i=1
root@localhost:~# let i++
root@localhost:~# echo $i
2
root@localhost:~# i=1
root@localhost:~# let ++i
root@localhost:~# echo $i
2
```

【方式三】使用(())

```bash
root@localhost:~# i=1
root@localhost:~# ((++i))
root@localhost:~# echo $i
2
root@localhost:~# i=1
root@localhost:~# ((i++))
root@localhost:~# echo $i
2
```

【方式四】使用expr命令

```bash
root@localhost:~# i=1
root@localhost:~# i=`expr $i + 1`
root@localhost:~# echo $i
2
root@localhost:~# i=1
root@localhost:~# i=$(expr $i + 1)
root@localhost:~# echo $i
2
```

【方式五】使用$(())

```bash
root@localhost:~# i=1
root@localhost:~# i=$(($i + 1))
root@localhost:~# echo $i
2
```

【方式六】使用$[]

```bash
root@localhost:~# i=1
root@localhost:~# i=$[$i + 1]
root@localhost:~# echo $i
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200312194818517.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDEzMzY5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200312195141358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDEzMzY5,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200312195959209.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNDEzMzY5,size_16,color_FFFFFF,t_70)
-a -d -n -p -t -s

printf
```bash
printf "Hello %s!" "A"
#Hello A!
```
## 函数使用方法

```bash
#方法1
_name_() {

}
#方法2
function _name_() {

}

#使用的时候
_name_ 
```

## 流程控制IF

```bash
#1
if [[ condition ]]; then
fi

#2
if [[ condition ]]; then
	else
fi

#3
if [[ condition ]]; then
elif [[ condition ]]; then
	else
fi

```
## test表达式
### 整数比较大小

```shell
max=0
if [ "$1" -ge "$2" ]
then
max="$1"
else
max="$2"
fi

#-gt是大于
#-lt是小于
#-eq是等于
#-ne是不等于
#-ge是大于等于
#-le是小于等于
```

 **注意：**

"-eq" 对整数进行比较

"=="  对字符串进行比较，

```shell
[root@localhost script]# num1=1
[root@localhost script]# num2=2
[root@localhost script]# if [ $num1 -lt $num2 ]; then
> echo "yes"
> fi
yes
```

### 整数加减乘除运算

- 使用 expr命令 （其中做乘的时候*号要用斜杠进行转义）

```
echo `expr 9 / 3`             #3 
echo `expr 9 + 3`             #12
echo `expr 9 - 3`             #6
echo `expr 9 \* 3`            #27
echo `expr 9 % 2`             #1
```

- 使用bc命令

```
echo 9 + 3 |bc               #12
echo 9 \* 3 |bc              #27
echo 9 - 3 |bc               #6 
echo 9 / 3 |bc               #3
echo 9 % 2 |bc               #1
```

### 小数运算

```shell
echo `awk -v x=2.45 -v y=3.123 'BEGIN{printf "%.2f\n",x*y}'`
s=1.5
echo $s
echo `awk -v x=2.45 -v y="$s" 'BEGIN{printf "%.2f\n",x*y}'`
#product=`awk -v x=2.45 -v y="$s" 'BEGIN{printf "%.2f\n",x*y}'`

s2=$(echo "scale=3; 6 / 5"  | bc)
echo $s2

s3=$(echo "scale=3; $s / 5"  | bc)
echo $s3
```

### 小数比较

```shell
max=0.1
min=0.01
if [ `echo "$max > $min" | bc` -eq 1 ]; then
	echo Yes
fi
```







## 流程控制

### FOR

**第一类：数字性循环**

for1-1.sh

```
#!/bin/bash  
  
for((i=1;i<=10;i++));  
do   
echo $(expr $i \* 3 + 1);  
done  
```


for1-2.sh

```
#!/bin/bash  
  
for i in $(seq 1 10)  
do   
echo $(expr $i \* 3 + 1);  
done   
```


for1-3.sh

```
#!/bin/bash  
  
for i in {1..10}  
do  
echo $(expr $i \* 3 + 1);  
done  
```


for1-4.sh

```
#!/bin/bash  
  
awk 'BEGIN{for(i=1; i<=10; i++) print i}'  
```

**第二类：字符性循环**

for2-1.sh

```
#!/bin/bash  
  
for i in `ls`;  
do   
echo $i is file name\! ;  
done   
```


for2-2.sh

```
#!/bin/bash  
  
for i in $* ;  
do  
echo $i is input chart\! ;  
done  
```


for2-3.sh

```
#!/bin/bash  
  
for i in f1 f2 f3 ;  
do  
echo $i is appoint ;  
done  
```


for2-4.sh



```
#!/bin/bash  
  
list="rootfs usr data data2"  
for i in $list;  
do  
echo $i is appoint ;  
done  
```



**第三类：路径查找**

for3-1.sh

```
#!/bin/bash  
  
for file in /proc/*;  
do  
echo $file is file path \! ;  
done  
```


for3-2.sh

```
#!/bin/bash  
  
for file in $(ls *.sh)  
do  
echo $file is file path \! ;  
done  
```



### WHILE

```bash
while [[ condition ]]; do
	#statements
done
```
### CASE

```bash
case word in
	contidion )
		#statements
		;;
	contidion )
		#statements
		;;
esac
```
## 数组

```bash
declare -a a
	name[sub]=value
	name=(value1 value2...)
#数组
#输出数组内容
${arry[*]}
${arry[@]}

#确定数组元素个数
${#arry[@]}

#找到数组下标
${!arry[@]}

#数组追加
arry+=(a b c)

#删除数组与元素
unset
```

