---
title: intellij IDEA 错误:requested without authorization, you can copy URL and open it in browser to trust it.解决办法
date: 2016-07-05 15:12:55
tags:
---
有时使用intellij idea 自带的浏览服务时,会弹出:

        Page 'xxxxxx' requested without authorization, you can copy URL and open it in browser to trust it.


解决办法为设置:
 设置中,搜索找到 [Debugger>allow unsigned requrest]并勾选上,就可以了

{% asset_img 1.png  %}
