title: 爬虫抓取代理IP
keywords: '爬虫,代理IP'
date: 2018-1-17 21:03:04
categories: 爬虫 
tags: 爬虫
description:
---
<blockquote class="blockquote-center">
由于某些网站对会对爬虫做限制，因此常常需要通过代理将爬虫的实际IP隐蔽起来，代理也有分类，如透明代理，高匿代理等。本文主要讲述如何获取代理IP，并且如何存储和使用。
</blockquote>



<!--more-->

 某些网站会免费提供代理IP，如下面的几个
获取这些页面上的代理IP及端口也是通过爬虫抓取，下面以第一个网站http://www.xicidaili.com为例，解释如何获取并存储这些代理IP。一般的流程为：解析当前页面-->存储当前页面的代理IP-->跳转到下一页面，重复该流程即可。
##解析页面
首先要解析页面，由于网页中显示代理IP时是在表格中显示的，因此只需要通过找出网页源码中相关的表格元素即可。下面是通过python中的requests和bs4获取页面http://www.xicidaili.com/nt/上显示的IP及端口。
  ```
  import requests
from bs4 import BeautifulSoup

user_agent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.106 Safari/537.36'
referer = 'http://www.xicidaili.com/'
headers = {'user-agent': user_agent, 'referer': referer}
target = 'http://www.xicidaili.com/nt/'

//获取页面源码
r = requests.get(target, headers = headers)
//解析页面源码
soup = BeautifulSoup(r.text, 'lxml')
for tr in soup.find_all('tr')[1:]:
tds = tr.find_all('td')
proxy = tds[1].text+':'+tds[2].text
print proxy
  ```

输出如下：

  ```
  36.235.1.189:3128
219.141.225.149:80
125.44.132.44:9999
123.249.8.100:3128
183.54.30.186:9999
110.211.45.228:9000
...........
  ```


###代理IP的存储
上面代码获取的代理IP可以通过在代码一开始建立一个集合（set）来存储，这种情况适用于一次性使用这些代理IP，当程序发生异常或正常退出后，这些存储在内存中的代理IP也会丢失。但是爬虫中使用代理IP的情况又是非常多的，所以有必要把这些IP存储起来，从而可以让程序多次利用。
这里主要通过redis数据库存储这些代理IP，redis是一个NOSQL数据库，具体使用参照官方文档，这里不做详细解释。
下面是ConnectRedis.py文件，用于连接redis
  ```
import redis

HOST = 'XXX.XXX.XXX.XXX' # redis所在主机IP
PORT = 6379 # redis服务监听的端口
PASSWORD = 'XXXXXX' # 连接redis的密码
DB = 0 # IP存储的DB编号

def get_connection():
r = redis.Redis(host = HOST, port = PORT, password = PASSWORD, db= DB)
return r
  ```
下面是在上面的代码基础上将IP存储到redis中，

  ```
import requests
from bs4 import BeautifulSoup
from ConnectRedis import get_connection
//获取redis连接
try:
conn = get_connection()
except Exception:
print 'Error while connecting to redis'
return

user_agent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.106 Safari/537.36'
referer = 'http://www.xicidaili.com/'
headers = {'user-agent': user_agent, 'referer': referer}
target = 'http://www.xicidaili.com/nt/'

//获取页面源码
r = requests.get(target, headers = headers)
//解析页面源码
soup = BeautifulSoup(r.text, 'lxml')
for tr in soup.find_all('tr')[1:]:
tds = tr.find_all('td')
proxy = tds[1].text+':'+tds[2].text
conn.sadd("ip_set", proxy)
print '%s added to ip set'%proxy
  ```
  
上面的conn.sadd("ip_set", proxy)将代理proxy加入到redis的集合"ip_set"，这个集合需要预先在redis中创建，否则会出错。
###页面跳转
上面的代码获取的只是一个页面上显示的代理，显然这个数量不够，一般通过当前页面中的下一页的超链接可以跳转到下一页，但是我们测试的由于每页的的url都有规律，都是http://www.xicidaili.com/nt/page_number,其中的page_number表示当前在哪一页，省略时为第一页。因此，通过一个for循环嵌套上面的代码即可获取多个页面的代理。但是更一般的方法是通过在当前页面获取下一页的超链接而跳转到下一页。
###代理IP的使用
当我们需要通过代理访问某一网站时，首先需要从redis中随机选出一个代理ip，然后尝试通过代理ip是否能连到我们需要访问的目标网站，因为这些代理IP是公共使用的，所以往往也会被封的很快，假如通过代理无法访问目标网站，那么就要从数据库中删除这个代理IP。反之即可通过此代理访问目标网站
下面是实现上面所说流程的代码：
  ```
import requests
from ConnectRedis import get_connection

//判断IP是否能访问目标网站
def is_valid(url, ip):
proxy = {
'http': 'http://%s' %ip
}
user_agent = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.106 Safari/537.36'
headers = {'user-agent': user_agent}
try:
r = requests.get(url, headers = headers, proxies = proxy, timeout = 6)
return True
except Exception:
return False

if __name__ == '__main__':
my_proxy, proxies, ip_set = None, None, 'amazon_ips'
conn = get_connection()
target = 'https://www.amazon.com/'
while not is_valid(target, my_proxy):
if my_proxy:
conn.srem(ip_set, my_proxy) #删除无效的代理IP
if proxies:
my_proxy = proxies.pop()
else:
proxies = conn.srandmember(ip_set, 5) #从redis中随机抽5个代理ip
my_proxy = proxies.pop()
print 'valid proxy %s' %my_proxy
  ```
requests.get(url, headers = headers, proxies = proxy, timeout = 6)是通过代理去访问目标网站，超时时间设为6s，也就是说在6秒内网站没有回应或返回错误信息就认为这个代理无效。
除此之外，在爬取免费提供代理的网站上的代理IP的时候，爬取的速度不要太快，其中的一个原因是爬取太快有可能会被封，另外一个原因是如果每个人都无间隙地从这种网站上爬取，那么网站的负担会比较大，甚至有可能垮掉，因此采用一个可持续爬取的策略非常有必要，我爬取的时候是没爬完一个页面后让程序sleep大概2分钟，这样下来不会被封而且爬取的代理的量也足够使用。实际中可以根据自己使用代理的频率来进行调整。

