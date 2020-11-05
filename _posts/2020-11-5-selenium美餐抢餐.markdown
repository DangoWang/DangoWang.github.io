---
layout:     post
title:      "selenium+chrome抢餐"
subtitle:   " \"手快有，手慢无\""
date:       2020-11-5 16:40:00
author:     "DD"
header-img: "img/post-bg-alitrip.jpg"
catalog: true
tags:
    - 美餐
    - selenium
---
> 我中毒了
<iframe src="//player.bilibili.com/player.html?aid=584724198&bvid=BV1rz4y1Z7iU&cid=238677930&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

这篇记录一下美餐爬虫+抢餐的脚本开发过程。最后没有写点餐的动作，怕被公司it搞:tw-1f480:

#### meican
之前用github上的开源项目[meican](https://github.com/LKI/meican)来点餐，但是发现美餐有反爬虫限制，每次刷个二三十次就被反爬虫了。遂放弃。
### selenium
selenium 是一个 web 的自动化测试工具，具体介绍[在这里](https://www.jianshu.com/p/1531e12f8852).它可以前台（或后台）打开浏览器，爬取页面内容。
### 代码
```python
# coding: utf-8
import datetime
import time
import json
import os
from bs4 import BeautifulSoup
from selenium import webdriver
import logging
import requests
from selenium.webdriver.common.keys import Keys

logger = logging.getLogger(__name__)
logger.setLevel(level=logging.INFO)
log_dir_name = os.dirname(__file__)+'/log/'
log_file_name = log_dir_name + "%s.log" % str(datetime.datetime.today()).split()[0]
handler = logging.FileHandler(log_file_name)
handler.setLevel(logging.INFO)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)


login_url = 'https://meican.com/accounts/login'
calendar_url = 'https://meican.com/preorder/api/v2.1/calendarItems/list?' \
               'beginDate={begin}&endDate={end}&noHttpGetCache={cache}' \
               '&withOrderDetail=False'

restaurants_url = 'https://meican.com/preorder/api/v2.1/restaurants/list?' \
              'noHttpGetCache={}&tabUniqueId={}&targetTime={}'

wechat_url = 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=84ab2208-4ac1-423e-ace8-43c9a11386a0' #  发现可以抢餐之后发到企业微信机器人通知大家
wechat_data = {
                  "msgtype": "markdown",
                  "markdown": {
                    "content": u"[鱼酷可以点啦！！！](https://meican.com/)\n"*5
                  }
                }
mention_data = {
                "msgtype": "text",
                "text": {
                    "content": u"可以点餐啦！！",
                    "mentioned_list":["@all"],
                    }
                }



def login(user, password):
    chrome_options = webdriver.ChromeOptions()
    # 使用headless无界面浏览器模式
    chrome_options.add_argument('--headless')
    chrome_options.add_argument('--disable-gpu')
    driver = webdriver.Chrome(chrome_options=chrome_options)
    driver.get(login_url)
    driver.find_element_by_name('username').clear()
    driver.find_element_by_name('username').send_keys(user)
    driver.find_element_by_class_name('next').click()
    driver.find_element_by_name('password').clear()
    driver.find_element_by_name('password').send_keys('Adong123')
    driver.find_element_by_name('password').send_keys(Keys.ENTER)
    time.sleep(3)
    driver.find_element_by_class_name('confirm').click()
    logger.info(u'成功登录')
    return driver


def get_calendar_item_list(driver, url):
    driver.get(url)
    html = driver.page_source
    soup = BeautifulSoup(html, 'lxml')
    items = json.loads(soup.text)
    logger.info(u'成功拉取日历')
    return items


def get_restaurants(driver, url):
    driver.get(url)
    html = driver.page_source
    soup = BeautifulSoup(html, 'lxml')
    items = json.loads(soup.text)
    logger.info(u'成功拉取餐厅')
    return items


if __name__ == '__main__':
    driver = login('username', 'password')

    time_delta = datetime.timedelta(5)

    date = (datetime.datetime.today() + time_delta).strftime('%Y-%m-%d')
    calendar_url = calendar_url.format(**dict(begin=date, end=date, cache=int(time.time() * 1000)))
    calendar_item_list = get_calendar_item_list(driver, calendar_url)

    while 1:
        if calendar_item_list['dateList'][0].get('calendarItemList')[0].get('status') in ['AVAILABLE', 'ORDER']:
            #  可以点餐了
            unique_id = calendar_item_list['dateList'][0].get('calendarItemList')[0].get('userTab').get('uniqueId')
            target_time = (datetime.datetime.today() + time_delta).strftime('%Y-%m-%d+%H') + '%3A00%3A00%2B08%3A00'
            restaurants_url = 'https://meican.com/preorder/api/v2.1/restaurants/list?noHttpGetCache={}&tabUniqueId={}&targetTime={}' \
                .format(int(time.time() * 1000), unique_id, target_time)
            restaurants = get_restaurants(driver, restaurants_url)
            if u'鱼酷' in str(restaurants):
                print(str(datetime.datetime.now()) + u'可以点鱼酷了!')
                logger.info(u'可以点鱼酷了')
                requests.post(wechat_url, json.dumps(wechat_data))
                requests.post(wechat_url, json.dumps(mention_data))
                driver.quit()
                break
            else:
                #  可以点餐，但还没有鱼酷
                print(str(datetime.datetime.now()) + u'可以点餐，但还没有鱼酷.')
                logger.info(u'可以点餐，但还没有鱼酷.')
                time.sleep(2)
                continue
        else:
            #  还不能点餐
            print(str(datetime.datetime.now()) + u'还不能点餐')
            logger.info(u'还不能点餐')
            time.sleep(30)
            driver.refresh()
```

