# 介绍 Intro
A web crawler that gets environmental reports of top companies in China
一个获得社会责任报告内容的爬虫工具

## 主要思路
主要思路：已知股票代码，通过东方财富网站获得该公司的论坛主页，并获取到该公司最新一期的社会责任报告pdf。

将pdf的文本转换成文本，并使用whoosh创建全文索引。

搜索关键词并获得目标文字附近的文本，如果有需要可以查看pdf当页具体截图。

## 提前准备工作
### company_id.csv

这个csv包括所有的公司股票代码，用\n隔行隔开。

### requirements.txt

需要安装的依赖都在这个文档中，打开cmd后请在所在目录下输入下面的内容：
`pip install -i requirements.txt`

### Chrome driver下载 

因为爬虫翻页使用到selenium虚拟点击，所以需要提前下载Chrome Driver并放到特定目录下面。

具体安装方法请查看： 下载地址[https://blog.csdn.net/wendyw1999/article/details/107414953]

### 载入所有的package

```python
from __future__ import unicode_literals

import time
import os, os.path
import urllib

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import pandas as pd
import re
import requests
import os, os.path
from whoosh import index
from whoosh.qparser import QueryParser
from whoosh.fields import Schema, STORED, ID, KEYWORD, TEXT
from whoosh.query import And,Term,Or

from jieba.analyse import ChineseAnalyzer

import pdfplumber
```

# 实际操作
请查看[https://github.com/wendyw1999/crawl_company_info/blob/master/pdf_crawl.ipynb](crawl_company notebook)


### initialize selenium Driver
```python
def initialize():
    '''
    初始化一个Selenium chrome driver
    '''
    options = Options()
    options.add_argument("--headless")  # 无界面
    driver = webdriver.Chrome(options=options)   
    return driver
```
### 通过rooturl+股票代码的方式找到公司的论坛页面
### 爬虫查找帖子标题中带有关键字的（社会，责任）
- 如果找到就return，如果没有找到就继续翻页
- 如果最后一页还是没有，就return False，（这就需要人工手动在百度上搜索了）

### 进入帖子，通过rooturl+帖子编码，获得pdf可下载的url地址
### 下载pdf到本地 所有的pdf会被放在pdfs文件夹内
### 创建全文索引

