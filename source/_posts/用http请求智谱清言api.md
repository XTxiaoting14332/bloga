---
title: 用http请求智谱清言api
date: 2024-03-20 12:45:02
tags:
- ChatGLM
- http请求
---

智谱ai开放平台已经给我们提供了SDK可以直接使用，详见[这里](https://open.bigmodel.cn/dev/api#sdk "点我前往")
<br>
按照文档的教程，我们可以在安装好sdk后直接进行调用：

```
from zhipuai import ZhipuAI
 
client = ZhipuAI(api_key="") # 请填写您自己的APIKey
response = client.chat.asyncCompletions.create(
    model="glm-4",  # 填写需要调用的模型名称
    messages=[
        {
            "role": "user",
            "content": "请你作为童话故事大王，写一篇短篇童话故事，故事的主题是要永远保持一颗善良的心，要能够激发儿童的学习兴趣和想象力，同时也能够帮助儿童更好地理解和接受故事中所蕴含的道理和价值观。"
        }
    ],
)
print(response)
```

**但是**，智谱的SDK使用的``pydantic``版本大于等于``2.5.2``<br><br>
在某些需要``pydantic 1.x``的环境下~~(比如Nonebot)~~就无法使用智谱的SDK
所以这时候，我们就需要用requests（或者httpx），来对url发送请求<br>

先安装下依赖

```
pip install PyJWT
```

# 单次调用
示例代码

```
import jwt
from datetime import datetime, timedelta
import time
import httpx #当然这里用requests也可以，用法一样的
#生成JWT
def generate_jwt(apikey: str):
    try:
        id, secret = apikey.split(".")
    except Exception as e:
        raise Exception("错误的apikey！", e)

    payload = {
        "api_key": id,
        "exp": datetime.utcnow() + timedelta(days=1),
        "timestamp": int(round(time.time() * 1000)),
    }

    return jwt.encode(
        payload,
        secret,
        algorithm="HS256",
        headers={"alg": "HS256", "sign_type": "SIGN"},
    )


#请求AI
def req_glm(auth_token):
    headers = {
        "Authorization": f"Bearer {auth_token}"
    }
    data = {
        "model": "这里填你要用的模型名称",
        "messages": [        
            {
            "role": "user",
            "content": "你好" #这里填你的消息内容
         }
        ]
    }

    timeout = httpx.Timeout(connect=10, read=60, write=None, pool=None)
    res = httpx.post("https://open.bigmodel.cn/api/paas/v4/chat/completions", headers=headers, json=data, timeout=timeout)
    res = res.json()
    try:
        #这会将api返回的信息提取出来
        res_raw = res['choices'][0]['message']['content']
    except Exception as e:
        #如果请求失败，就返回原json数据
        res_raw = res
    return res_raw

api = "你的api_key" #这里填你的api_key


auth = generate_jwt(api)
res = str(req_glm(auth))

print(res) #输出结果
```
<br>
将api_key替换成你自己的就可以进行使用

运行：
```
$ python3 demo.py 
你好👋！我是人工智能助手智谱清言，可以叫我小智🤖，很高兴见到你，欢迎问我任何问题。
```

<br>

# 实现上下文对话
智谱api的上下文主要是通过以下的形式来实现（来自官方文档）

```
    messages=[
        {"role": "user", "content": "作为一名营销专家，请为我的产品创作一个吸引人的slogan"},
        {"role": "assistant", "content": "当然，为了创作一个吸引人的slogan，请告诉我一些关于您产品的信息"},
        {"role": "user", "content": "智谱AI开放平台"},
        {"role": "assistant", "content": "智启未来，谱绘无限一智谱AI，让创新触手可及!"},
        {"role": "user", "content": "创造一个更精准、吸引人的slogan"}
    ]
```

其中``user``是用户向api发送的消息，``assistant``为api返回给用户的消息，那么我们要做的，就是将api返回的消息储存至一个文件中，在对话时读取<br>

示例代码

```
import jwt
import httpx #当然这里用requests也可以，用法一样的
from datetime import datetime, timedelta
import time
import json,os




#聊天记录实现
#用户输入
def user_in(text):
    if os.path.exists("history.json"):
        with open('history.json', 'a') as file:
            file.write(',\n{"role": "user", "content": "' + text + '"}')
    else:
        with open('history.json', 'w') as file:
            file.write('{"role": "user", "content": "' + text + '"}')

#AI输出
def ai_out(text):
    with open('history.json', 'a') as file:
        file.write(',\n{"role": "assistant", "content": "' + text + '"}')

#生成JWT
def generate_jwt(apikey: str):
    try:
        id, secret = apikey.split(".")
    except Exception as e:
        raise Exception("错误的apikey！", e)

    payload = {
        "api_key": id,
        "exp": datetime.utcnow() + timedelta(days=1),
        "timestamp": int(round(time.time() * 1000)),
    }

    return jwt.encode(
        payload,
        secret,
        algorithm="HS256",
        headers={"alg": "HS256", "sign_type": "SIGN"},
    )


#请求AI
def req_glm(auth_token,usr_message):
    headers = {
        "Authorization": f"Bearer {auth_token}"
    }
    data = {
        "model": "这里填你要用的模型名称",
        "messages": usr_message
    }

    timeout = httpx.Timeout(connect=10, read=60, write=None, pool=None)
    res = httpx.post("https://open.bigmodel.cn/api/paas/v4/chat/completions", headers=headers, json=data, timeout=timeout)
    res = res.json()
    try:
        #这会将api返回的信息提取出来
        res_raw = res['choices'][0]['message']['content']
    except Exception as e:
        #如果请求失败，就返回原json数据
        res_raw = res
    return res_raw

api = "你的api_key" #这里填你的api_key


auth = generate_jwt(api)
msg = input("请输入内容：")
user_in(msg)
with open(f'history.json', 'r') as file:
                history = file.read()
#将聊天记录进行拼接
history = str(history)
history = f"""
[
{history}
]
"""
history = json.loads(history)
try:
    res = str(req_glm(auth,history))
    res_raw = res.replace("\n","\\n") #转义，避免api返回换行导致炸json
    ai_out(res_raw)
    print(res) #输出结果
except httpx.HTTPError as e:
    res = f"请求接口出错～\n返回结果：{e}"
    print(res)
```

运行代码：
```
$ python3 demo.py 
请输入内容：你好
你好👋！我是人工智能助手智谱清言，可以叫我小智🤖，很高兴见到你，欢迎问我任何问题。
$ python3 demo.py 
请输入内容：1+1=?
1+1=2。这是基本的算术加法运算。
$ python3 demo.py 
请输入内容：我的上一个问题是什么
你的上一个问题是："1+1=?"。
```

这时候我们可以发现当前的目录下多了一个``history.json``，这个就是我们储存聊天记录的地方，打开可以看到我刚才的聊天记录都在这里

```
{"role": "user", "content": "你好"},
{"role": "assistant", "content": "你好👋！我是人工智能助手智谱清言，可以叫我小智🤖，很高兴见到你，欢迎问我任何问题。"},
{"role": "user", "content": "1+1=?"},
{"role": "assistant", "content": "1+1=2。这是基本的算术加法运算。"},
{"role": "user", "content": "我的上一个问题是什么"},
{"role": "assistant", "content": "你的上一个问题是："1+1=?"。"}
```