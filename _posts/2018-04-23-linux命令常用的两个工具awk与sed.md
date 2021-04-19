---
title: Linux常用的两个工具
description: Linux常用的两个工具
categories:
- linux
tags:
- linux
---

<br>



### `AWK`  文本出现神器



`AWK`是因为其取了三位创始人 Alfred `Aho`，Peter `Weinberger`, 和 Brian `Kernighan` 的Family Name的首字符


`netstat -nat >> netstat.txt`



获取 `netstat.txt`中的内容


> `cat netstat.txt`

    
    Proto Recv-Q Send-Q Local Address               Foreign Address             State
    tcp        0      0 0.0.0.0:9995                0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:7051                0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:10059               0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:9099                0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:4171                0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:5003                0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:6379                0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:20171               0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:139                 0.0.0.0:*                   LISTEN
    tcp        0      0 0.0.0.0:10060               0.0.0.0:*                   LISTEN





#### 基本使用



    awk '{print $1, $4}' netstat.txt
    
        单引号中的被大括号括着的 是awk语句  (只能是单引号)
        
        $1 ~ $n 标识第几例 
        $0 标识整个行
        
    
 
#### 格式化输出 (类似于C的`printf`)


    
        awk '{printf "%-8s %-8s %-8s %-18s %-22s %-15s\n",$1,$2,$3,$4,$5,$6}' netstat.txt
        
            -8 右对其, 占8位
        
        

#### 条件过滤 



        awk '$3==0 && $6=="LISTEN" ' netstat.txt
        awk '$3==0 && $6=="LISTEN" {print $1, $4}' netstat.txt

        
            第三列等于0 且 第六列等于 LISTEN

            
#### 内建变量



<table border="0" cellspacing="1" cellpadding="4">
<tbody>
<tr>
<td bgcolor="#ffffff">$0</td>
<td bgcolor="#ffffff">当前记录（这个变量中存放着整个行的内容）</td>
</tr>
<tr>
<td bgcolor="#ffffff">$1~$n</td>
<td bgcolor="#ffffff">当前记录的第n个字段，字段间由FS分隔</td>
</tr>
<tr>
<td bgcolor="#ffffff">FS</td>
<td bgcolor="#ffffff">输入字段分隔符 默认是空格或Tab</td>
</tr>
<tr>
<td bgcolor="#ffffff">NF</td>
<td bgcolor="#ffffff">当前记录中的字段个数，就是有多少列</td>
</tr>
<tr>
<td bgcolor="#ffffff">NR</td>
<td bgcolor="#ffffff">已经读出的记录数，就是行号，从1开始，如果有多个文件话，这个值也是不断累加中。</td>
</tr>
<tr>
<td bgcolor="#ffffff">FNR</td>
<td bgcolor="#ffffff">当前记录数，与NR不同的是，这个值会是各个文件自己的行号</td>
</tr>
<tr>
<td bgcolor="#ffffff">RS</td>
<td bgcolor="#ffffff">输入的记录分隔符， 默认为换行符</td>
</tr>
<tr>
<td bgcolor="#ffffff">OFS</td>
<td bgcolor="#ffffff">输出字段分隔符， 默认也是空格</td>
</tr>
<tr>
<td bgcolor="#ffffff">ORS</td>
<td bgcolor="#ffffff">输出的记录分隔符，默认为换行符</td>
</tr>
<tr>
<td bgcolor="#ffffff">FILENAME</td>
<td bgcolor="#ffffff">当前输入文件的名字</td>
</tr>
</tbody>
</table>
        
        
        
       awk '$3==0 && $6=="LISTEN" && NR>=2  {printf "%02s %s %s %s\n",  NR, FNR,  $1, $4}' netstat.txt

        
        
#### 分隔字符


      awk -F: '{print $1, $3, $6}' /etc/passwd | grep jiamin
            
           指定分割符号
          -F :
          -F '[;;]'




#### 匹配字符


    awk -F : '$6 ~ /oo/ {print $1, $3, $6}' /etc/passwd
    awk -F : '$6 ！~ /oo/ {print $1, $3, $6}' /etc/passwd
    awk -F : '$6 ~ /oo|xx/ {print $1, $3, $6}' /etc/passwd
    
        $6 ~ /oo/     第六列包含 oo
        $6 !~ /oo/    第六列不包含 oo
        $6 ~ /oo|xx/  第六列包含 oo或者xx
        
        

#### 拆分字符到文件

    awk 'NR!=1{print > $6}' netstat.txt
    
        NR!=1表示 跳过表头
        
        
    分隔成不同的文件 
    
    ls
        ESTABLISHED  FIN_WAIT1  FIN_WAIT2  LAST_ACK  LISTEN  netstat.txt  TIME_WAIT




#### 增加条件分支


    
    awk 
    'NR!=1
    {
    if($6 ~ /TIME|ESTABLISHED/) print > "1.txt";
    else if($6 ~ /LISTEN/) print > "2.txt";
    else print > "3.txt" 
    }' 
    netstat.txt



#### 统计


    ls -l *.txt | awk '{sum+=$5} END {print sum}'
    
    
    awk 'NR!=1 {a[$6]++;} END {for (i in a) print i "," a[i];} ' netstat.txt
    
        注意这里会构建一个数组 a
        
        
    ps -aux | awk 'NR!=1 {a[$1]+=$6;} END {for(i in a) print i "," a[i]"KB";}'



#### 例子


    #从file文件中找出长度大于80的行
    awk 'length>80' file
    
    #按连接数查看客户端IP
    netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr
    
    查看连接你服务器 top10 用户端的 IP 地址：  
    netstat -nat | awk '{print $5}' | awk -F ':' '{print $1}' | sort | uniq -c | sort -rn | head -n 10

        sort -n  生序
        sort -rn 降序
    
    INFO 2021-04-17 19:14:15,755 /usr/xxx.py task_1 386: 账户额度调整，账户：123,调整额度：0.01,结果：{"success":true}
    awk '$2~/2021-04/&&/.账户额度调整/{print$2,$3, $7}' user_operate.log | awk -F ",调整额度：" '{print$1,$2}' | awk -F "账户额度调整，账   户：" '{print$1,$2,$3}' | sort -n -t " " -k 3
       

<br>

        
###  `SED` stream editor，流编辑器

`sed`基本上就是玩`正则模式匹配`



    
    demo.txt
    
        This is jimmy's cat
          jimmy's cat's name is betty
        This is jimmy's dog
          jimmy's dog's name is frank
        This is jimmy's fish
          jimmy's fish's name is george
        This is jimmy's goat
          jimmy's goat's name is adam



#### 基本操作

    
     sed "s/my/my/g" demo.txt > demo_1.txt


`-i `参数`直接修改文件内容`


     sed -i  "s/my/my/g" demo.txt 
     
     

|正则表达式|例子|
|---|---|
|`^`表示一行的开头|`/^#/ `以`#`开头的匹配|
|`$` 表示一行的结尾|`/}$/` 以`}`结尾的匹配|
|`\<`表示词首|`\<abc `以` abc`为首的詞|
|`\>` 表示词尾|`abc\> `表示以 abc 結尾的詞|
|`.`|表示任何单个字符|
|`[]`|`[ ] `字符集合|



    s 第一行
    
    g  全部


#### 常用方法



- 指定替换的行数


    sed "3s/my/your/g" pets.txt
    


- 替换第3到第6行的文本


    sed "3,6s/my/your/g" pets.txt


- 只替换`每一行`的`第一个` s


    sed 's/s/S/1' my.txt



- 替换`每一行` `第二个` 


    sed 's/s/S/2' my.txt


- 替换`每一行`的`第3个以后`的


     sed 's/s/S/3g' my.txt
     


#### 多个匹配条件


    sed "1,3s/jimmy/tony/g; 1,3s/tony/jimm/g" demo.txt

    分组, 变量
    sed 's/This is my \([^,&]*\),.*is \(.*\)/\1:\2/g' my.txt
    
    
    

#### sed的命令 `i`,`a` 

`append`，`insert`


    在第1行前插入一行
    sed "1 i hello" demo.txt
    
    
    匹配条件 ，插入
    sed "/fish/i hello" my.txt
    
    
    
    在第1行前追加一行
    sed "1 a hello" demo.txt
    
    
    
#### sed的命令 `c` 行替换


    第1行替换
    sed "1 c hello" demo.txt
    
    匹配条件 ，替换
    sed "/fish/c hello" my.txt
    
    


#### sed命令 `d` 删除

    
    sed '1 d' demo.txt
    
    sed '/fish/d' my.txt
    
    sed '2,$d' demo.txt  删除全部
    
    



#### sed 命令 `p` 输出打印


    sed -n '/fish/p' my.txt






[原文链接 -- SED 简明教程 By 酷 壳 – COOLSHELL](https://coolshell.cn/articles/9104.html)
