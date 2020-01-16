#### splinter快速入门


##### splinter使用前的准备

1. 安装splinter
```shell
[sudo] pip install splinter
```

2. 这里我自己使用使用的是chrome, 所以我要安装ChromeDriver(需要翻墙), 不安装默认使用firefox
```text
https://chromedriver.chromium.org/downloads
```
下完完后解压zip包, 添加到环境变量
这里需要注意的是, 不需要指定到chromedriver, 指定到chromedriver所在的文件夹下即可。否则会抛出异常。
```shell
vi ~/.zshrc
export PATH=$PATH:$ChromeDriverDir
```

##### 一个实例

该例子就是官网上的一个例子, 只不过稍微有些修改。

我们要做到的是
  1. 搜索"python官网"
  2. 查找python官网是否存在


###### 创建Browser实例
1. 首先导入Browser类, 并创建一个实例
```python
from splinter import Browser
browser = Browser()
# 指定driver为chrome浏览器
# browser = Browser('chrome')
```
如果没有给Browser指定driver, 默认使用firefox


###### 访问百度搜索页面
使用 browser.visit 方法可访问任意网站. 让我们访问一下百度搜索页面:

```shell
browser.visit('http://baidu.com')
```


###### 输入搜索关键词
页面加载完毕后, 可以进行一系列交互动作, 如点击, 输入框填充内容, 选择单选按钮和复选框等等。
现在让我们在百度搜索框中填充"Python"

```python
# wd可以看到baidu在搜索的时候有个wd, 可以自己去输入框输入看看(https://www.baidu.com/s?wd=Python)
browser.fill('wd', 'Python') # 这样我们当然不会有问题, 但是如果我们写入python官网.带入中文就会有问题了,

# UnicodeDecodeError: 'utf8' codec can't decode byte 0xe4 in position 0: unexpected end of data
# 我们需要使用decode("utf-8")就可以解决了。
# browser.fill('wd', 'python官网.')
```


###### 点击搜索按钮

告诉 Splinter 哪一个按钮需要点击。这个按钮 - 或任意其他元素 - 可以通过它的css, xpath, id, tag 或 name来识别。


```shell
# 通过id进行查找, 我比较喜欢使用, 唯一.
button = browser.find_by_id('su')

# 当然也可以使用xpath查找
button = browser.find_by_xpath('//input[@type="submit"]')

# 获取到button我们就可以点击了
button.click()
```

###### 查看 Python官方网站是否在搜索结果中
点击搜索按钮后，可以通过以下步骤检测Python官方网站是否在搜索结果中。


```shell
if browser.is_text_present('https://www.python.org/'):
    print 'Yes found it. :)'
else:
    print 'No. didn\'t find it :( '
```

以上结束之后, 通过browser.quit()退出浏览器



##### 完整代码

```python

#!/usr/bin/env python
# -*- coding: utf-8 -*-

from splinter import Browser

def getBrowser():
    # browser = Browser()
    # 指定driver为chrome, 不指定则为firefox
    browser = Browser('chrome')
    return browser

def accessURL(url):
    browser = getBrowser()
    browser.visit(url)

    # 搜索测试
    search_text = 'python官网.'
    browser.fill('wd', search_text.decode("utf-8"))
    button = browser.find_by_id('su')
    button.click()

    if browser.is_text_present('https://www.python.org/'):
        print 'Yes found it. :)'
    else:
        print 'No. didn\'t find it :( '

    browser.quit()

if __name__ == '__main__':
    accessURL('http://baidu.com')
```
