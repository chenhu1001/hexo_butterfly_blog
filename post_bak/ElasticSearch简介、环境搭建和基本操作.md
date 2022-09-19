---
title: ElasticSearch简介、环境搭建和基本操作
date: 2022-05-22 12:31:51
categories: Java
tags: [Java,ElasticSearch,Lucene,Zookeeper,Solr]
---
# ElasticSearch简介
## ElasticSearch（简称ES）
Elasticsearch是用Java开发并且是当前最流行的开源的企业级搜索引擎。  
能够达到实时搜索，稳定，可靠，快速，安装使用方便。  
客户端支持Java、.NET（C#）、PHP、Python、Ruby等多种语言。

官方网站: https://www.elastic.co/  
下载地址：https://www.elastic.co/cn/start

创始人:Shay Banon（谢巴农）

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_1.png-watermark)

应用场景

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_2.png-watermark)

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_3.png-watermark)

## ElasticSearch与Lucene的关系
Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库（框架）  
但是想要使用Lucene，必须使用Java来作为开发语言并将其直接集成到你的应用中，并且Lucene的配置及使用非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。  

Lucene缺点：
- 只能在Java项目中使用,并且要以jar包的方式直接集成项目中.
- 使用非常复杂-创建索引和搜索索引代码繁杂.
- 不支持集群环境-索引数据不同步（不支持大型项目）. 
- 索引数据如果太多就不行，索引库和应用所在同一个服务器,共同占用硬盘.共用空间少.

上述Lucene框架中的缺点,ES全部都能解决.

## 哪些公司在使用Elasticsearch
```
1. 京东
2. 携程
3. 去哪儿
4. 58同城
5. 滴滴
6. 今日头条
7. 小米
8. 哔哩哔哩
9. 联想
10. GitHup
11. 微软
12. Facebook
等等...
```

## ES vs Solr比较
### ES vs Solr 检索速度
当单纯的对已有数据进行搜索时，Solr更快。

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_4.png-watermark)

当实时建立索引时, Solr会产生io阻塞，查询性能较差, Elasticsearch具有明显的优势。

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_5.png-watermark)

大型互联网公司，实际生产环境测试，将搜索引擎从Solr转到 Elasticsearch以后的平均查询速度有了50倍的提升。

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_6.png-watermark)

总结：  
二者安装都很简单。  
- Solr 利用 Zookeeper 进行分布式管理，而Elasticsearch 自身带有分布式协调管理功能。
- Solr 支持更多格式的数据，比如JSON、XML、CSV，而 Elasticsearch 仅支持json文件格式。
- Solr 在传统的搜索应用中表现好于 Elasticsearch，但在处理实时搜索应用时效率明显低于 Elasticsearch。
- Solr 是传统搜索应用的有力解决方案，但 Elasticsearch更适用于新兴的实时搜索应用。

### ES vs 关系型数据库

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_7.png-watermark)

# Lucene全文检索框架
## 什么是全文检索
全文检索是指：
- 通过一个程序扫描文本中的每一个单词，针对单词建立索引，并保存该单词在文本中的位置、以及出现的次数
- 用户查询时，通过之前建立好的索引来查询，将索引中单词对应的文本位置、出现的次数返回给用户，因为有了具体文本的位置，所以就可以将具体内容读取出来了

hello   what  world    ====>   hello 和 what 和 world

## 分词原理之倒排索引

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_8.png-watermark)

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_9.png-watermark)

![](https://clang.oss-cn-shenzhen.aliyuncs.com/blog/2022/ElasticSearch%E7%AE%80%E4%BB%8B%E3%80%81%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E5%92%8C%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C_10.png-watermark)

倒排索引总结：  
索引就类似于目录，平时我们使用的都是索引，都是通过主键定位到某条数据，那么倒排索引呢，刚好相反，数据对应到主键．这里以一个博客文章的内容为例:  

1.索引

|  文章ID   | 文章标题  | 文章内容  |
|  ----  | ----  | ----  |
| 1  | 浅析JAVA设计模式 |  JAVA设计模式是每一个JAVA程序员都应该掌握的进阶知识 |
| 2  | JAVA多线程设计模式 |JAVA多线程与设计模式结合 |

2.倒排索引  
假如，我们有一个站内搜索的功能，通过某个关键词来搜索相关的文章，那么这个关键词可能出现在标题中，也可能出现在文章内容中，那我们将会在创建或修改文章的时候，建立一个关键词与文章的对应关系表，这种，我们可以称之为倒排索引,因此倒排索引，也可称之为反向索引．如：

|  关键词   | 文章ID  |
|  ----  | ----  |
| JAVA  | 1 |
| 设计模式  | 1,2 |
| 多线程  | 2 |

注：这里涉及中文分词的问题
