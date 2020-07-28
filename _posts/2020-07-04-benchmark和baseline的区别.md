---
layout:		post
title		benchmark和baseline的区别
subtitle:	如何评价论文的工作成果
date:		2020-07-04
author		BY beta
header-img:	img/2020-07-04/head.png
catalog:	true
tags:
	-	科研
	-	转载
---

链接：https://www.zhihu.com/question/28823373/answer/101504099

来源：知乎

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



Benchmark和baseline都有性能比较的意思。

先看看字典定义。

> benchmark：N-COUNT A **benchmark** is something whose quality or quantity is known and which can therefore be used as a standard with which other things can be compared.

通俗的讲，一个算法之所以被称为benchmark，是因为它的**性能已经被广泛研究，人们对它性能的表现形式、测量方法都非常熟悉，因此可以作为标准方法来衡量其他方法的好坏**。
这里需要区别state-of-the-art（SOTA），能够称为SOTA的算法表明其性能在当前属于最佳性能。如果一个新算法以SOTA作为benchmark，这当然是最好的了，但如果比不过SOTA，能比benchmark要好，且方法有一定创新，也是可以发表的。

> baseline：N-COUNT A **baseline** is a value or starting point on a scale with which other values can be compared.

通俗的讲，一个算法被称为baseline，基本上表示**比这个算法性能还差的基本上不能接受的**，除非方法上有革命性的创新点，而且还有巨大的改进空间和超越benchmark的潜力，只是因为是发展初期而性能有限。所以baseline有一个自带的含义就是“**性能起点**”。这里还需要指出其另一个应用语境，就是**在算法优化过程中，一般version1.0是作为baseline的，即这是你的算法能达到的一个基本性能，在算法继续优化和调参数的过程中，你的目标是比这个性能更好**，因此需要在这个base line的基础上往上跳。

简而言之，
benchmark一般是和同行中比较牛的算法比较，比牛算法还好，那你可以考虑发好一点的会议/期刊；
baseline一般是自己算法优化和调参过程中自己和自己比较，目标是越来越好，当性能超过benchmark时，可以发表了，当性能甚至超过SOTA时，恭喜你，考虑投顶会顶刊啦。