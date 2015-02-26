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
#   版本： 1.3
#   作者：Cao Linjian
#   日期：2015-02-18
#   语言：Python 3.3 
#   说明：输入影人图片地址或者相册地址
#---------------------------------------

import urllib.request
import os,re,time
from bs4 import BeautifulSoup as bs #第3方库，解析html
from multiprocessing import Pool
import socket

socket.setdefaulttimeout(10)
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

def download(img):
    try:
        print(img)
        wq = urllib.request.Request(img,headers = headers)
        data = urllib.request.urlopen(wq).read()
        with open(destFile(img), 'wb') as f:
            f.write(data)
    except urllib.error.URLError as e:
        if isinstance(e.reason,socket.timeout):
            print('urllib超时重新抓取...')
            time.sleep(1)
            download(img)
    except socket.timeout as e:
        print('socket超时重新抓取...')
        time.sleep(1)
        download(img)
    except Exception as err:
        print('51')
        print(err)
        
#从当前页中获取图片地址并下载
def getImgByPage(url):
    img_urls = []
    req = urllib.request.Request(url,headers = headers)  
    webpage = urllib.request.urlopen(req)  
    data = webpage.read()
    soup=bs(data)
    div = soup.find(class_="photolst clearfix")
    if div:
        photolst = div.select(".photolst_photo")
    else:
        div = soup.find(class_="poster-col4 clearfix")
        photolst = div.select(".cover")
    for photo in photolst:
        img = photo.find('img').get('src')
        img = img.replace("/thumb/", "/photo/")    #缩略图地址改成大图
        download(img)

#根据地址类型读取相册页面
def getPagesByType(type):
    try:
        req = urllib.request.Request(hostname,headers = headers)  
        webpage = urllib.request.urlopen(req)  
        data = webpage.read()
        soup=bs(data)
        s = ''
        a = re.findall(r'[\u4e00-\u9fa5]',soup.title.text) #蛋疼的windows
        for i in a:
            s += i
        print(s)
        paginator = soup.find(class_='paginator')
        total_page = 1
        if paginator:
            total_page = int(paginator.find(class_='thispage').get('data-total-page'))
        if total_page == 1:
            getImgByPage(hostname)
        else:
            urls = []
            for i in range(total_page):
                if type == 'album':
                    urls.append(hostname+"?start="+str(i*18))
                elif type == 'celebrity':
                    urls.append(hostname+"?type=C&sortby=vote&size=a&subtype=a&start="+str(i*40))
            pool = Pool(8)
            pool.map(getImgByPage, urls)
    except Exception as err:
        print('99')
        print(err)
        
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
        getPagesByType("album")
    elif re.search(r'http://movie\.douban\.com/celebrity/\d+/photos/',hostname):  #匹配影人图片地址
        getPagesByType("celebrity")
    else:
        print('请检查地址是否有误!')
    
```

运行该文件后输入'http://www.douban.com/photos/album/18445613/' 或者 'http://movie.douban.com/celebrity/1013763/photos/'的格式的地址即可，可以不用加'http://'在相册中间的某页地址如'http://www.douban.com/photos/album/20570238/?start=18'形式的也可。
