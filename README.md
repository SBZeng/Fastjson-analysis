# Fastjson简析

        这是笔者在中国科学院大学上王伟老师的面向对象程序设计一课时的作业要求——对Fastjson进行分析以考察面向对象思想在其中的应用。

        因此，本文的重点不是Fastjson的介绍与分析，也不是具体源码实现的解析与算法设计的讨论，而是对需求建模、主要功能流程设计、类及类间关系、面向对象设计原则及设计模式的分析。总而言之，是对面向对象思想的探讨。

        因此如果读者感兴趣的是Fastjson的实现解析，可以参照商宗海的[fastjson-source-code-analysis](https://zonghaishang.gitbooks.io/fastjson-source-code-analysis/content/)，这篇文章提供了源码的解析。读者在阅读过程中也可以参照他的[fastjson](https://github.com/zonghaishang/fastjson)分支，其中添加了大量的注释，一定程度上解决了Fastjson的代码中注释过少带来的阅读不便。

        如果读者在阅读过程中有任何意见或者建议，可以发邮件与我联系，我的邮箱是：[zenghongbin16@mails.ucas.ac.cn](zenghongbin16@mails.ucas.ac.cn)。如果读者对此感兴趣，也很欢迎各位的加入，您同样可以通过邮件联系我。

        此外，每节的开头都有主要内容的介绍，希望这可以帮助大家抓住重点。

        **曾鸿斌**

        **2019.1.9**

