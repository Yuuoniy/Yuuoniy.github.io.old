---
layout: '''silver'
title: Bullet'
date: 2018-03-29 22:23:06
tags:
---


注意stack或是struct的layout

漏洞strncat(bullet->desc,buf,MAX - bullet->power); 
这样首先create长度为47的bullet，通过power_up 再增加一个power 此时bullet长度为48
通过strnact 最后还会赋值 null ,因此造成溢出，可以覆盖接下来的 power的最低位,power會從\x2f蓋成\x00  
又因为 power = strlen(power) + power 所以最后power为1
杀死gin才会触发return;

再次powerup的时候,由于len和char间没有\0,strncat会连接到len的后面,从而改写len并rop.
注意payload 中不能有null 
由於程式本身有NX保護，必須使用ROP控制程式行為 但許多address因包含null byte，這些address無法被strncat複製到buffer上，因此覆寫main frame的return address時最多只能使用ROP執行某些gadgets。


再做一次 power_up() 就可以修改掉 power 的值並 overflow 到 main() 的 return address

最後把 power 改超大殺死 Werewolf 就可以觸發 return


内置的ROP怎么用
怎么查看 main的return address是在overflow之後7個bytes 呢？？？

参考：http://veritas501.space/2018/02/21/pwnable.tw%201~10%E9%A2%98%20writeup/
http://look3little.blogspot.sg/2017/03/silverbullet-200.html (十分详细)
