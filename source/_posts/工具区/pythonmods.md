---
title: 挺常用的python小组件
date: 2023-07-25 15:00:06
tags: ['工具区','Python']
category: ['工具区','Python']
toc: true
---

## 观前提醒
这些都是我**个人**在学习python的时候所自己写的小组件，所以代码难免会比较基础，或者说只是勉强能跑，所以请大家酌情使用，然后如果有更好的思路麻烦评论或者Email联系一下我qwq




### 爬m3u8视频
```python

import requests
from concurrent.futures import ThreadPoolExecutor
from os import system

def combine_del(name):
    # 合并文件
    system(f'cd ./video_save & copy/b *.ts {name}.mp4')
    # 删除文件
    system('cd ./video_save & Del *.ts')


# 下载m3u8文件保存到脚本根目录的m3u8_save中             [过渡使用]
def get_m3u8(url, name, headers):
    M_temp = requests.get(url, headers=headers)
    m3u8 = M_temp.content
    # 保存m3u8文件
    with open('./m3u8_save/{}.txt'.format(name), 'wb') as file:
        file.write(m3u8)
        file.flush()
        print(name, 'm3u8提取成功')


# 下载一个ts所用的函数，保存到video_save中
def down_ts_one(url, name, headers):
    print(url)
    M_temp = requests.get(url,headers=headers,allow_redirects=False)
    M_video = M_temp.content
    with open('./video_save/{}.ts'.format(name), 'wb') as file:
        file.write(M_video)
    print(name,'提取成功')


# 多线程下载m3u8视频且合并
def download_video(url,name,headers,pattern):
    tick = 1

    # 下载一个m3u8文件
    get_m3u8(url, name ,headers)

    # 读取m3u8文件
    with open(f'./m3u8_save/{name}.txt',mode='r',encoding='utf-8') as file:

      # 定义提取m3u8内视频分片的正则表达式
        m3u8_pattern = re.compile(f'r{pattern}')

        # 打开文件
        a = file.read()

        # 用正则表达式提取m3u8内视频连接
        res_m3u8 = m3u8_pattern.findall(a)
        print(res_m3u8)

    # 然后多线程进行下载
    with ThreadPoolExecutor(128) as th:
        for i in res_m3u8:
            th.submit(down_ts_one,url=i,name=str(tick).zfill(4))
            tick += 1
    # 合并从m3u8下载的分片
    print('\nm3u8全部完成')
    combine_del(name)
    print('合并完成',name)



    # 单独使用时调用范例：
    #
    # for i in 集数:
    #     download_video(你m3u8的地址,i,headers=headers,pattern=pattern)
    #     tick1 -= 1


```



### 爬取m3u8视频（原理同上，上面不行再下面）
```python
import urllib.request
from concurrent.futures import ThreadPoolExecutor
from os import system
import re

def combine_del(name):

    # 合并文件
    system(f'cd ./video_save & copy/b *.ts {name}.mp4')

    # 删除文件
    system('cd ./video_save & Del *.ts')


# 下载m3u8文件保存到脚本根目录的m3u8_save中             [过渡使用]
def get_m3u8(url, name, headers):

    M_temp = urllib.request.Request(url,headers=headers)
    m3u8 = urllib.request.urlopen(M_temp).read()

    with open('./m3u8_save/{}.txt'.format(name), 'wb') as file:
        file.write(m3u8)
        file.flush()
        print(name, 'm3u8提取成功')


# 下载一个ts所用的函数，保存到video_save中
def down_ts_one(url, name):
    urllib.request.urlretrieve(url,filename=f"./video_save/{name}.ts")
    print(name,'提取成功')

# 多线程下载m3u8视频且合并
def download_video(url,name,headers,pattern):
    tick = 1
    get_m3u8(url, name ,headers)

    # 打开m3u8文件
    with open(f'./m3u8_save/{name}.txt',mode='r',encoding='utf-8') as file:
        m3u8_pattern = re.compile(r"{}".format(pattern))
        a = file.read()
        res_m3u8 = m3u8_pattern.findall(a)
        print(res_m3u8)

    # 多线程下载
    with ThreadPoolExecutor(128) as th:
        for i in res_m3u8:
            th.submit(down_ts_one,url=i,name=str(tick).zfill(4))
            tick += 1

    print('\nm3u8全部完成')
    combine_del(name)
    print('合并完成',name)


```

### 两张图片的简单比较
```Python
# 需要OpenCV4.5以及pyautogui两个库，用来比对两张图片相似度并且给出返回值，这个组件可能代码过于基础或者落后，所以高版本的OpenCV没办法用qwq(其实是我没学)

import cv2
import pyautogui
# 是的，就一个函数
def MatchIMG(mu_ban,weight,height,size_wei,size_hei):
    # mu_ban = cv2.imread('2.png')  //测试用
    # 先按照设定的锚点以及宽高来截图
    pyautogui.screenshot(region=(weight,height,size_wei,size_hei)).save('shot_2.png')
    # 读取这张截图
    shot_ = cv2.imread('shot_2.png')
    # 使得截图与模板图进行匹配
    res = cv2.matchTemplate(shot_,mu_ban,cv2.TM_SQDIFF_NORMED)
    # 得出匹配的值，具体看终端输出
    upper_ = cv2.minMaxLoc(res)[0]
    return upper_

```

### python调用aira2下载
是一位大佬改版过的pyaira2
调用的时候直接输入下面一行就ok了
```python
from pyaira2 import jsonrpc
```
```Python

#!/usr/bin/env python
# coding=utf-8


import json
import requests


class Jsonrpc(object):

    MUTI_METHOD = 'system.multicall'
    ADDURI_METHOD = 'aria2.addUri'

    def __init__(self, host, port, token=None):
        self._idCount = 0
        self.host = host
        self.port = port
        self.serverUrl = "http://{host}:{port}/jsonrpc".format(**locals())

    def _genParams(self, method , uris=None, options=None, cid=None):
        p = {
            'jsonrpc': '2.0',
            'id': self._idCount,
            'method': method,
            'test': 'test',
            'params': []
        }
        if uris:
            p['params'].append(uris)
        if options:
            p['params'].append(options)
        return p

    def _post(self, action, params, onSuccess, onFail=None):
        if onFail is None:
            onFail = Jsonrpc._defaultErrorHandle
        paramsObject = self._genParams(action, *params)
        resp = requests.post(self.serverUrl, data=json.dumps(paramsObject))
        result = resp.json()
        if "error" in result:
            return onFail(result["error"]["code"], result["error"]["message"])
        else:
            return onSuccess(resp)

    def addUris(self, uri, options=None):
        def success(response):
            return response.text
        return self._post(Jsonrpc.ADDURI_METHOD, [[uri,], options], success)


    @staticmethod
    def _defaultErrorHandle(code, message):
        print ("ERROR: {},{}".format(code, message))
        return None

```


### python调用motrix进行下载
不是我写的，但是很久之前看到的，原贴不知道去哪了，就分享出来吧，万一以后用得到呢（）
```Python
rpc_token = 'Y6YOKzhKasoo'  
video_url = f'{url}'  # 下载地址
output_name = f'{name}.mp4'  # 下载文件名
s = client.ServerProxy('http://localhost:16800/rpc')
s.aria2.addUri('token:' + rpc_token, [video_url], dict(out=output_name))

```
