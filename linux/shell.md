# Shell 技巧

## 条件判断

### 示例

```bash
num1=5
num2=3

if [ ${num1} -gt ${num2} ];then
    echo "5 > 3"
elif [ ${num1} -lt ${num2} ];then
    echo "5 <3"
else
    echo "equal"
fi
```

### 判断操作符

- 整型

```bash
-lt  小于
-le  小于等于
-eq  等于
-ge 大于等于
-gt   大于
-ne 不等于
```

- 浮点数

```bash
a=1.6
b=1.7
result=$(echo "$a>$b" |bc)
#result为0是假，为1是真
```

- 字符串

```bash
 =  字符串匹配
!=  字符串不匹配
>   字符器大于
<   字符串小于
-n  str   字符串不为NULL，长度大于0
-z  str   字符串为NULL，长度为0
```

- 文件比较

```bash
-f 文件存在为真
```

## 浮点数计算

```bash
sum=$(echo "scale=2;5+2.1" | bc)
```

## 变量嵌套

```bash
# HOST_TEST="test.domain.com"
DEV="TEST"
hostname=$(eval "echo \${HOST_${DEV}}")
echo ${hostname}
# test.domain.com
```

## 小写变大写

```bash
upper=$(echo "abc" |tr "a-z" "A-Z")
```

## 计算时调用变量

```bash
total=1
count=2
sum=$(echo "scal=5;$total+$count" | bc)
```
