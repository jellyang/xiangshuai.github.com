---
layout: post
title: shooter subtitle downloader详细分析
categories:
- Web
tags:
- 电影
- 电影字幕
- Python
---

##1. 动机
前段时间刚刚学习了一下Python，感觉Python确实非常的简单和强大，碰巧逛微博时发现了伯乐在线上面的一篇文章：[“你所写过的最好的Python脚本是什么？”](http://blog.jobbole.com/75244/)其中有个叫[subtitle-downloader](https://github.com/manojmj92/subtitle-downloader)的脚本引发了我的兴趣。此脚本可以根据电影在网上下载对应的字幕，可惜只能在[thesubdb](http://thesubdb.com/)网下载英文字幕。由于我也非常喜欢看电影，所以萌生了可下载中文字幕的Python脚本，也作为这段时间学习Python语言的实践总结。


##2. 国内字幕网站
国内比较知名的字幕网站有[人人影视](http://www.yyets.com/ "人人影视")和[射手网](http://www.shooter.cn/ "射手网")，所以分别分析了一下两个网站的html请求，主要采用firefox和超级赞的firebug插件进行分析。

###2.1 [人人影视](http://www.yyets.com/)：
首先我个人非常喜欢人人影视提供的字幕，字幕质量高且字体漂亮精致，所以我就先对人人影视网站的html文件进行分析，分析结果比较失望。主要有以下方面原因：
>
1. 许多电影的字幕人人影视都没有收录
2. 电影字幕主要通过关键字进行搜索，下载的字幕可能与实际电影不同步
3. 没有发现有提供的相关的API进行下载字幕

###2.2 [射手网](http://www.shooter.cn/)：
接着进行射手网的html分析，意外发现了一个惊喜，射手网除了提供关键字进行检索，而且可通过html5和Md5技术进行精确匹配电影文件对应的字幕，测试了几个文件，都搜寻到了结果，而且下载到的字幕也比较令人满意，所以最终采用射手网提供的字幕和基本流程进行Python脚本的开发。

##3. 详细分析射手网精确匹配流程
由于采用了比较文件hash值进行检索字幕，所以匹配到字幕的精度非常的高，通过进行实际操作和对射手网的html和javascript脚本进行分析，得到了精确匹配字幕的大致流程：

>
1. 通过拖拽文件到搜索框
2. 通过html5技术中的文件操作进行文件处理，这里没有上传整个文件，仅仅上传了文件中的4段分片（每段分片大概4KB），总计16KB
3. 通过SparkMd5对上传的4段分片进行hash值计算，合在一起作为此文件的唯一标记filehash
4. 通过filehash即文件名获取此文件对应的字幕文件列表，并以json格式输出
5. json中的每条则记录了字幕的链接和其对应延时等信息
6. 通过json中提供的链接便可下载对应电影的字幕文件

关于上述的精确匹配过程，主要是通过分析[sphash.js](http://www.shooter.cn/a/sphash.js)的脚本和url测试得知，下面主要对其进行分析：

###3.1 [sphash.js](http://www.shooter.cn/a/sphash.js)关键代码：
sphash脚本主要进行文件的分片和各个分片hash值得计算，关键通过以下两段代码实现。

3.1.1 文件分片代码

	a.CalcFileHashs = function (d) {
            var e = 4096;
            var c = Math.min(d.size, 4096);
            a.getSliceHash(d, c, Math.min(c + e, d.size), 0);
            c = Math.floor(d.size / 3 * 2);
            a.getSliceHash(d, c, Math.min(c + e, d.size), 1);
            c = Math.floor(d.size / 3);
            a.getSliceHash(d, c, Math.min(c + e, d.size), 2);
            c = Math.max(0, d.size - 8192);
            a.getSliceHash(d, c, Math.min(c + e, d.size), 3)
        };
上面函数参数d代表文件的基本信息包括：大小、文件路径等信息，主要进行文件分片操作，可看出每个分片大致为4096字节。下图为d的格式：
![](http://xiangshuai.github.io/resources/20140823185020.png)

3.1.2 hash值计算代码

	a.getSliceHash = function (g, i, d, h) {
            var c = new FileReader();
            c.onload = function (j) {
                if (j.target.readyState == FileReader.DONE) {
                    var f = j.target.result;
                    var k = new SparkMD5.ArrayBuffer();
                    k.append(f);
                    a.addSliceHash(k.end(), h)
                }
            };
            var e = g.slice(i, d, 'application/experimental');
            c.readAsArrayBuffer(e)
        };
上述代码的作用是对某个分片进行读取并通过SparkMD5算法进行hash值计算，关于SparkMD5算法比较复杂，感兴趣自行分析，这里就不做分析了。
###3.2 url测试：
	 
通过获取到文件4个分片的hash值，变可以精确的进行文件匹配。主要通过以下两个url请求。
3.2.1 获取对应文件的字幕列表

	https://www.shooter.cn/api/subapi.php?filehash=d751294c05ec7ce0dad50ba97807cac6%3Bde12cb6be164a22a7db10d9fb69d6db5%3Bd1a9a584d3a1846cd0fc7711b8ba40ed%3B0b9a6ceb459802fcec506e2f98d10392&pathinfo=Divergent.2014.RETAIL.1080p.WEB-DL.H264.AC3-EVO.mkv&format=json
通过上面url可得到返回的字幕列表，并以json格式输出：

	[{"Desc":"","Delay":0,"Files":[{"Ext":"srt","Link":"https://www.shooter.cn/api/subapi.php?fetch=MTQwODc5NjY1N3wwMms0NGRCNC1UNmZPekdvLXJfRko2bU8zNkE1eGFyWndlRzFSOC1OZUFKOWpCRVMtVEc4MmZXTVlwNjR6NjdiZkpUZW12eG9JRjFWRHgwN01QWnc2Y01vZGlhRUl1NHMxbm1UT3ZPa1UwZXhheTc1SkJqRlhrOFc3cWQ1RjROWXg1aWRJTlJ5NDBfMGN4UjUwZjFJVC1SYzBqRTRwUU1CM2pRMGQtWT18oXpmBxVzkrWQj2gHyNldXfRT2FhEZI9eOPbRR8RNulc=\u0026nonce=%98%EA-%0F%BFr%AA%17%81%E0%5EU%83~%88%DB"}]},
	{"Desc":"","Delay":0,"Files":[{"Ext":"srt","Link":"https://www.shooter.cn/api/subapi.php?fetch=MTQwODc5NjY1N3xqc3RVNVVFMl83OXVJMVhhVjZGQUpOS2UxMVFQSzFRWmdGc2huaVBNVG05UDNobVJYeWg0NlcyN3FmZTFKUUVPaHk3Y01iNVNGbmM3ZS13M2dCWkc1by0tUVg4Yy1JbEN5b3RBWURvUkViY3RzSkdhWnJSRTNubERXZ0JJYVpNc3JUSW9RV01YQWRGYjV0MklIanVuRXdCQVV6UXhDWDZFaVFJN3FlZz18G86t0qzz5vpC6_1ObZWtI-sp4h7qI7ZtiTmlTFRANBM=\u0026nonce=J%CCF%B7%88%B6s%A6%FD%B6%1A%9FZ%5D%B8%7B"}]},
	{"Desc":"","Delay":0,"Files":[{"Ext":"srt","Link":"https://www.shooter.cn/api/subapi.php?fetch=MTQwODc5NjY1N3w5VjJDUV9SYXpnNzdDeUtFRGFSQ0xEekdqWlp1YW9ZbEZCOUlBOG83RlJ4al9nOFFjc3ROcXJfeVFSTjcwaHVfZkdVeTdodC1jZ1c5RFNPMktMeDczQzVRVW1JMkFVX3VQQi1tTW9hOG5hVWNrOWxHQkVPM2NxUTNmWlkxTzBaSC0tMDcybmMtS1Y0STZHbFRtNF9rMll6WFJweFB0S0F2QlgxZUJQWT18NXOIF52s_qsZIuupexlMqRLZV4z8lzLX1nOSKVfskJo=\u0026nonce=%2Fu%E2%C4A%B7%DA%3EZ%ADr%A4m%CE%A3%B7"}]}]
![](http://xiangshuai.github.io/resources/20140823204055.png)

3.2.2 获取列表中的其中一个条目，根据此条目中的信息便可获取字幕文件的url地址，以上述条目中的第一条为例：
	
	https://www.shooter.cn/api/subapi.php?fetch=MTQwODc5NjY1N3wwMms0NGRCNC1UNmZPekdvLXJfRko2bU8zNkE1eGFyWndlRzFSOC1OZUFKOWpCRVMtVEc4MmZXTVlwNjR6NjdiZkpUZW12eG9JRjFWRHgwN01QWnc2Y01vZGlhRUl1NHMxbm1UT3ZPa1UwZXhheTc1SkJqRlhrOFc3cWQ1RjROWXg1aWRJTlJ5NDBfMGN4UjUwZjFJVC1SYzBqRTRwUU1CM2pRMGQtWT18oXpmBxVzkrWQj2gHyNldXfRT2FhEZI9eOPbRR8RNulc=&nonce=%98%EA-%0F%BFr%AA%17%81%E0%5EU%83~%88%DB

> 由于编码原因：所以需将上述url中的`\u0026`用`&`代替

通过进行上述url请求，即可请求道所需要的字幕文件。另外，经过测试可知，上述链接会随时间进行改变，所以每次得重新进行请求字幕列表，在根据选择列表中的条目进行构建url，直接用上述链接可能失效。
![](http://xiangshuai.github.io/resources/20140823204237.png)

##4 脚本实现代码：
根据以上分析的流程，用Python进行实现，通过简单操作便可方便的从射手网下载到字幕。

GitHub源代码: [shooter-subtitle-downloader](https://github.com/xiangshuai/shooter-subtitle-downloader)

##参考：
1. 伯乐在线:[“你所写过的最好的Python脚本是什么？”](http://blog.jobbole.com/75244/)
2. GitHub: [subtitle-downloader](https://github.com/manojmj92/subtitle-downloader)
3. SparkMD5:[http://www.ruby-doc.org/gems/docs/c/condo-1.0.4/app/assets/javascripts/condo/md5/spark-md5_js.html](http://www.ruby-doc.org/gems/docs/c/condo-1.0.4/app/assets/javascripts/condo/md5/spark-md5_js.html)
4. MD5 Shootout:[http://jsperf.com/md5-shootout/7](http://jsperf.com/md5-shootout/7)
5. Python md5 module: [http://effbot.org/librarybook/md5.htm](http://effbot.org/librarybook/md5.htm)
6. MD5测试比对：[http://www.cmd5.com/](http://www.cmd5.com/)