title: Python爬虫
author: Kevin Zhou
tags:
  - Python
  - 爬虫
categories:
  - 爬虫
date: 2018-08-20 22:12:00
---
### 如何使用requests登录豆瓣并且爬取内容
Note:
1.如果登录之后要去其他页面查看相关内容得记录session
```python
   s=requests.session()
  r = s.post(loginUrl, data=formData, headers=headers
  res=s.get("http://movie.douban.com/mine",cookies=r.cookies,headers=headers)
```
2.r.history可以记录login之后的302 status
<!--more-->
Code:
```python
# -*- encoding:utf-8 -*-  
##############################  
__author__ = "KevinZhou"
__date__ = "2017/7/23"
###############################  

import requests
from bs4 import BeautifulSoup
import urllib.request
import re

loginUrl = 'https://accounts.douban.com/login'
formData = {
    "redir": "http://movie.douban.com/mine",
    "form_email": "******",
    "form_password": "******",
    "login": u'登录',
    "source":"index_nav"
}
headers = {'user-agent': 'Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36'}

r = requests.post(loginUrl, data=formData, headers=headers)
page = r.text
print (r.url)

'''''获取验证码图片'''
# 利用bs4获取captcha地址
soup = BeautifulSoup(page, "html.parser")
captchaAddr = soup.find('img', id='captcha_image')['src']
# 利用正则表达式获取captcha的ID
# reCaptchaID = r'<input type="hidden" name="captcha-id" value="(.*?)"/'
# captchaID = re.findall(reCaptchaID, page)

# htm=requests.get("https://accounts.douban.com/login")
#print(htm.content.decode('utf-8'))
a=soup.find("img",class_="captcha_image")
print(a.attrs['src'])
pattern = re.compile(r'id=(\w*)(\W)en&size=(\w*)$')
match=pattern.search(a.attrs['src'])
if match:
    captchaID=match.group()[3:][:-7]
    print(captchaID)

# print captchaID
# 保存到本地
urllib.request.urlretrieve(captchaAddr,"captcha.jpg")
captcha = input('please input the captcha:')

formData['captcha-solution'] = captcha
formData['captcha-id'] = captchaID
s=requests.session()
r = s.post(loginUrl, data=formData, headers=headers)

page = r.text
print(r.url)
print(r.history)
# print(r.cookies)
# print(r.content.decode('utf-8'))
# res=s.get("http://movie.douban.com/mine",cookies=r.cookies,headers=headers)
#print(res.content.decode('utf-8'))
if r.url == 'https://movie.douban.com/mine':
    print('Login successfully!!!')
print
'我看过的电影', '-' * 60
# 获取看过的电影
soup = BeautifulSoup(page, "html.parser")
result = soup.findAll('li')
for item in result:
    s=item.find('a', class_="cover")
    if s is not None:
        # print(s)
        print(s.get("href"))
        print(s.img["alt"])
        # for img in  s:
        #     print(img["alt"])
        #     print(img.alt)

else:
    print
"failed!"
```