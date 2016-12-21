---
layout: post
title: Ubuntu上手动使sublime text 3 支持中文
categories: [环境搭建]
tags: [Ubuntu, Sublime Text 3]
fullview: false
comments: true
---
安装了sublime之后一直没尝试输入中文，今天突然想用sublime写点东西，然后发现系统上的fcitx(搜狗输入法)没法用。

在网上发现了两种修改办法，逐一尝试了下，于是记录一下。

##第一种方法，网上大神写的脚本。

地址在这里: <https://github.com/lyfeyaj/sublime-text-imfix>

clone下来尝试运行了一下，发现没成功，重启了一下，依旧不好使。第一种方法pass。

##第二种方法，手动修改。

修改参考：<>

在～目录上创建一个名为sublime_imfix.c的文件

然后将下面的代码写入文件

{% highlight c%}
<code>
\#include <gtk/gtkimcontext.h>
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
</code>
{% endhighlight %}





