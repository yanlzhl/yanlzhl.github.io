---
layout: post
title: Jekyll 建站总结
date: 2017-08-30 10:30
description: 此次建站，完全使用开源的博客框架，节省了大量自建站从需求、编码到测试的工作时间，此外开源框架还有大量用户和开发人员的完善，且更易获得最新的更新并获得技术维护。
categories: [Jekyll,Hexo]
tags: [建站,博客,开源技术]
---

# Jekyll 建站总结
------
## 从Java到开源博客

此次建站，完全使用开源的博客框架，节省了大量自建站从需求、编码到测试的工作时间，此外开源框架拥有大量用户和开发人员的完善，且更易获得最新的更新并获得技术维护。

本是Java开发，早期第一想法便是做全栈，从需求、前端到运维自己开发维护，可一直因前端技术实力有限而受限，时间一拖再拖，如果能完整的做完定是一次非常不错的经验和能力的训练...从知识成本、时间周期、运营成本等考虑，大致如下：
> * 前端知识：H5、CSS3、JavaScript、Jquery、AngularJs/Vue
> * 后端知识：SpringBoot(SpringMVC+SpringData)、Nginx、ElasticSe rach、Logstash、Kibala、Redis
> * 运营成本：2G+RAM服务器*2...
> * 时间周期：3个月

主要问题在于前端开发，UI界面设计不出自己满意的成果，此要求不到，基本也就没有动力自建博客了，如果能和前端人员合作，对方拥有我的初衷还是非常乐意的；当然后端知识上用了一些不必的技术，由于自建站，当然想玩些新花样，而这些技术还是在自己掌握之内...一个人还是算了吧。
    
## Hexo&Jekyll总结 

本站建立在GithubPages上，结合Jekyll框架，能以超乎自己想象的速度建立在几十上百主题可供你选择的基础上拥有自己的博客，且不需要花钱购买额外的服务器及域名，便可开始自己的博客之旅。在选择Jekyll之前曾使用Hexo建站，建站成功之后却需要每次写好文章后再次generated、deploy到GitHub，也就意味着并不是每一台电脑都能自由的Post新的文章，因为你需要一个Node+npm+Hexo环境...而Jekyll不同，只需要一个浏览器，使用在线的MarkDown编辑器，文章成稿后形成Jekyll固定的.md文件格式，复制内容并在自己xxx.github.io的_post文件之下新建.md上便可，保存刷新自己网站，几乎秒速级别的发布程序，这是Jekyll的优势。相比Hexo，Jekyll是“古老的”，使用ruby,开发语言都是小众，遇到问题相关资料比较少，即使找到了，解决问题的成本也更高；而Hexo，可以说是后起之秀，使用Node.js，当下技术江湖热闹，资料繁多，且用户Js基础，前景也不错，大量使用人员可能倾向Hexo。而我选择Jekyll，一是在自己折腾过两者之后，Jekyll更符合我的需求，成功建站之后，使用基本没有额外任何烦劳，需要补充的是，Hexo优秀的theme,Jekyll也基本可以找到，还是能得到你满意的一款的！
    
## Jekyll建站过程中问题的解决方案

25号晚至今，几乎是5天时间整，从Hexo到Jekyll发布第一天文章，遇到的问题还是不少，尤其是自己电脑运行环境导致的千奇百怪只有自己想象不到的问题，从自己上网百度、google问题未果，到GitHub上提issue甚至写Email、Twitter联系开源者解决问题...折腾到心累而又不得不坚持到最后，终于是挺过来。下面给出几篇在Jekyll建站所遇问题的解决方案的链接，一是以后仍能后使用到而不必藏在自己的收藏夹中，这也是自己建立博客的原因，二是总结共享出来，在获得开源成果之下，也能继续开花结果。

> * [ windows7下安装ruby,rubyGems和devkit](http://blog.csdn.net/u012882134/article/details/51685127)
> * [NextT使用文档](http://theme-next.simpleyyt.com/)
> * [在 Windows 上搭建本地 Jekyll 编译环境时问题汇总](http://theme-next.simpleyyt.com/)
> * [Windows 下安装 Jekyll 及启动遇到的问题](https://silocean.github.io/2016/10/19/windows-install-jekyll/)
> * [GitHub Metadata No GitHub API authentication could be found](http://knightcodes.com/miscellaneous/2016/09/13/fix-github-metadata-error.html/)



