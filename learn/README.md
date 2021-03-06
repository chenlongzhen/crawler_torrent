# 爬虫学习
https://www.bilibili.com/video/BV1Yh411o7Sz?p=2
https://book.apeland.cn/

## 1. 分类
1. 通用爬虫： 一整张页面数据
2. 聚焦爬虫：特定局部内容
3. 增量爬虫：检测数据更新，只抓取更新出来的数据

## 2. robots.txt 协议
 规定那些可以以及不能爬取。 https://www.taobao.com/robots.txt

## 3.1 http
- 概念：服务器和客户端进行数据交互的一种形式。

### 常用请求头信息
- User-Agent: 请求载体的身份信息
    - chrome network 中能够获取
- Connection: 请求完毕后，是断开还是保持连接

### 常用响应头信息
- Content-Type: 服务器端响应回服务器端的数据类型

## 3.2 https
- 概念: 安全的超文本传输协议

### 加密方式
- 对称加密
- 非对称加密
- 证书加密

## 4 requests模块
网络请求模块, 模拟浏览器发请求

流程:
- 指定url
- 发起请求 get post
- 获取相应数据
- 持久化存储相应数据

### 4.1 UA伪装
让爬虫对应的请求伪装成浏览器, 在chrome里network扎到user-agent即可
```python
# UA伪装
headers = {'User-Agent' :'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36'}
response = requests.get(url=url, params=param, headers = headers)
```

## 4.2 post请求
```
https://fanyi.baidu.com/#zh/en/%E7%8B%97
network-xhr-选择相应post结果-response里是结果
Request URL: https://fanyi.baidu.com/sug
Request Method: POST
content-type: application/json
Form Data:  kw:gou (输入的是狗)
```
关于post的返回格式: 
1. text
2. content 二进制 例如图片
3. json

post请求携带了参数,携带了json数据,如何发post请求?
```python
w = input('enter a word')
data = {
    'kw':kw
}

# 4. 发送请求
response = requests.post(url = post_url, data = data, headers=headers)

# 5.获取数据 (如果确认返回时json的才行, content-type中可以看到!)
page_text =response.json()
```

## 5. 数据解析
1. 正则
2. bs4
3. xpath 重点!
https://book.apeland.cn/details/78/

### 5.1 正则匹配
https://book.apeland.cn/details/79/

### 5.2 bs4
- 数据解析原理:
    1. 标签定位
    2. 提取标签\标签属性中存储的数据值
- bs4数据解析原理:
    1. 实例化一个beautifulSoup对象, 并将源码数据加载
    2. 调用对象中的方法定位
- install
    1. pip install bs4
    2. pip install lxml

    ```python
    from bs4 import BeautifulSoup
    ```
- 实例化方法:  
1. 加载html
```python
fp = open('./text.html', 'r', encoding='utf-8')
soup = BeautifulSoup(fp, 'lxml')
```
2. 将web上获取的页面源码加载到对象中  
```python
page_text = response.text
soup = BeatifulSoup(page_text, 'lxml')
```

- 用于解析的方法和属性
```
soup.tagName : 第一次出现的name对应的标签
soup.find('div'): 第一次出现的div标签
soup.find('div',class_  = 'song'): class='song' 对应的div .[/id/attr]
soup.find_all('a') : 返回列表

soup.select('.tang'): ( 匹配 div class='tang', 但会列表.
soup.select('.tang > ul > li > a')[0] : 层级选择器, 取第0个标签
soup.select('.tang > ul a')[0]: 空格表示多个层级! > 表示一个层级. 
```
- 获取标签中的文本
```
soup.select('.tang > ul a').text/string/get_text() 
text: 所有内容
string: 只获取直系文本 
get_text:所有内容 

soup.select('.tang > ul a').['href'] # 获取href内容
```

### 5.3 xpath
首选xpath解析

1. 实例化etree
```
from lxml import etree
tree = etree.parse(filePath)
tree = etree.HTML('page_text')
```
2. xpath('xpath表达式')
```
tree.xpath('/html/body/div')
tree.xpath('/html//div') #//多个层级
tree.xpath('//div') #//多个层级 所有div

tree.xpath('//div[@class="song"]') # 属性定位, class=song
tree.xpath('//div[@class="song"]/p[3]') #索引定位 从1开始的
tree.xpath('//div[@class="tang']//li[5]/a/text())[0] 

tree.xpath('//div[@class="song"]/img/@src') #取属性
tree.xpath('//div[@class="song"]/img/@src | '//div[@class="song"]/img/@src') #同时取多个 或

url_video = tree.xpath('//div[contains(@class, "prism-player")]/video/@src')[0] # contains 尤其是带空格的用这个
```
3. 验证码
百度ocr : https://ai.baidu.com/ai-doc/OCR/1k3h7y3db
```
# encoding:utf-8

import requests
import base64

'''
通用文字识别（高精度版）
'''

request_url = "https://aip.baidubce.com/rest/2.0/ocr/v1/accurate_basic"
# 二进制方式打开图片文件
f = open('[本地文件]', 'rb')
img = base64.b64encode(f.read())

params = {"image":img}
access_token = '[调用鉴权接口获取的token]'
request_url = request_url + "?access_token=" + access_token
headers = {'content-type': 'application/x-www-form-urlencoded'}
response = requests.post(request_url, data=params, headers=headers)
if response:
    print (response.json())
```
## 6. 模拟登陆
见ocr_login.py代码

## 7. cookie模拟登陆
见ocr_login.py代码

http/https协议特性: 无状态, 不会记录用户状态.
需要用到cookie: 用来让服务器端记录客户端状态.
- 方法
1. 手动放到heaher上
2. 使用session模拟登陆post请求发送(cookie保存在这里)
   ```
    session  = request.Session()
   ```
## 8. 代理
代理评测: https://www.zhihu.com/question/55807309
http://www.goubanjia.com/

代理ip类型:
1. http
2. https

## 9. 异步爬虫
高性能异步爬虫
目的：在爬虫中使用异步实现高性能的数据爬取操作。

异步爬虫的方式：
    - 1.多线程，多进程（不建议）：
        好处：可以为相关阻塞的操作单独开启线程或者进程，阻塞操作就可以异步执行。
        弊端：无法无限制的开启多线程或者多进程。
    - 2.线程池、进程池（适当的使用）：
        好处：我们可以降低系统对进程或者线程创建和销毁的一个频率，从而很好的降低系统的开销。
        弊端：池中线程或进程的数量是有上限。

- 3.单线程+异步协程（推荐）：
    event_loop：事件循环，相当于一个无限循环，我们可以把一些函数注册到这个事件循环上，
    当满足某些条件的时候，函数就会被循环执行。

    coroutine：协程对象，我们可以将协程对象注册到事件循环中，它会被事件循环调用。
    我们可以使用 async 关键字来定义一个方法，这个方法在调用时不会立即被执行，而是返回
    一个协程对象。

    task：任务，它是对协程对象的进一步封装，包含了任务的各个状态。

    future：代表将来执行或还没有执行的任务，实际上和 task 没有本质区别。

    async 定义一个协程.

    await 用来挂起阻塞方法的执行。
### 例子
ascync_learn_v3.py   

## 10 selenium使用
动态加载数据处理:
1. 抓包工具
2. selenium
用途:
1. 动态加载获取
2. 模拟登陆


selenium使用流程：
   1. 环境安装：pip install selenium
    
   2. 下载一个浏览器的驱动程序（谷歌浏览器）
        - 下载路径：http://chromedriver.storage.googleapis.com/index.html
        - 驱动程序和浏览器的映射关系：http://blog.csdn.net/huilan_same/article/details/51896672
        https://chromedriver.chromium.org/
   3. 实例化一个浏览器对象
   4. 编写基于浏览器自动化的操作代码
        - 发起请求：get(url)
        - 标签定位：find系列的方法
        - 标签交互：send_keys('xxx')
        - 执行js程序：excute_script('jsCode')
        - 前进，后退：back(),forward()
        - 关闭浏览器：quit()
   5. selenium处理iframe
        - 如果定位的标签存在于iframe标签之中，则必须使用switch_to.frame(id)
        - 动作链（拖动）：from selenium.webdriver import ActionChains
            - 实例化一个动作链对象：action = ActionChains(bro)
            - click_and_hold（div）：长按且点击操作
            - move_by_offset(x,y)
            - perform()让动作链立即执行
            - action.release()释放动作链对象




