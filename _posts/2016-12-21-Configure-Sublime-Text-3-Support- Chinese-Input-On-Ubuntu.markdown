---
layout: post
title: Ubuntu上使sublime text 3 支持中文
categories: [Configuration]
tags: [Ubuntu, Sublime-Text-3]
fullview: false
comments: true
---
安装了sublime之后一直没尝试输入中文，今天突然想用sublime写点东西，然后发现系统上的fcitx(搜狗输入法)没法用。

在网上发现了两种修改办法，逐一尝试了下，于是记录一下。

## 第一种方法，网上大神写的脚本。

地址在这里: <https://github.com/lyfeyaj/sublime-text-imfix>

clone下来尝试运行了一下，发现没成功，重启了一下，依旧不好使。第一种方法pass。

## 第二种方法，手动修改。

修改参考：<http://jingyan.baidu.com/article/f3ad7d0ff8731609c3345b3b.html>

在<code>~</code>目录上创建一个名为sublime_imfix.c的文件

然后将下面的代码写入文件

{% highlight c%}
#include <gtk/gtkimcontext.h>
void gtk_im_context_set_client_window (GtkIMContext *context,
         GdkWindow    *window)
{
 GtkIMContextClass *klass;
 g_return_if_fail (GTK_IS_IM_CONTEXT (context));
 klass = GTK_IM_CONTEXT_GET_CLASS (context);
 if (klass->set_client_window)
   klass->set_client_window (context, window);
 g_object_set_data(G_OBJECT(context),"window",window);
 if(!GDK_IS_WINDOW (window))
   return;
 int width = gdk_window_get_width(window);
 int height = gdk_window_get_height(window);
 if(width != 0 && height !=0)
   gtk_im_context_focus_in(context);
}
{% endhighlight %}

将上面的代码编译成共享库，这个时候需要编译环境

<code>sudo apt-get install build-essential libgtk2.0-dev</code>

然后在～目录下使用下面的命令进行编译

<code>gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC</code>

然后将编译好的库文件放入Sublime Text的文件夹中。

<code>sudo mv libsublime-imfix.so /opt/sublime_text/</code>

然后将Sublime Text的启动文件为修改下面的代码。

<code>sudo vi /usr/bin/subl</code>

{% highlight bash %}
#!/bin/sh
LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text "$@"
{% endhighlight %}

这个时候使用<code>subl</code>命令的时候就可以使用中文输入法了。

为了更好适配平常的使用习惯，还需修改Sublime Text的图标启动文件。

这个文件位置在这里-> <code>/usr/share/applications/sublime_text.desktop</code>

将[Desktop Entry]中的字符串
<code>Exec=/opt/sublime_text/sublime_text %F</code>
修改为
<code>Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text %F"</code>
将[Desktop Action Window]中的字符串
<code>Exec=/opt/sublime_text/sublime_text -n</code>
修改为
<code>Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text -n"</code>
将[Desktop Action Document]中的字符串
<code>Exec=/opt/sublime_text/sublime_text --command new_file</code>
修改为
<code>Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text --command new_file"</code>

上面是原文中提到的修改方法。

为了省事【偷懒？
可以直接将<code>Exec</code>直接指向刚刚修改的<code>/usr/bin/subl</code>

即：<code>Exec=/usr/bin/subl %F</code>

举一反三上面的三种替换都可以用这个代替。
