## String类型
1. stripslashes(string) 去除字符串的反斜杠，常见用于去除json数据中的反斜杠
2. str_repaet(string, int) 重复字符串 第一个参数指定字符串 第二个参数为重复次数
3. sprintf(string, $string...) 根据格式化字符串生成的字符串 第一个字符串为需要格式化字符串，后面均为替换文本, %s代表字符串 %u代表整数型
4. addslashes(string) 使用反斜线引用字符串 (只在 ‘ “ \ 前添加\ ) 



## array类型
1. array_multisort(&$array, arg, $arr); 使用第一个作为参数的数组对后续的数组进行排序，需要注意的是，两个数组的长度需一样，否则会报错
2. 
