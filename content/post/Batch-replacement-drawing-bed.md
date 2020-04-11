---
title: "Batch Replacement Drawing Bed"
date: 2020-04-11T16:56:14+08:00
draft: flase
tags: [图床工具]
categories: [python脚本]
comment: true
---
 批量替换图床地址
<!--more-->

**由于在运营ChaBug公众号中，部分图床需要翻墙，而公众号发出去之后不能看到，于是写了个小脚本批量替换为自己的图床（可以关注一下公众号：ChaBug）**

- Code

```
#!/usr/bin/python3
# -*- coding: utf-8 -*-
# auther = syst1m

import re
import os 
import requests
from qcloud_cos import CosConfig
from qcloud_cos import CosS3Client
from qcloud_cos import CosServiceError
from qcloud_cos import CosClientError

secret_id = ''  # 替换为用户的secret_id
secret_key = ''  # 替换为用户的secret_key
region = ''  # 替换为用户的region
token = None  # 使用临时密钥需要传入Token，默认为空,可不填
scheme = 'https'
config = CosConfig(Region=region, SecretId=secret_id, SecretKey=secret_key, Token=token, Scheme=scheme)  # 获取配置对象
client = CosS3Client(config)

def GetImage(name):
    f = open(name,'r')
    data = f.read()
    ImageRule = re.compile(r'!\[.*?\]\((.*?)\)')
    ImageList = ImageRule.findall(data)
    return ImageList

def UploadImage(name):
    ImageList = GetImage(name)
    imagedict = {}
    for i in ImageList:
        response = requests.get(i)
        img = response.content
        file = i.split('/')[-1]
        with open(file,'wb') as f:
            f.write(img)
            response = client.upload_file(
                Bucket='', #存储桶名称
                LocalFilePath='/Users/syst1m/code/{0}'.format(file),
                Key=file,         
                PartSize=10,
                MAXThread=10,
            )
            os.remove(file)
            imagedict["![image.png]("+i+")"] = "![image.png](https://maekdown-XXX.cos.ap-beijing.myqcloud.com/"+file+")"
    return imagedict

def ReplaceMd(name):
    imagedict = UploadImage(name)
    f = open(name,'r')
    data = f.read()
    f.close()
    for key,value in imagedict.items():
        data = data.replace(key,value)
        lastmd = open(name,'w')
        lastmd.write(data)
        lastmd.close()

if __name__ == "__main__":
    name = input("请输入需要转换的Markdown文档名称：\n")
    print("正在更换MarkDown文档中的图床，请稍等哦亲！")
    ReplaceMd(name)
    print("更换完毕，想干嘛干嘛吧。")
```