# Lab1
实验的文档写的很好，包含一些代码和特殊需求，如果弄懂mapreduce的原理的话多读两遍文档基本上可以把大致思路想好，剩下的可能是debug上比较耗时间。\
弄懂mapreduce的话还是建议看paper，如果细节上有些没懂的话可以看这个讲解视频[here](https://www.bilibili.com/video/BV1Vb411m7go/?spm_id_from=333.880.my_history.page.click&vd_source=6843e8130f0ab21fead4e499615c57cb)\
实验使用golang，如果学过c的话做一遍文档中提供的教程基本可以掌握实验所需的部分，值得注意的是golang的结构体的变量在赋值为0或nil的时候，值不发生变化。(我因为这个debug了一个下午 T.T)\
我下载的代码中提供的test中的crash测试似乎有点问题，只启动了一个worker，建议最好启动8个，不然很有可能全部worker都die了。

# Lab2
