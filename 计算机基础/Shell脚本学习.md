##### 第一个简单的shell脚本
shell脚本以 .sh 为扩展名，其中 #! 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行。
```shell
#!/bin/bash
echo "Hello World !"
```
#### shell变量
##### 定义和使用变量
以下是一个使用变量的例子。shell 语言以#表示每行注释。
```shell
# 定义一个变量，注意变量名和等号之间不能有空格。
your_name="sayHello"
# 使用一个变量，即在变量面前加 $, 或着使用 ${} 将其包围起来
echo $your_name
echo ${your_name}
# readonly 变量名， 将变量变为只读状态
readonly your_name
# 删除变量
unset your_name
```
##### shell字符串
shell可用双引号或单引号定义一个字符串，使用双引号定义字符串可以使用转义字符和变量。
```shell
echo "hello, ${your_name}"  # 输出为： Hello, sayHello

# 使用#可获取字符串长度
str="abcd"
echo ${#str} # 输出为 4
```
##### shell数组
数组中可以存放多个值。Bash Shell 只支持一维数组（不支持多维数组）。
```shell
# 定义一个一维数组
array_name=(value1 value2 ... valuen)
# 也可以使用下标定义一维数组
array_name[0]=value1

# 读取一维数组的值
echo ${array_name[0]} # 输出 value1
```
shell也支持**关联数组**，类似于字典
```shell
# 使用declare来声明, -A 表示为关联数组
declare -A array_name
array_name["key1"]=value1
```
shell提供获得数组长度/数组所有内容/数组所有键的操作
```shell
declare -A my_array
my_array["hello"]="hello"
my_array["bye"]="bye"
# 使用@或者*获取数组中所有元素
echo ${my_array[*]}
echo ${my_array[@]}
# 使用！获取关联数组所有键
echo ${!my_array[*]}
# 使用#获取数组长度
echo ${#my_array[*]}
```
##### shell输入参数
我们可以在执行 Shell 脚本时，向脚本传递参数，脚本内获取参数的格式为：**$n**。**n** 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……
```shell
#!/bin/bash

echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
```
另外有一些特殊的参数：

| $# | 传递到脚本的参数个数 |
| --- | --- |
| $* | 以一个单字符串显示所有向脚本传递的参数。
如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 |
| $$ | 脚本运行的当前进程ID号 |
| $! | 后台运行的最后一个进程的ID号 |
| $@ | 与$*相同，但是使用时加引号，并在引号中返回每个参数。
如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 |

#### shell运算符
##### shell调用Linux命令
原生bash不支持简单的数学运算，但是可以通过其他命令来实现，如expr
```shell
val=`expr 2 + 2` #表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2
# 反引号`也可以用来使用Linux的命令
echo `ls`
```
###### 关系运算符
假设 a=10, b=20

| -eq | 检测两个数是否相等，相等返回 true。 | [ $a -eq $b ] 返回 false。 |
| --- | --- | --- |
| -ne | 检测两个数是否不相等，不相等返回 true。 | [ $a -ne $b ] 返回 true。 |
| -gt | 检测左边的数是否大于右边的，如果是，则返回 true。 | [ $a -gt $b ] 返回 false。 |
| -lt | 检测左边的数是否小于右边的，如果是，则返回 true。 | [ $a -lt $b ] 返回 true。 |
| -ge | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。 |

条件表达式要放在方括号之间，并且要有空格，例如: **[$a==$b]** 是错误的，必须写成 **[ $a == $b ]**。
##### 布尔运算符
假设 a=10, b=20

| ! | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。 |
| --- | --- | --- |
| -o | 或运算，有一个表达式为 true 则返回 true。 | [ $a -lt 20 -o $b -gt 100 ] 返回 true。 |
| -a | 与运算，两个表达式都为 true 才返回 true。 | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |

##### 字符串运算符
假定变量 a 为 "abc"，变量 b 为 "efg"：

| = | 检测两个字符串是否相等，相等返回 true。 | [ $a = $b ] 返回 false。 |
| --- | --- | --- |
| != | 检测两个字符串是否不相等，不相等返回 true。 | [ $a != $b ] 返回 true。 |
| -z | 检测字符串长度是否为0，为0返回 true。 | [ -z $a ] 返回 false。 |
| -n | 检测字符串长度是否不为 0，不为 0 返回 true。 | [ -n "$a" ] 返回 true。 |
| $ | 检测字符串是否不为空，不为空返回 true。 | [ $a ] 返回 true。 |

##### 文件运算符
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。 | [ -b $file ] 返回 false。 |
| --- | --- | --- |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。 | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。 | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。 |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。 | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。 | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。 | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。 | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。 | [ -r $file ] 返回 true。 |
| -w file | 检测文件是否可写，如果是，则返回 true。 | [ -w $file ] 返回 true。 |
| -x file | 检测文件是否可执行，如果是，则返回 true。 | [ -x $file ] 返回 true。 |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。 | [ -s $file ] 返回 true。 |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。 | [ -e $file ] 返回 true。 |

#### 流程控制
##### if 判断
```shell
# if-else
if condition
then
    command1 
    command2
    ...
    commandN
else
    command
fi

# if else-if else
if condition1
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```
##### 循环控制
```shell
# for循环
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done

# while语句
while condition
do
    command
done

# case语句
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2)
    command1
    command2
    ...
    commandN
    ;;
esac

# 同样，break 和 continue 也具有循环控制的能力
```
#### shell函数
```shell
# 下面为一个简单的函数
funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```
