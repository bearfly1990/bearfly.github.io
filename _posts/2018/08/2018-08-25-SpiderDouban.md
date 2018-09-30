---
layout: post
title: Spider for Douban (01)
subtitle: Top n Movies of Spider for Douban
date: 2018-08-25
author: BF
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
  - python
  - spider
---

# Top n Movies of Spider for Douban

网上有许多爬虫的入门脚本，都是去爬豆瓣前 250 的电影信息，我也参考并写了一个，做为入门。

因为豆瓣的网页结构相对稳定，变化不多，不然要花太多时候去维护。

在代码中可以修改 page 的值来取得更多的页数，我发现其实是可以超过 250 个的，排名 250 后的资源，也可以通过手动修改得到。

## Result

运行后的结果大致如下：

```txt
肖申克的救赎(9.6)
霸王别姬(9.5)
这个杀手不太冷(9.4)
阿甘正传(9.4)
美丽人生(9.5)
泰坦尼克号(9.3)
千与千寻(9.3)
辛德勒的名单(9.4)
盗梦空间(9.3)
机器人总动员(9.3)
忠犬八公的故事(9.3)
三傻大闹宝莱坞(9.2)
海上钢琴师(9.2)
放牛班的春天(9.2)
大话西游之大圣娶亲(9.2)
楚门的世界(9.1)
...
```

下载的海报如下：
![Posts](https://raw.githubusercontent.com/bearfly1990/bearfly1990.github.io/master/_posts/2018/08/imgs/2018-08-25-SpiderDouban-PostSamples.jpg)

## Code

```python
'''
author: xiche
create at: 08/24/2018
description:
    Spider for Douban
Change log:
Date        Author      Version    Description
08/24/2018  xiche       1.0        Top n Movies of Spider for Douban
08/28/2018  xiche       1.1        refactor the code
'''
import requests
import os
import codecs
import re
from contextlib import closing
from bs4 import BeautifulSoup
from abc import abstractmethod, ABC

class Spider(ABC):
    url_request = ''
    html = ''
    HEADER_BROWER = {
        'User-Agent':'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko)'
            'Chrome/63.0.3239.132 Safari/537.36'}

    def download_page(self):
        data = requests.get(self.url_request, headers=self.HEADER_BROWER).text
        self.html = data
        return data

    @abstractmethod
    def parse_html(self, html):
       pass

    @abstractmethod
    def start_spider(self):
        pass

    def getPic(self, data):
        pic_list = re.findall(r'src="http.+?.jpg"', data)
        return pic_list

    def download_pic(self, url, name, folder = 'imgs/'):
        rootPath = folder
        if not os.path.exists(rootPath):
            os.makedirs(rootPath)
        response = requests.get(url, stream=True)
        pic_type = '.' + url.split('.')[-1]
        with closing(requests.get(url, stream=True)) as responses:
            with open(rootPath + name + pic_type, 'wb') as file:
                for data in response.iter_content(128):
                    file.write(data)

class SpiderDouban250(Spider):
    start = 0
    page  = 2
    movies_list_total = []
    def __init__(self, page=2):
        self.page = page
    def parse_html(self, html=None):
        html = html if html else self.html
        soup = BeautifulSoup(html, "html.parser")
        movie_list_soup = soup.find('ol', attrs={'class': 'grid_view'})
        movie_name_list = []
        for movie_li in movie_list_soup.find_all('li'):
            detail = movie_li.find('div', attrs={'class': 'hd'})
            movie_name = detail.find('span', attrs={'class': 'title'}).getText()
            score = movie_li.find('div', attrs={'class': 'bd'})
            movie_score = score.find('span', attrs={'class': 'rating_num'}).getText()
            movie_detail = "{}({})".format(movie_name , movie_score)
            movie_name_list.append(movie_detail)

        next_page = soup.find('span', attrs={'class': 'next'}).find('a')
        if next_page:
            return movie_name_list, self.url_request + next_page['href']
        return movie_name_list, None

    def start_spider(self):
        start = self.start
        movies_list_total = []
        while(start < 25 * self.page):
            self.url_request = 'https://movie.douban.com/top250?self.start={}'.format(start)
            self.download_page()
            picdata = self.getPic(self.html)
            movies, url = self.parse_html()
            movies_list_total = movies_list_total + movies
            index_movies = 0
            for picinfo in picdata:
                self.download_pic(picinfo[5:-1], 'Top' + str(index_movies+1) + '-' + movies[index_movies])
                print(movies[index_movies] + ' download finished')
                index_movies += 1
                start += 1
        with codecs.open('movies.txt', 'w', encoding='utf-8') as fp:
            fp.write(u'\r\n'.join(movies_list_total))

if __name__ == '__main__':
    spider_douban = SpiderDouban250(2)
    spider_douban.start_spider()
```
