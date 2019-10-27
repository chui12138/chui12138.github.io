# CTF记录
- 可以直接尝试php代码的网站 https://www.php.cn/dic/29.html
- 遇事不决
 1. 扫目录
 2. 抓包看请求
 3. 看源码
 
---

开始时间 2019/10/17

------------

## 编码总结

* base16：只有16进制的字符串
* base32：base16+=填充
* base64：base32+ ‘/’，‘+’
* md5：标准的md5有16位或32位，16进制字符串

---

## hackergame 2019
### 信息安全2077 时间类
haha 第一题我就🕊了，不愧是我
一开始的思路没错改那个时间戳但是貌似 是burp改不行还是我burp有问题？。。。。

来了来了，wp来了，中科大官方wp https://github.com/ustclug/hackergame2019-writeups?tdsourcetag=s_pctim_aiomsg

### 达拉崩吧 调试类
挺简单的，js文件很容易看出来变量v是关键，直接在买菜的时候改v就可以了

## bugkuctf
### web4
打开之后看源码，1.是post，2.如果a不为undefined而且为67d709b2b54aa2aa648cf6e87a7114f1就返回！0，即真

--------
### 文件包含漏洞

---
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

- 解法1.利用eval
可以构造payload，有点像数据库注入 hello=1);show_source(%27flag.php%27);var_dump(3 把这个带到eval( "var_dump($a);");中就是eval( "var_dump(1);show_source(%27flag.php%27);var_dump(3);");拆分成执行三条语句了 %27为单引号的ASCII
- 解法2.直接将flag.php文件读入变量hello中

?hello=get_file_contents('flag.php')或

?hello=file('flag.php')

- 解法3.利用php://filter + eval

http://120.24.86.145:8003/index.php?hello=1);include $_POST['f'];//  （把后面的括号注释掉）

在POST区域：f=php://filter/convert.base64-encode/resource=flag.php?

得到的结果base64解码即可 这个不懂可看下面的php：//filter妙用的链接

试着试着就崩了，没拿到flag可能是网站问题。。。

命令执行漏洞学习总结 https://www.jianshu.com/p/bf64c59b64e6

BugkuCTF-web-本地包含 https://blog.csdn.net/huangming1644/article/details/82818482

谈一谈php://filter的妙用 https://www.leavesongs.com/PENETRATION/php-filter-magic.html

---
### 调试类
#### 点击100万次
定位对应变量clicks，断电，在控制台赋值即可

---
### 综合类
#### 备份是个好习惯 双写绕过str_replace，md5弱类型绕过
详细题解https://blog.csdn.net/qq_42133828/article/details/84871544
1. 点击网址后可以看到一串16进制字符串，初步判断可以是base16或者md5
2. 扫文件目录（备份提示）发现有index.php.bak
3. 点击后发现要求下一个源码，看了源码后有几个点，首先是要绕过把key变成空字符，其次是要md5绕过

- $_SERVER['REQUEST_URI'] 的解释 https://baike.baidu.com/item/%24_SERVER/4897514
- parse_str函数缺陷 https://cloud.tencent.com/developer/article/1370529
- md5弱类型绕过 https://blog.csdn.net/dongyanwen6036/article/details/77658983

---
## XSS注入小游戏
这个系列的题目都是要你实现网站弹窗就可以了
### LEVEL3
问题抽象：双引号和尖括号被实体化，<变成`&lt;`   >变成`&gt;` 加了htmlspecialchars函数处理
解决方法：htmlspecialchars函数默认不过滤单引号，既然尖括号已经被实体化了，就不能再额外创造`<script></script>`标签了，直接利用`<input name=keyword  value=''>`标签本身，在这个标签里面注入`<input name=keyword value='' onclick='window.alert()'>`
### LEVEL5
问题抽象：html中所有事件都是以o开头，现在如果有on同时出现就会加一个_在on之间。想像之前那样调用html时间来弹窗会被改成o_n
解决方法：