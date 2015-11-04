#Open edX中国社区筹备
##第一次筹备会
时间：2015年9月26日

内容：
* 社区的目标为推广Open edX在国内的应用，推进Open edX的本地化、研发技术的普及
* 社区的组成包括开发者和用户
* 社区将发布基于Docker的Open edX国内版本（开发演示版） 
* 建立技术委员会，用户委员会
* 定期Open edX线下交流活动
* 将组织社区技术力量编撰出版《Open edX开发指南》
* 欢迎有研发能力和愿意奉献时间的开发者加入，贡献者有小奖励哦~
* 技术委员会职责：
    * 制定基于Docker版本的开发规范、开发方法
    * 负责审核和发布代码提交流程规范
    * 定期发布Open edX国内社区版本

本文列出中国区Open edX使用者及开发者关注的一些问题，整理开发相关中文资源。

#《Open edX开发指南》
以社区协作的方式编写出版。
目录规划
* Open edX介绍
* Python＋Django
* 开源组件：Mysql/Mongo/rabbitMQ/Celery...
* 安装和配置
* 基础开发实例（选课人数／分类／标签／推荐）
* Xblock开发
* 数据分析
* 课程制作和运维

#社交网络
edx安装互助QQ群 197475193

Open edX开发交流QQ群 106781163

#中国社区版特性
* 完整汉化 by eduStack [访问](http://edustack.org/2015/10/26/open-edx-cypress%E5%AE%8C%E6%95%B4%E6%B1%89%E5%8C%96%E8%AF%AD%E8%A8%80%E5%8C%85/)
* 中文教师手册 by eduStack [访问](http://docs.edustack.org/Building_and_Running_an_edX_Course/)
* 中文学生手册 by eduStack [访问](http://docs.edustack.org/edX_Guide_for_Students/)
* 介绍视频使用youtube之外的视频源
* 选课人数显示
* 课程搜索
* 课程分类
* Xblock-OfficeMix
* Xblock-主观题
* 国内高教、职教、普教、基教的学科分类预置

#question list
1. 安装技巧
2. 课程介绍视频
3. 教师成绩单
4. 购物车货币
5. 支付宝接口
6. 日常运维（安全策略，备份策略，故障修复）
7. theme使用
8. 版本汉化
9. 文档本地化    
  *  edX学生指南  http://docs.edustack.org/edX_Guide_for_Students (By iflab.org & edustack.org)
  *  [edX 開始使用 LMS](http://edx-lms-zhtw.readthedocs.org/zh_TW/latest/read_me.html) (By edXPDRLab)
10. 与amazon S3相关的存储问题
11. Django Admin 退出500错误
12. 磁盘扩展方案

#文档
[edX Guide for Students中文版](http://docs.edustack.org/edX_Guide_for_Students/)

[Building and Running an edX Course中文版](http://docs.edustack.org/Building_and_Running_an_edX_Course/)

#development
1. Docker
  * [使用Open edX中国社区Docker版来开发调试edX](http://wwj718.github.io/edx-cypress-cn-for-deveiopment.html)
  * [Open edX中国社区Docker版安装使用](http://wwj718.github.io/edx-cypress-cn-install-and-use.html)

2. 数据分析本地化
  * [基于docker的edx数据分析](http://wwj718.github.io/edx-data-analysis-on-docker.html)
  * [edx-analytics-pipeline源码解读](http://wwj718.github.io/edx-analytics-pipeline-code-analysis.html)
  * [edx中数据可视化相关](http://wwj718.github.io/edx-insight.html)
3. workflow（代码管理，协作）
4. 开发环境
5. 移动端
6. API接口
7. 社交平台登录
8. 课程搜索/分类

#Xblock
1. 主观题
2. 数学函数图形
3. Office Mix

#Blog
*  [Jason Zhu](https://www.idefs.com/) 竹轩的博客
*  [writing for time](http://wwj718.github.io/category/edx.html)  种瓜的博客
*  [Card Games] (http://www.cnblogs.com/cardgames/)
*  [datochan](http://datochan.me/)  Datochan的博客
*  [eduStack](http://edustack.org/)
*  [晨飞](http://ichenfei.com)
*  [中科大移动云计算实验室](http://www.mc2lab.com/)

#解决方案
development.6.API接口 : [启用edx的API](http://wwj718.github.io/edx-api.html)