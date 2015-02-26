title: python3豆瓣爬虫
date: 2015-02-18 23:01:22
categories: python
tags: [python,python3,爬虫]
---
大年30没事干，也抢不到红包，于是写写代码娱乐娱乐，复习复习最近学的python。

源码如下：
<!-- more -->
```python
# -*- coding: utf-8 -*-

#---------------------------------------  
#   程序：豆瓣相册爬虫
#   版本： 1.0
#   作者：Cao Linjian
#   日期：2015-02-18
#   语言：Python 3.3 
#   说明：输入影人图片地址或者相册地址
#---------------------------------------

import urllib.request
import os,re,time
from bs4 import BeautifulSoup as bs #第3方库，解析html
from multiprocessing import Pool

abs = os.path.abspath('.')
targetDir = os.path.join(abs,'pic')
if not os.path.isdir(targetDir):  
    os.mkdir(targetDir) 
headers = {
        'User-Agent':'Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.6)'
        ' Gecko/20091201 Firefox/3.5.6'
    }

#生成的文件名
def destFile(path):        
    pos = path.rindex('/')  
    t = os.path.join(targetDir, path[pos+1:])  
    return t  
 
#从当前页中获取图片地址并下载
def getImgByPage(url):
    img_urls = []
    req = urllib.request.Request(url,headers = headers)  
    webpage = urllib.request.urlopen(req)  
    data = webpage.read()
    soup=bs(data)
    div = soup.find(class_="photolst clearfix")  #相册页面
    if div:
        photolst = div.select(".photolst_photo")
    else:
        div = soup.find(class_="poster-col4 clearfix")  #影人图片页面
        photolst = div.select(".cover")
    for photo in photolst:
        img = photo.find('img').get('src')
        img = img.replace("/thumb/", "/photo/")    #缩略图地址改成大图
        print(img)
        wq = urllib.request.Request(img,headers = headers)
        data = urllib.request.urlopen(wq).read()
        with open(destFile(img), 'wb') as f:
            f.write(data) 
        
        
if __name__ == "__main__": 
    '''
    hostname = "http://www.douban.com/photos/album/18445613/" 
    hostname = "http://movie.douban.com/celebrity/1013763/photos/" 
    这两种格式都行╮(￣▽￣")╭ 
    '''
    hostname = input("请输入豆瓣相册地址： \n")
    if hostname.find('http:') < 0:
        hostname = 'http://'+hostname
    if hostname.find('?') > 0:
        hostname = hostname[0:hostname.find('?')]
    if hostname[:-1] != '/':
        hostname = hostname+"/"
    if re.search(r'http://www\.douban\.com/photos/album/\d+/',hostname):  #匹配相册地址
        try:
            req = urllib.request.Request(hostname,headers = headers)  
            webpage = urllib.request.urlopen(req)  
            data = webpage.read()
            soup=bs(data)
            print(soup.title)
            paginator = soup.find(class_='paginator')
            total_page = 1
            if paginator:
                total_page = int(paginator.find(class_='thispage').get('data-total-page'))
            if total_page == 1:
                getImgByPage(hostname)
            else:
                urls = []
                for i in range(total_page):
                    urls.append(hostname+"?start="+str(i*18))
                pool = Pool(4)
                pool.map(getImgByPage, urls)
        except Exception as err:
            print(err)
    elif re.search(r'http://movie\.douban\.com/celebrity/\d+/photos/',hostname):  #匹配影人图片地址
        try:
            req = urllib.request.Request(hostname,headers = headers)  
            webpage = urllib.request.urlopen(req)  
            data = webpage.read()
            soup=bs(data)
            print(soup.title)
            paginator = soup.find(class_='paginator')
            total_page = 1
            if paginator:
                total_page = int(paginator.find(class_='thispage').get('data-total-page'))
            if total_page == 1:
                getImgByPage(hostname)
            else:
                for i in range(total_page):
                    getImgByPage(hostname+"?type=C&sortby=vote&size=a&subtype=a&start="+str(i*40))
           
        except Exception as err:
            print(err)
    else:
        print('请检查地址是否有误!')
    
```

使用了`Pool`来实现多进程，感觉速度提升不大，而且很容易造成假死(இдஇ;），待解决。影人图片部分已经去掉多进程了，不过也会出现假死。
运行该文件后输入'http://www.douban.com/photos/album/18445613/' 或者 'http://movie.douban.com/celebrity/1013763/photos/'的格式的地址即可，可以不用加'http://'在相册中间的某页地址如'http://www.douban.com/photos/album/20570238/?start=18'形式的也可。
