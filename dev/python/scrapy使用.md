## 创建scrapy项目
进入当前环境，创建项目
```
scrapy startproject ArticleSpider
```

### scrapy项目结构
#### settings.py
##### Obey robots.txt rules
是否遵循 robots.txt 规则，我们一般设置为False

## 创建网站模板
进入项目，创建具体网站的模板
```
cd ArticleSpider 
scrapy genspider jobbole blog.jobbole.com
```
创建伯乐网站的模板。默认是 `basic`模板。

通过 `scrapy genspider -l`查看scrapy模板列表

```
Available templates:
  basic
  crawl
  csvfeed
  xmlfeed
```

指定模板创建爬虫: `scrapy genspider -t crawl test www.test.com`


## 运行程序
进入项目的根目录，	通过命令启动spider
```
scrapy crawl jobbole
```

使用debug模式调试程序，需要通过`scrapy`的`cmdlime`模块的`execute`函数。在项目根目录下创建`main.py`
```python
from scrapy.cmdline import execute

import os
import sys

# 当前文件目录
current_file_name = os.path.abspath(__file__)
# 项目根目录
project_path = os.path.dirname(current_file_name)
sys.path.append(project_path)
# 启动爬虫
execute(["scrapy", "crawl", "jobbole"])
```
通过debug `main.py`来调试程序。

### 使用 shell调试url
使用shell进入Spider终端
```
scrapy shell url
```
之后我们可以通过终端匹配xpath
```shell
>>> title = response.xpath('//*[@id="post-30956"]/div[1]/h1/text()')
>>> title
[<Selector xpath='//*[@id="post-30956"]/div[1]/h1/text()' data='常被问到的
十个 Java 面试题'>]
>>> title.extract()
['常被问到的十个 Java 面试题']
```

### Spider的暂停与重启

```shell
scrapy crawl lagou -s JOBDIR=job_info/001
```

在爬取拉钩网时，添加参数 `-s JOBDIR=job_info/001`,scrapy将会把当前爬虫的状态信息保存到 `%project_path%/job_info/001/`目录下。当我们想要中断爬虫时，在交互界面使用 `Ctrl C` 中断，或者使用 `kill -s pid`命令发起中断请求。之后使用同样的命令重启爬虫。

注意：每个爬虫都应该有一个独立的目录保存中断的状态信息。

---

## Scrapy 架构

![scrapy架构](C:\Users\59261\AppData\Roaming\Typora\typora-user-images\1547384589201.png)

## 反爬虫机制

- 随机更换 User-Agent
- 设置IP代理
- 

### 随机更换 User-Agent

scrapy 中通过 `UserAgentMiddleware`来设置 User-Agent

```python
class UserAgentMiddleware(object):
    """This middleware allows spiders to override the user_agent"""

    def __init__(self, user_agent='Scrapy'):
        self.user_agent = user_agent

    @classmethod
    def from_crawler(cls, crawler):
        o = cls(crawler.settings['USER_AGENT'])
        crawler.signals.connect(o.spider_opened, signal=signals.spider_opened)
        return o

    def spider_opened(self, spider):
        self.user_agent = getattr(spider, 'user_agent', self.user_agent)

    def process_request(self, request, spider):
        if self.user_agent:
            request.headers.setdefault(b'User-Agent', self.user_agent)

```

scrapy 通过在 `settings.py`中配置`USER_AGENT`属性设置User-Agent

随机更换 User-Agent就是仿照 `UserAgentMiddleware`这个中间件，重写 `process_request`方法。在这之前，我们先禁用 `UserAgentMiddleware`这个中间件。

> If you want to disable a builtin middleware (the ones defined in [`SPIDER_MIDDLEWARES_BASE`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES_BASE), and enabled by default) you must define it in your project [`SPIDER_MIDDLEWARES`](https://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES) setting and assign None as its value. For example, if you want to disable the off-site middleware:
>
> ```
> SPIDER_MIDDLEWARES = {
>     'myproject.middlewares.CustomSpiderMiddleware': 543,
>     'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': None,
> }
> ```

在 `middlewares.py`文件中定义 `RandomUserAgentMiddlewares`类，实现在请求中随机添加User-Agent。

```python
class RandomUserAgentMiddleware(object):
    """
    随机更换User-Agent
    """

    def __init__(self, crawler):
        super(RandomUserAgentMiddleware, self).__init__()
        self.ua = UserAgent()
        self.ua_type = crawler.settings.get('USER_AGENT_TYPE', 'random')

    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler)

    def process_request(self, request, spider):
        def get_ua():
            return getattr(self.ua, self.ua_type, 'random')

        request.headers.setdefault('User-Agent', get_ua())
```

在这里没有自己维护User-Agent列表，而是通过 `fake_useragent`这个第三方模块构造。导入 `from fake_useragent import UserAgent`，通过 `UserAgent`类生成不同的User-Agent。

最后在 `settings.py`的 `DOWNLOADER_MIDDLEWARES`中添加 `RandomUserAgentMiddleware`

```python
DOWNLOADER_MIDDLEWARES = {
    'ArticleSpider.middlewares.RandomUserAgentMiddleware': 543,
    # 禁用内置UserAgentMiddleware
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None
}
```

---

### 设置IP代理

自定义 `ProxyIPMiddleware`中间件

```
    def process_request(self, request, spider):
        proxy = self.get_proxy()
        # proxy 已经在validUsefulProxy中验证，不需要再进行验证
        if proxy:
            request.meta['proxy'] = 'http://{0}'.format(proxy)
```

`request.meta['proxy'] = 'http://{0}'.format(proxy)`在请求中设置proxy

---

### 禁用Cookie

在一些不需要登录就可以访问的网站，我们禁用Cookie就可以提高爬取率

在 `settings.py`中设置 `COOKIES_ENABLED = False`

如果一些Spider需要设置Cookie，可以在Spider中通过 `custom_settings`设置

```
custom_settings = {
	'COOKIES_ENABLED': True
}
```

---

## Selenium自动测试框架

使用selenium可以模拟浏览器的操作

python中先导入selenium模块

```shell
pip install selenium
```

### 模拟登录操作

```python
from selenium import webdriver
browser = webdriver.Chrome(executable_path=SELENIUM_DRIVER_PATH)

browser.find_element_by_css_selector('.SignContainer-switch span').click()
time.sleep(1)
browser.find_element_by_css_selector('.Login-content input[name="username"]').send_keys('账号')
time.sleep(2)
browser.find_element_by_css_selector('.Login-content input[name="password"]').send_keys('密码')
time.sleep(1.5)
browser.find_element_by_css_selector('.Login-content button[type="submit"]').click()

browser.close()
```

`SELENIUM_DRIVER_PATH` 是浏览器驱动的目录，我这里使用chrome浏览器的驱动

---

### 无界面的Selenium

Xvfb是流行的虚拟实现库，可以使很多需要图形界面的程序虚拟运行。

pyvirtualdisplay是Xvfb的python封装。

```python
# linux下无界面的selenium chrome
from pyvirtualdisplay import Display
# 设置窗口不可见
display = Display(visible=0, size=(800, 600))
display.start()

browser = webdriver.Chrome(executable_path=SELENIUM_DRIVER_PATH, options=chrome_opt)
browser.get('https://www.taobao.com/')
print(browser.page_source)
browser.close()
```

---

### scrapy中集成Selenium

在scrapy中通过中间件方式使用selenium

```python
class SeleniumPageMiddleware(object):
    """
    使用selenium访问动态网页
    """

    def process_request(self, request, spider):
        if spider.browser:
            spider.browser.get(request.url)
            time.sleep(3)
            logger.debug('selenium 访问: {}'.format(request.url))
            return HtmlResponse(url=spider.browser.current_url,
                                body=spider.browser.page_source,
                                encoding='utf-8')
```

在 `spider`中初始化 `chrome`，并在spider停止时关闭 `chrome`

```python
    def __init__(self):
        self.browser = webdriver.Chrome(executable_path=SELENIUM_DRIVER_PATH)
        super(JobboleSpider, self).__init__()
        # 在spider关闭时关闭selenium
        dispatcher.connect(self.spider_closed, signals.spider_closed)

    def spider_closed(self, spider):
        self.browser.quit()
```

---

### 设置不加载图片

```python
chrome_opt = webdriver.ChromeOptions()
prefs = {'profile.managed_default_content_settings.images': 2}
chrome_opt.add_experimental_option('prefs', prefs)
browser = webdriver.Chrome(executable_path=SELENIUM_DRIVER_PATH, options=chrome_opt)
```

不加载图片可以提高网页的访问速度

---

## Scrapy 去重策略

scrapy默认的去重策略在 `scrapy/dupefilters.py`文件中

```python
    def request_seen(self, request):
        fp = self.request_fingerprint(request)
        if fp in self.fingerprints:
            return True
        self.fingerprints.add(fp)
        if self.file:
            self.file.write(fp + os.linesep)
```

通过 `hashlib.sha1()`获取数据摘要

如果要自定义去重策略，需要重写 `request_fingerprint`方法

---

## Scrapy telnet

启动spider时，scrapy将启动telnet监听

```shell
2019-01-17 19:27:33 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
```

我们可以通过 `telnet localhost 6023` 获取爬虫状态

---

## 错误状态码处理

scrapy默认处理200-300之间的HTTP状态码，`HttpErrorMiddleware `中间件会过滤其他错误状态码。如果我们想要对其他状态码进行处理，设置 `handle_httpstatus_list`属性或者设置 ` HTTPERROR_ALLOWED_CODES`列表添加要处理的状态码。[官方参考文档](https://doc.scrapy.org/en/latest/topics/spider-middleware.html#module-scrapy.spidermiddlewares.httperror)

## 分布式 scrapy

### 分布式需要解决问题

- request队列集中管理
- 去重集中管理

