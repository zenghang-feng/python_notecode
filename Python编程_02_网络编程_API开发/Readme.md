（文章来自2023.03.13的博客）  

&emsp;&emsp;在一些场景下，需要把算法或者机器学习模型进行封装，为其他程序调用。本文采用 Python 的 Flask 模块实现了几种不同的API创建、调用的方式。
&emsp;&emsp;API：应用程序接口（Application Programming Interface，简称：API），主要目的是提供应用程序与开发人员以访问一组例程的能力，而又无需访问源码，或理解内部工作机制的细节。
&emsp;&emsp;API可以看作一个服务器，调用API的程序可以看作是客户端，客户端向API发送请求，然后获取希望得到的数据。
    ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/59496603bce6059a29ae721205055196.png)
&emsp;&emsp;下文使用Flask模块创建API，使用Requsets模块作为客户端向API发送请求。
# 1、基于GET的带参数请求
## 1.1 API（服务端）
&emsp;&emsp;首先是基于GET的带参数请求（GET和POST的区别可以参考：https://www.zhihu.com/question/28586791 ），这里GET请求中的参数是以字典的格式存储，创建API的代码如下：

```python
# 导入需要的模块
from flask import Flask, request

app = Flask(__name__)                              # 应用实例
@app.route("/", methods=['GET'])                   # 路由
def hello():                                       # 视图函数：可以在内部添加自定义的计算逻辑

    # https://dormousehole.readthedocs.io/en/latest/quickstart.html?highlight=request.args.get#id16
    the_text = request.args.get('name', '')        # 获取get请求参数
    """
    这里可以添加函数内部的计算逻辑
    """
    return '你好,' + the_text                       # 返回response

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=9999)           # 启动服务器
```

## 1.2 客户端
&emsp;&emsp;客户端发送请求的代码如下：

```python
# -*- coding:utf-8 -*-

import requests

url = "http://127.0.0.1:9999/"
# https://requests.readthedocs.io/en/latest/user/quickstart/

# 通过GET发送带参数的请求
params = {'name':'tom'}
p = requests.get(url=url, params=params)
print(p.text)
```

&emsp;&emsp;先执行服务端程序，再执行客户端程序，客户端得到的返回数据如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/63345c1de0b1668e5040cf657a3823e5.png)

# 2、基于POST的带JSON格式参数请求
## 2.1 API（服务端）
&emsp;&emsp;JSON格式在Python里面可以作为字典处理，创建API的代码如下：

```python
from flask import Flask, request

app = Flask(__name__)
@app.route("/", methods=['POST'])
def hello():

    the_text = request.json['name']
    """
    这里可以添加函数内部的计算逻辑
    """
    return '你好,' + the_text  # 返回response

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=9999)           # 启动服务器
```
## 2.2 客户端
&emsp;&emsp;客户端的代码如下：

```python
# -*- coding:utf-8 -*-

import requests

url = "http://127.0.0.1:9999/"
# https://requests.readthedocs.io/en/latest/user/quickstart/

# POST发送带 json格式数据 的请求
j = {'name':'jerry'}
p = requests.post(url=url, json=j)
print(p.text)
```
&emsp;&emsp;先执行服务端程序，再执行客户端程序，客户端得到的返回数据如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1046aa716fad0d8e209148976fd8cf9a.png)
# 3、基于POST的带字符串格式参数请求
## 3.1 API（服务端）
&emsp;&emsp;创建API的代码如下：

```python
from flask import Flask, request

app = Flask(__name__)
@app.route("/", methods=['POST'])
def hello():

    the_text = request.data
    the_text = the_text.decode('utf-8')
    """
    这里可以添加函数内部的计算逻辑
    """
    return '你好,' + the_text  # 返回response

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=9999)           # 启动服务器
```
## 3.2 客户端
&emsp;&emsp;客户端的代码如下：

```python
# -*- coding:utf-8 -*-

import requests

url = "http://127.0.0.1:9999/"
# https://requests.readthedocs.io/en/latest/user/quickstart/

# POST发送带 字符串 的请求
d = "小明"
p = requests.post(url=url, data=d.encode("utf-8"))
print(p.text)
```
&emsp;&emsp;先执行服务端程序，再执行客户端程序，客户端得到的返回数据如下：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d484c4f9f10a2a0f2129628b76c7e841.png)

# 4、总结
&emsp;&emsp;以上就实现了几种不同的API创建方式。
