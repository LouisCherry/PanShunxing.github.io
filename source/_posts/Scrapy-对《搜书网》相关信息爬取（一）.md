---
title: Scrapy 对《搜书网》相关信息爬取（一）
date: 2019-10-16 08:38:01
tags: 
- Python
- Scrapy
- 爬虫
---

## 工具版本

1. pyhon:3.7.3
2. scrapy:1.7.3
3. 存储工具：mysql 5.7.25

## 文件说明

* scrapy.cfg: 项目的配置文件
* tutorial/: 该项目的python模块。之后您将在此加入代码。
* tutorial/items.py: 项目中的item文件.
* tutorial/pipelines.py: 项目中的pipelines文件.
* tutorial/settings.py: 项目的设置文件.
* tutorial/spiders/: 放置spider代码的目录.

## 分析爬取需求

1. 按不同分类爬取
2. 按搜索关键字爬取
3. 爬取章节详细内容

### 按不同分类爬取

1. **爬取地址：[分类链接（例如玄幻类）](https://www.soshuw.com/xuanhuan/)** 该地址通过改变地址的最后一个节点来改变分类。
2. **爬取范围，如下图**
![](https://i.loli.net/2019/10/15/rKWIXM3SEle7A2G.png)

3. **提取爬取内容的Xpath格式**
* 每行记录的Xpath:`// *[ @ id = "newscontent"] / div[1] / ul / li`
* 提取每单个记录Xpath:

```
# 书籍类型
booktype = node.xpath('./span[1]/text()').extract();
# 书籍名称
booktitle = node.xpath('./span[2]/a/text()').extract();
# 最新章节名（可选项）
lastchapter = node.xpath('./span[3]/a/text()').extract();
# 书籍作者
bookauthor = node.xpath('./span[4]/text()').extract();
# 更新时间
updatetime = node.xpath('./span[5]/text()').extract();
# 书籍详情url
bookdetailurl = node.xpath('./span[2]/a/@href').extract();
# 最新章节url
lastchapterurl = node.xpath('./span[3]/a/@href').extract();
```
### 按搜索关键字爬取
1. **爬取地址：[搜索地址](https://www.soshuw.com/search.html)** 我们这边使用综合搜索，需要使用post请求提交搜索请求。
2. **爬取范围，如下图**!
![](https://i.loli.net/2019/10/15/SpOJ8GxMwbIWocn.png)

3. **请求模式**

```
POST /search.html HTTP/1.1
Host: www.soshuw.com
Connection: keep-alive
Content-Length: 43
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36
Cache-Control: no-cache
Origin: chrome-extension://aicmkgpgakddgnaphhhpliifpcfhicfo
X-Postman-Interceptor-Id: a9f9db52-39cf-e74f-57ba-cabd9b92eaab
Postman-Token: 1dfc8428-7ba5-eeef-1049-3448529a8887
DNT: 1
Content-Type: application/x-www-form-urlencoded
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: __cfduid=dbeeccea34cdcce4a2dbec9ef3f504ade1570091620; Hm_lvt_45ae70efc2af09e02ffd19a6568a6991=1570091597,1571010856; Hm_lpvt_45ae70efc2af09e02ffd19a6568a6991=1571012049
Pragma: no-cache

searchtype=all&searchkey=%E5%87%A1%E4%BA%BA
```
4. **提取爬取内容的Xpath格式**
* 每行记录的Xpath:`//*[@id="main"]/div[1]/ul/li`
* 提取每单个记录Xpath:

```
# 书籍类型
booktype = node.xpath('./span[1]/a/text()').extract();
# 书籍名称
booktitle = node.xpath('./span[2]/a/text()').extract();
# 最新章节名（可选项）
lastchapter = node.xpath('./span[3]/a/text()').extract();
# 书籍作者
bookauthor = node.xpath('./span[4]/text()').extract();
# 更新时间
updatetime = node.xpath('./span[5]/text()').extract();
# 书籍状态（可选）
bookstatus = node.xpath('./span[6]/text()').extract();
# 书籍详情url
bookdetailurl = node.xpath('./span[2]/a/@href').extract();
# 最新章节url
lastchapterurl = node.xpath('./span[3]/a/@href').extract();
```

### 爬取章节详细内容

1. **爬取地址：[章节内容地址](https://www.soshuw.com/FuTianShi/3420365.html)** 通过地址后面两个节点控制书籍和章节。
2. **爬取范围，如下图**
![](https://i.loli.net/2019/10/15/rQaXo7MVdibY6N9.png)
3. **提取爬取内容的Xpath格式**
* 内容标题的Xpath:`'//*[@id="chapter' + chapternum + '"]/div[3]/div[1]/h1'`
* 内容主体的Xpath:`'//*[@id="con' + chapternum + '"]'`
这里的chapternum都是地址的最后一个节点。


## 爬虫开发流程

1. 创建一个Scrapy项目
2. 定义提取的Item
3. 编写爬取网站的 spider 并提取 Item
4. 编写 Item Pipeline 来存储提取到的Item(即数据)

### step1:创建项目
`scrapy startproject tutorial`

### setp2:定义提取的Item
**文件名：items.py**

``` 
import scrapy


# 书籍详情
class BookItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    # 唯一id
    bookid = scrapy.Field();
    # 简单id
    simpleid = scrapy.Field();
    # 书籍拼音
    pingyin = scrapy.Field();
    # 书籍类型
    booktype = scrapy.Field();
    # 书籍名称
    booktitle = scrapy.Field();
    # 书籍作者
    bookauthor = scrapy.Field();
    # 最新章节名（可选项）
    lastchapter = scrapy.Field();
    # 更新时间
    updatetime = scrapy.Field();
    # 书籍状态（可选）
    bookstatus = scrapy.Field();
    # 书籍详情url
    bookdetailurl = scrapy.Field();
    # 最新章节url
    lastchapterurl = scrapy.Field();
    pass

# 书籍目录
class ChapterItem(scrapy.Item):
    # 章节名（卷名）
    chaptername = scrapy.Field();
    # 是否是卷
    ifvolume = scrapy.Field();
    # 章节地址
    chapterurl = scrapy.Field();
    # 章节序号
    chapternum = scrapy.Field();
    # 书籍id
    simpleid = scrapy.Field();
    pass

# 书籍章节内容
class ChapterContentItem(scrapy.Item):
    # 内容标题
    title = scrapy.Field();
    # 内容主体
    content = scrapy.Field();
    pass
```

### setp3:编写爬取网站的 spider 并提取 Item

**在项目根目录下使用scrapy提供的命令，通过内置模板生成spider.**
`scrapy genspider mydomain mydomain.com`
**文件名：soushuw.py**

```
# -*- coding: utf-8 -*-
import scrapy
import re
from soshuw.items import BookItem, ChapterItem, ChapterContentItem
from urllib.request import quote
import uuid

class SoshuwSpider(scrapy.Spider):
    name = 'soshu'
    allowed_domains = ['www.soshuw.com']
    # 从总榜开始爬
    start_urls = ['https://www.soshuw.com/top/allvisit.html/']
    # 搜索地址
    searchurl = ['https://www.soshuw.com/search.html'];

    # 主页地址
    domainurl = ['https://www.soshuw.com'];

    # def __init__(self,booktype,keywod,*args, **kwargs):
    #     super(SoshuwSpider, self).__init__(self,args,kwargs);


    def start_requests(self):
        url = self.start_urls[0];
        # 书籍分类
        booktype = getattr(self, "booktype", None);
        if booktype is not None:
            url = "https://www.soshuw.com/" + booktype;
            yield scrapy.Request(url, self.parsetypelist);
        # 搜索关键字
        keyword = getattr(self, "keyword", None);
        if keyword is not None:
            url = self.searchurl[0];
            yield scrapy.Request(url, self.parse, method='POST', body='searchtype=all&searchkey=' + quote(keyword)
                                 , headers={'Content-Type': 'application/x-www-form-urlencoded'});
        # 查看具体章节内容
        chapterurl = getattr(self, "chapterurl", None);
        if chapterurl is not None:
            url = self.chapterurl;
            splits = re.split('/', url);
            chapternum = re.sub('.html', '', splits[len(splits) - 1]);
            yield scrapy.Request(url, self.parsechapter, meta={"chapternum": chapternum});


    # 点击榜，搜索列表等通用列表解析
    def parse(self, response):
        nodelist = response.xpath('//*[@id="main"]/div[1]/ul/li[position()<3]');
        for node in nodelist:
            item = BookItem();
            # 书籍类型
            booktype = node.xpath('./span[1]/a/text()').extract();
            # 书籍名称
            booktitle = node.xpath('./span[2]/a/text()').extract();
            # 最新章节名（可选项）
            lastchapter = node.xpath('./span[3]/a/text()').extract();
            # 书籍作者
            bookauthor = node.xpath('./span[4]/text()').extract();
            # 更新时间
            updatetime = node.xpath('./span[5]/text()').extract();
            # 书籍状态（可选）
            bookstatus = node.xpath('./span[6]/text()').extract();
            # 书籍详情url
            bookdetailurl = node.xpath('./span[2]/a/@href').extract();
            # 最新章节url
            lastchapterurl = node.xpath('./span[3]/a/@href').extract();
            if len(booktitle) > 0:
                # try:
                item['bookid'] = str(uuid.uuid1());
                item['booktype'] = booktype[0];
                item['booktitle'] = booktitle[0];
                if len(lastchapter) > 0:
                    item['lastchapter'] = lastchapter[0];
                item['bookauthor'] = bookauthor[0];
                item['updatetime'] = updatetime[0];
                item['bookstatus'] = bookstatus[0];
                item['bookdetailurl'] = self.domainurl[0] + (bookdetailurl[0]);
                item['pingyin'] = re.sub('/', '', bookdetailurl[0]);
                if len(lastchapterurl) > 0:
                    item['lastchapterurl'] = self.domainurl[0] + lastchapterurl[0];
                    # except IndexError and TypeError :
                    #     pass
                yield scrapy.Request(item['bookdetailurl'], self.parsebook);
            yield item;
        pass

    # 解析分类列表
    def parsetypelist(self, response):
        nodelist = response.xpath('// *[ @ id = "newscontent"] / div[1] / ul / li');
        for node in nodelist:
            item = BookItem();
            # 书籍类型
            booktype = node.xpath('./span[1]/text()').extract();
            # 书籍名称
            booktitle = node.xpath('./span[2]/a/text()').extract();
            # 最新章节名（可选项）
            lastchapter = node.xpath('./span[3]/a/text()').extract();
            # 书籍作者
            bookauthor = node.xpath('./span[4]/text()').extract();
            # 更新时间
            updatetime = node.xpath('./span[5]/text()').extract();
            # 书籍详情url
            bookdetailurl = node.xpath('./span[2]/a/@href').extract();
            # 最新章节url
            lastchapterurl = node.xpath('./span[3]/a/@href').extract();
            if len(booktitle) > 0:
                # try:
                item['bookid'] = str(uuid.uuid1());
                item['booktype'] = booktype[0];
                item['booktitle'] = booktitle[0];
                if len(lastchapter) > 0:
                    item['lastchapter'] = lastchapter[0];
                item['bookauthor'] = bookauthor[0];
                item['updatetime'] = updatetime[0];
                item['bookdetailurl'] = self.domainurl[0] + (bookdetailurl[0]);
                item['pingyin'] = re.sub('/', '', bookdetailurl[0]);
                if len(lastchapterurl) > 0:
                    item['lastchapterurl'] = self.domainurl[0] + lastchapterurl[0];
                    # except IndexError and TypeError :
                    #     pass
                yield scrapy.Request(item['bookdetailurl'], self.parsebook);
            else:
                continue;
            yield item;
        # 分页
        # nextpage = response.xpath('//*[@id="pagelink"]/a[text()="下一页"]/@href').extract();
        # if len(nextpage) > 0:
        #     yield scrapy.Request(self.domainurl[0] + nextpage[0], self.parsetypelist);
        pass

    # 解析书籍章节列表
    def parsebook(self, response):
        simpletext = response.xpath('//body[@id]/@id').extract();
        simpleid = re.sub("xiaoshuo", "", simpletext[0]);
        nodelist = response.xpath('//*[@id="novel' + simpleid + '"]//dl/*[position()<30]');
        i = 0;
        if len(nodelist) > 0:
            for node in nodelist:
                item = ChapterItem();
                juan = node.xpath('./b/text()').extract();
                item['simpleid'] = simpleid;
                if len(juan):
                    # 章节名（卷名）
                    chaptername = node.xpath('./b/text()').extract();
                    item['chaptername'] = chaptername[0];
                    item['ifvolume'] = 1;
                else:
                    # 章节名（卷名）
                    chaptername = node.xpath('./a/text()').extract();
                    # 章节地址
                    chapterurl = node.xpath('./a/@href').extract();
                    item['chaptername'] = chaptername[0];
                    item['ifvolume'] = 0;
                    item['chapterurl'] = self.domainurl[0] + chapterurl[0];
                item['chapternum'] = i;
                i += 1;
                yield item;
        pass

    # 解析章节详情
    def parsechapter(self, response):
        chapternum = response.meta['chapternum'];
        item = ChapterContentItem();
        # 内容标题
        title = response.xpath('//*[@id="chapter' + chapternum + '"]/div[3]/div[1]/h1').extract();
        # 内容主体
        content = response.xpath('//*[@id="con' + chapternum + '"]').extract();
        item['title'] = title[0];
        item['content'] = content[0];
        yield item;
        pass
```

### setp4:编写 Item Pipeline 来存储提取到的Item(即数据)

**文件名：piplines.py**

```
# -*- coding: utf-8 -*-
import json
from soshuw.items import BookItem, ChapterItem, ChapterContentItem
import csv
import pymysql
import re
import uuid


class BookPipeline(object):

    def open_spider(self,spider):
        db = spider.settings.get('MYSQL_DB_NAME', '')
        host = spider.settings.get('MYSQL_HOST', '')
        port = spider.settings.get('MYSQL_PORT', '')
        user = spider.settings.get('MYSQL_USER', '')
        passwd = spider.settings.get('MYSQL_PASSWORD', '')

        self.db_conn = pymysql.connect(host=host, port=port, db=db, user=user, passwd=passwd, charset='utf8')
        self.db_cur = self.db_conn.cursor()

    # 关闭数据库
    def close_spider(self, spider):
        self.db_conn.commit()
        self.db_conn.close()

    # 插入数据
    def insert_db(self, item):
        iffinish = 0;
        if item.get('bookstatus', None) == '完结':
            iffinish = 1;
        # 书籍类型处理
        booktype = item.get('booktype', None).replace("[","").replace("]","");
        values = (
            item.get('bookid', None), item.get('sampleid', None), item.get('pingyin', None),
            booktype,item.get('booktitle', None),item.get('bookauthor', None),
            item.get('lastchapter', None),item.get('updatetime', None), item.get('bookstatus', None),
            item.get('bookdetailurl', None),item.get('lastchapterurl', None),iffinish
        )
        sql = "insert ignore into `book`.`book` ( `bookid`, `sampleid`, `pingyin`, `booktype`, `booktitle`, `bookauthor`, " \
              "`lastchapter`, `updatetime`, `bookstatus`, `bookdetailurl`, `lastchapterurl`, `iffinish`) " \
              "values ( %s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)"
        self.db_cur.execute(sql, values)

    def process_item(self, item, spider):
        if isinstance(item, BookItem):
            self.insert_db(item);
            pass
        return item


class ChapterPipeline(object):
    def open_spider(self,spider):
        db = spider.settings.get('MYSQL_DB_NAME', '')
        host = spider.settings.get('MYSQL_HOST', '')
        port = spider.settings.get('MYSQL_PORT', '')
        user = spider.settings.get('MYSQL_USER', '')
        passwd = spider.settings.get('MYSQL_PASSWORD', '')

        self.db_conn = pymysql.connect(host=host, port=port, db=db, user=user, passwd=passwd, charset='utf8')
        self.db_cur = self.db_conn.cursor()

    # 关闭数据库
    def close_spider(self, spider):
        self.db_conn.commit()
        self.db_conn.close()

    # 插入数据
    def insert_db(self, item):
        values = (
            str(uuid.uuid1()),item.get('simpleid', None),item.get('chaptername', None), item.get('ifvolume', None),
            item.get('chapterurl', None),item.get('chapternum', None)
        )
        sql = "insert ignore into `book`.`chapter` ( `chapterid`,`bookid`, `chaptername`, `ifvolume`, `chapterurl`, `chapternum`) " \
                "values ( %s,%s,%s,%s,%s,%s)"
        self.db_cur.execute(sql, values)


    def process_item(self, item, spider):
        if isinstance(item, ChapterItem):
            self.insert_db(item);
        return item

```

## 运行爬虫

### 运行爬取分类爬虫
`crawl soshu -a booktype=xuanhuan`

### 运行搜索爬虫
`crawl soshu -a keyword=凡人`

### 运行章节内容爬虫
`crawl soshu -a chapterurl=https://www.soshuw.com/FuTianShi/3420365.html`