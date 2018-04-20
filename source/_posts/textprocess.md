---
title: textprocess
date: 2018-04-20 10:19:34
tags:
---

<font color='blue' style="font-style:italic" size="3">Years may wrinkle the skin,but to give up enthusiasm wrinkles soul.</font>

------

# 文本处理

&emsp;&emsp; 最近任务中需要对输出日志作一些处理，所以贴上来一些用作处理数据的代码，发现文本处理其实是挺有意思的意见事情，需要运用一些工具和技巧，然后得到自己需要的数据。

   首先是通过python从excel文件中抽取所需的数据列，然后格式化后输出为文本。代码如下:
	
	# -*- coding: utf-8 -*-
	import os, sys
	import xlrd
	
	
	def readexcel () :
		print('start process {0}'.format(sys.argv[1]))
		book = xlrd.open_workbook(sys.argv[1])
		ocrdata = open("ocrdata.txt","w")
		sh = book.sheet_by_index(0)
		i = 0;
		ocrdatastr = ""
		idList = []
		for rx in range(sh.nrows):
			if i==0 :
				i+=1
				continue;
			print(str(sh.cell_value(i,2)))
			ocrdatastr += str(sh.cell_value(i,2)).strip(" ")+"|"+str(sh.cell_value(i,1)).strip(" ")
			ocrdatastr += "\n"
			i+=1
			
		ocrdata.write(ocrdatastr)
	
		ocrdata.close()
		print('processing done')
	
	readexcel()

&emsp;&emsp;这样就能够截取我们感兴趣的excel列并按格式输出。

&emsp;&emsp;在跑完数据后，输出的日志如下:
   	
	d5e2ecb193124475a0645fb7e20933fd|586897328879702024|1|1
	28bde301fa5744d480a0f18bba89a3ab|600876334243123208|1|1
	87a7d2d79c524250ae0ba8c3270ac65f|592055097710612488|1|1
	a17ea816b2b7466d848aee1c9cc64136|447746225098199304|1|1
	1164dac206ab4e1292f5c6c07dddd618|502636076154753032|1|1
	c4fedb1b7c454c198962edda29f02234|555922166944632840|1|1
	88978559898e44df8ea7de92015ae521|635385481797832712|1|1

&emsp;&emsp;需要判断该列某些字段的值，然后输出到不同文件，于是想到了使用awk,参考陈皓的[博客](https://coolshell.cn/articles/9070.html)，但是发现里边的格式化实例不生效，于是尝试了用直接拼接的方式，发现可以work
    
	awk -F '|' '$4=="0" || $5=="0"  {print $1"|"$2"|"$4"|"$5}' ocrfailed.data  > noimagesfound.txt
	awk -F '|' '$4=="1"   {print $1"|"$2"|"$4"|"$5}' ocrfailed.data  > ocrfailed.txt
  
&emsp;&emsp;然后针对元数据需要找出重复的数据，用了
    
    sort ocrfix.data|unique -d 
  
&emsp;&emsp;嗯，此为一篇流水账。