---
layout: post
title: CTF for PHP note
categories: [CTF]
tags: [Struts2,]
description: "CTF遇到的PHP题"
fullview: false
comments: true
---

# 解题说明

等级保护能力验证的题，本菜鸡当时懒癌发作没有尝试自己的解题思路，诶。。。

首先是给了一个页面，提示 `Sorry. You have no permissions.`。

![](http://o8lgx56x1.bkt.clouddn.com/blog/img/ctfphp1.png)

查看 cookie 发现是 base64 。解密之后替换中间的 `guest` 为 `admin` 绕过登陆限制。

![](http://o8lgx56x1.bkt.clouddn.com/blog/img/ctfphp2.png)

![](http://o8lgx56x1.bkt.clouddn.com/blog/img/ctfphp3.png)

![](http://o8lgx56x1.bkt.clouddn.com/blog/img/ctfphp4.png)

这时页面提供了源码，看下。

    <?php
    $role = "guest";
    $flag = "flag{test_flag}";
    $auth = false;
    if(isset($_COOKIE["role"])){
        $role = unserialize(base64_decode($_COOKIE["role"]));
        if($role === "admin"){
            $auth = true;
        }
        else{
            $auth = false;
        }
    }
    else{
        $role = base64_encode(serialize($role));
        setcookie('role',$role);
    }
    if($auth){
        if(isset($_POST['filename'])){
            $filename = $_POST['filename'];
            $data = $_POST['data'];
            if(preg_match('[<>?]', $data)) {
                die('No No No!');
            }
            else {
                $s = implode($data);
                if(!preg_match('[<>?]', $s)){
                    $flag="None.";
                }
                $rand = rand(1,10000000);
                $tmp="./uploads/".md5(time() + $rand).$filename;
                file_put_contents($tmp, $flag);
                echo "your file is in " . $tmp;
            }
        }
        else{
            echo "Hello admin, now you can upload something you are easy to forget.";
            echo "<br />there are the source.<br />";
            echo '<textarea rows="10" cols="100">';
            echo htmlspecialchars(str_replace($flag,'flag{???}',file_get_contents(__FILE__)));
            echo '</textarea>';
        }
    }
    else{
        echo "Sorry. You have no permissions.";
    }
    ?>

首先会查看提交的请求中是否存在 `<>` 如果没有则将传入的数据转化为字符串。如果其中存在 `<>` 则将flag生成在一个随机命名的文件中。

说实话还是自己菜，`implode()` 这个函数需要传入数组，如果传入的是字符串将报错，变量 `$s` 自然也就没有值。

![](http://o8lgx56x1.bkt.clouddn.com/blog/img/ctfphp7.png)

想要通过Post请求的形式传入数组可以使用 `data[0]=123&data[1]=<>` 的形式传入数组，这样的话在执行 `implode()` 函数的时候就不会使 `&s` 为空，成功绕过这段逻辑拿到flag。

![](http://o8lgx56x1.bkt.clouddn.com/blog/img/ctfphp8.png)