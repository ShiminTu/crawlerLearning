# Chapter1 初见网络爬虫
## 网络连接
**浏览器获取信息的过程**  
1. Bob客户端发送请求头和消息体。请求头包含bob本地路由器的MAC地址和Alice服务器的ip地址。消息体包含bob的请求。
2. Bob本地路由器收到packet，再加上bob的ip地址作为发件地址，由bob的mac地址寄往Alice的ip地址。
3. Alice的服务器在Alice的ip地址收到packet。
4. 读取packet请求头中的目标端口，传递到对应的网络服务器应用。
5. 网路服务器应用从服务器处理器收到一串数据（这是一个get请求；请求index.html）。
6. 网路服务器应用找到index.html，打包成一个packet发回给Bob。

**urllib**  
urllib是标准库，包含从网页 请求数据，处理 cookie，甚至改变像请求头和用户代理这些元数据的函数。
```
from urllib.request import urlopen
html = urlopen('http://pythonscraping.com/pages/page1.html')
print(html.read())
```

## Beautiful Soup
### 简介
BS通过html标签来组织和格式化网页信息，展现XML结构信息。
```
from urllib.request import urlopen
from bs4 import BeautifulSoup
html = urlopen('http://www.pythonscraping.com/pages/page1.html')
#第一个参数是该对象基于的html文本，第二个参数是解析器（‘html.parser','lxml','html5lib)。
bs = BeautifulSoup(html.read(), 'html.parser')
print(bs.h1)
```
### 网络连接和异常处理
1. 网页在服务器上不存在  
**返回http错误**，可能是"404 Page Not Found"或"500 Internal Server Error"，此时urlopen会给出HTTPError。
```
from urllib.request import urlopen
from urllib.error import HTTPError
try:
	html = urlopen('http://pythonscraping.com/pages/page1.html')
except HTTPError as e:
	print(e)
	# 返回空值，中断程序，或者执行另一个方案
else:
	# 程序继续。注意:如果你已经在上面异常捕捉那一段代码里返回或中断(break)， 
	# 那么就不需要使用else语句了，这段代码也不会执行
```
2. 服务器不存在  
**urlopen给出URLError**,意味着获取不到服务器，也得不到服务器返回的httperror代码。
```
from urllib.request import urlopen
from urllib.error import HTTPError
from urllib.error import URLError
try:
	html = urlopen('http://pythonscraping.com/pages/page1.html')
except HTTPError as e:
	print(e)
	# 返回空值，中断程序，或者执行另一个方案
except URLError as e:
	print('The server can not be found!')
else:
	# 程序继续。注意:如果你已经在上面异常捕捉那一段代码里返回或中断(break)， 
	# 那么就不需要使用else语句了，这段代码也不会执行
	print('It worked!')
```
3. tag不存在  
检查标签是否存在，不存在则返回None，否则调用其子标签会返回AttributeError错误。
```
bs.nonExistingTag ## return None
bs.nonExistingTag.someTag ## return AttributeError
```

```
try:
    badContent = bs.nonExistingTag.someTag
except AttributeError as e:
    print('Tag is not found!')
else:
    if badContent == None:
        print('Tag is not found!')
    else:
        print(badContent)
```

**封装函数**
```
from urllib.request import urlopen
from urllib.error import HTTPError
from bs4 import BeautifulSoup

def getTitle(url):
    try:
        html = urlopen(url)
    except HTTPError as e:
        return None
    
    try:
        bs = BeautifulSoup(html.read(),'html.parser')
        title = bs.body.h1
    except AttributeError:
        return None
    else:
        return title
    
url = 'http://www.pythonscraping.com/pages/page1.html'

getTitle(url)
```

# Chapter2 复杂HTML解析
## 找tag
### find() & findAll()函数
```
find(
	tag,
	attributes,
	recursive,
	text,
	keywords
)

find_all(
	tag,
	attributes, ## 字典形式
	recursive, ## 是否递归
	text, ## 标签文本值匹配
	limit, ## 返回的标签数量
	keywords ##选择具有制定属性的标签
)
```
```
url = 'https://www.pythonscraping.com/pages/warandpeace.html'

html = urlopen(url)
bs = BeautifulSoup(html, 'html.parser')

nameList = bs.find_all('span', {'class':{'green','red'}}) ##bs4.element.ResultSet
for name in nameList:
    print(name.get_text()) ## get_text() return string.
```
### BeautifulSoup对象
- BeautifulSoup对象：bs
- 标签tag对象：bs.div.h1
- Navigable String: 标签里的文字
- Comment: html文档的注释标签。

### 通过标签位置找标签，导航树
- 子标签和后代标签
子标签是父标签的下一级，后代标签是父标签下面所有级别的标签。find默认找后代标签，如果只想找子标签，可以
```
bs.find('table',{'id':'giftlist'}).children
```
- 兄弟标签
```
# next_siblings调用之后所有的兄弟标签
# next_sibling
# previous_siblings抵用前面所有的兄弟标签
# previous_sibling
bs.find('img',{'src':'../img/gifts/img1.jpg'}).parent.previous_sibling.get_text()
```
## 正则表达式
**常用正则表达式符号**  
![avatar](/Users/tushimin/Desktop/截屏2022-12-05 18.29.53.png)

```
imgs = bs.find_all('img',{'src':re.compile('\.\.\/img\/gifts\/img.*\.jpg')})
for img in imgs:
    print(img['src'])
```
## 获取tag属性
```
tag.attrs ##返回字典
```
## lambda函数
```
bs.find_all(lambda tag: len(tag.attrs) == 2)
```

# Chapter3 编写网络爬虫
## 遍历单个域名
```
from urllib import urlopen
from bs4 import BeautifulSoup
import re
from selenium import webdriver

## open url and bs
site = 'http://en.wikipedia.org/wiki/Kevin_Bacon'
driver = webdriver.Chrome(executable_path = '/Users/tushimin/Desktop/chromedriver')
driver.get(site)
bs = BeautifulSoup(driver.page_source, 'html.parser')
driver.close()

## 
links = bs.find('div',{'id':'bodyContent'}).find_all('a',href = re.compile('^(/wiki/)((?!:).*)$'))

for link in links:
	print(link.attrs['href'])
```

```
import datetime
import random 
from urllib.request import urlopen
import re
from selenium import webdriver

random.seed(datetime.datetime.now())

def getLinks(articleUrl):
    site = f'http://en.wikipedia.org{articleUrl}'
    driver = webdriver.Chrome(executable_path = '/Users/tushimin/Desktop/chromedriver')
    driver.get(site)
    bs = BeautifulSoup(driver.page_source, 'html.parser')
    driver.close()
    
    links = bs('div',{'id':'bodyContent'}).find_all('a',href = re.compile('^(/wiki/)((?!:).)*$'))
    
    return links

links = getLinks('/wiki/Kevin_Bacon')

while len(links) > 0:
    newActicleUrl = links[random.randint(2,len(links)-1)].attrs['href']
    print(newActicleUrl)
    links = getLinks(newActicleUrl)
```

## 抓取整个网站
```
from urllib.request import urlopen
from bs4 import BeautifulSoup
import re
from selenium import webdriver

pages = set()

def getLinks(pageUrl):
    global pages
    
    site = f'http://en.wikipedia.org{pageUrl}'
    driver = webdriver.Chrome(executable_path = '/Users/tushimin/Desktop/chromedriver')
    driver.get(site)
    bs = BeautifulSoup(driver.page_source,'html_parser')
    driver.close()
    
    links = bs.find('div',{'id':'bodyContent'}).find_all('a',href = re.compile('^(/wiki/)((?!:).)*$'))
    
    for link in links:
        if link.attrs['href'] not in pages:
            newPage = link.attrs['href']
            pages.add(newPage)
            getLinks(newPage)
```

```
from urllib.request import urlopen
from bs4 import BeautifulSoup
import re
from selenium import webdriver

pages = set()

def getLinks(pageUrl):
    global pages
    
    site = f'http://en.wikipedia.org{pageUrl}'
    driver = webdriver.Chrome(executable_path = '/Users/tushimin/Desktop/chromedriver')
    driver.get(site)
    bs = BeautifulSoup(driver.page_source,'html_parser')
    driver.close()
    
	## 每个打印语句按照数据在页面上出现的可能性从高到低排序
    try:
        print(bs.find('h1').find('span').get_text())
        print(bs.find(id ='mw-content-text').find_all('p')[1])
        print(bs.find(id='ca-edit').find('a').attrs['href'])
    except AttributeError:
        print('lost some tags!')
    
    links = bs.find('div',{'id':'bodyContent'}).find_all('a',href = re.compile('^(/wiki/)((?!:).)*$'))
    
    for link in links:
        if link.attrs['href'] not in pages:
            newPage = link.attrs['href']
            pages.add(newPage)
            getLinks(newPage)

getLinks('')
```

**处理重定向**
- 服务器端重定向，页面在加载之前URL就变了。urllib可自动处理，requests库需要all_redirects=True。
- 客户端重定向，页面在跳转到新页面之前已经加载。

***抓取的页面url不是进入该页面的url。***

## 在互联网上抓取

首先需要明确的问题：
- 我要收集哪些数据？从预定义网站爬够吗？爬虫需要发现那些我可能不知道的网站吗？
- 到达一个网页后，离开顺着出站链接出站还是在该网页上深度爬？
- 有没有不想爬的网页？
- 法律问题

**可以作为小项目回头看** 

# Chapter4 网络爬虫模型



```
class GameMan:
	def __init__(self, name, gender, age, index):
		self.name = name
		self.gender = gender
		self.age = age
		self.index = index

	def grassland(self):
		self.index -= 200

	def practice(self):
		self.index += 100

	def incest(self):
		self.index -= 500

	def detail(self):
	"""注释：当前对象的详细情况"""

	temp = "姓名:%s ; 性别:%s ; 年龄:%s ; 战斗力:%s" % (self.name, self.gender, self.age, self.fight)
	print(temp)

Cang = GameMan('苍井井','女',18,1000)
Dong = GameMan('东尼木木','男',20,1800)
Bo = GameMan('波多多','女',19,2500)
```

## 处理不同的网页布局
一次爬取多个布局不同的网页。
```
import requests
from bs4 import BeautifulSoup

class Website:
    def __init__(self,name,url,titleTag,bodyTag):
        self.name = name
        self.url = url
        self.titleTag = titleTag
        self.bodyTag = bodyTag
        
class Content:
    def __init__(self,url,title,body):
        self.url = url
        self.title = title
        self.body = body
        
    def print(self):
        print('url:{}'.format(self.url))
        print('title:{}'.format(self.title))
        print('body:{}'.format(self.body))
        
class Crawler:
    def getPage(self,url):
        try:
            req = requests.get(url)
        except requests.exceptions.RequestException:
            return None
        return BeautifulSoup(req.text, 'html.parser')
    
    def safeGet(self, pageObj, selector):
        selectedItems = pageObj.select(selector)
        if selectedItems is not None and len(selectedItems) > 0:
            return '\n'.join([elem.get_text() for elem in selectedItems])
        return ''
    
    def parse(self,site,url):
        bs = self.getPage(url)
        if bs is not None:
            title = self.safeGet(bs,site.titleTag)
            body = self.safeGet(bs,site.bodyTag)
        if title != '' and body != '':
            content = Content(url,title,body)
            content.print()

crawler = Crawler()
siteData = [
         ['O\'Reilly Media', 'http://oreilly.com',
         'h1', 'section#product-description'],
         ['Reuters', 'http://reuters.com', 'h1',
         'div.StandardArticleBody_body_1gnLA'],
         ['Brookings', 'http://www.brookings.edu',
         'h1', 'div.post-body'],
         ['New York Times', 'http://nytimes.com',
         'h1', 'p.story-content']
     ]
websites = []
for row in siteData:
    websites.append(Website(row[0],row[1],row[2],row[3]))
crawler.parse(websites[0], 'http://shop.oreilly.com/product/0636920028154.do')
crawler.parse(websites[1], 'http://www.reuters.com/article/'\
 'us-usa-epa-pruitt-idUSKBN19W2D0')
crawler.parse(websites[2], 'https://www.brookings.edu/blog/'\
 'techtank/2016/03/01/idea-to-retire-old-methods-of-policy-education/')
crawler.parse(websites[3], 'https://www.nytimes.com/2018/01/'\
 '28/business/energy-environment/oil-boom.html')
```

## 结构化爬虫
### 搜索关键词搜集结果
- 关键词常在url中
> http://example.com?search=myTopic  

- 搜索后，常用链接列表形式呈现搜索结果。
> \<span class="result"\>

- 每个结果链接，要么是个相对url，要么是个绝对url。

```
import requests
from bs4 import BeautifulSoup

class Website:
    def __init__(self, name, url, searchUrl, resultListing,
             resultUrl, absoluteUrl, titleTag, bodyTag):
        self.name = name
        self.url = url
        self.searchUrl = searchUrl
        self.resultListing = resultListing
        self.resultUrl = resultUrl
        self.absoluteUrl=absoluteUrl
        self.titleTag = titleTag
        self.bodyTag = bodyTag

class Content:
    def __init__(self,topic,url,title,body):
        self.topic = topic
        self.url = url
        self.title = title
        self.body = body
        
    def print(self):
        print("New article found for topic: {}".format(self.topic))
        print("TITLE: {}".format(self.title))
        print("BODY:\n{}".format(self.body))
        print("URL: {}".format(self.url))
        
class Crawler:
    def getPage(self,url):
        try:
            req = requests.get(url)
        except requests.exceptions.RequestException:
            return None
        return BeautifulSoup(req.text, 'html.parser')
    
    def safeGet(self, pageObj, selector):
        selectedItems = pageObj.select(selector)
        if selectedItems is not None and len(selectedItems) > 0:
            return selectedItems[0].get_text()
        return ''
    
    def search(self,topic,site):
        bs = self.getPage(site.searchUrl + topic)
        searchResults = bs.select(site.resultListing)
        for result in searchResults:
            url = result.select(site.resultUrl)[0].attrs['href']
            if(site.absoluteUrl):
                bs = self.getPage(url)
            else:
                bs = self.getPage(site.url + url)
            if bs is None:
                print("Something was wrong with that page or URL. Skipping!")
                return None
            title = self.safeGet(bs,site.titleTag)
            body = self.safeGet(bs,site.bodyTag)
            if title != '' and body != '':
                content = Content(topic,url,title,body)
                content.print() 
   
crawler = Crawler()
siteData = [
 ['O\'Reilly Media', 'http://oreilly.com',
     'https://ssearch.oreilly.com/?q=','article.product-result',
     'p.title a', True, 'h1', 'section#product-description'],
 ['Reuters', 'http://reuters.com',
     'http://www.reuters.com/search/news?blob=',
     'div.search-result-content','h3.search-result-title a',
     False, 'h1', 'div.StandardArticleBody_body_1gnLA'],
 ['Brookings', 'http://www.brookings.edu',
     'https://www.brookings.edu/search/?s=',
     'div.list-content article', 'h4.title a', True, 'h1',
     'div.post-body']
]
sites = []
for row in siteData:
    sites.append(Website(row[0], row[1], row[2],
                          row[3], row[4], row[5], row[6], row[7]))
    
topics = ['python', 'data science']
for topic in topics:
    print("GETTING INFO ABOUT: " + topic)
for targetSite in sites:
    crawler.search(topic, targetSite)
```

### 通过链接抓取网站
```
import requests
from bs4 import BeautifulSoup
import re

class Website:
    def __init__(self,name,url,targetPattern,absoluteUrl,titleTag,bodyTag):
        self.name = name
        self.url = url
        self.absoluteUrl = absoluteUrl
        self.targetPattern = targetPattern
        self.titleTag = titleTag
        self.bodyTag = bodyTag
        
class Content:
    def __init__(self,url,title,body):
        self.url = url
        self.title = title
        self.body = body
        
    def print(self):
        print('url:{}'.format(self.url))
        print('title:{}'.format(self.title))
        print('body:{}'.format(self.body))

class Crawler:
    '''
    进入主页，根据target pattern找内链，for循环进内链爬内容。
    '''
    def __init__(self, site):
        self.site = site
        self.visited = []
    
    def getPage(self,url):
        try:
            req = requests.get(url)
        except requests.exceptions.RequestException:
            return None
        return BeautifulSoup(req.text, 'html.parser')
    
    def safeGet(self, pageObj, selector):
        selectedElems = pageObj.select(selector)
        if selectedElems is not None and len(selectedElems) > 0:
            return '\n'.join([elem.get_text() for elem in selectedElems])
        return ''
        
    def parse(self, url):
        bs = self.getPage(url)
        if bs is not None:
            title = self.safeGet(bs,self.site.titleTag)
            body = self.safeGet(bs,self.site.bodyTag)
            if title != '' and body != '':
                content = Content(url, title, body)
                content.print()
                
    def crawl(self):
        bs = getPage(self.site.url)
        targetPages = bs.find_all('a', href=re.compile(self.site.targetPattern))
        for targetPage in targetPages:
            targetPage = targetPages.attrs['href']
            if targetPage not in self.visited:
                self.visited.append(targetPage)
                if self.site.absoluteUrl == False:
                    targetPage = '{}{}'.format(self.site.url, targetPage)
                self.parse(targetPage)

reuters = Website('Reuters', 'https://www.reuters.com', '^(/article/)', False,
         'h1', 'div.StandardArticleBody_body_1gnLA')
crawler = Crawler(reuters)
crawler.crawl()
```

### 抓去多种类型的页面
**识别页面类型**
- 通过url
> http://example.com/blog/title- of-post

- 通过网站中存在或缺失的特定字段
> 如果一个页面包含日期，但是不包含作者名字，那你可以将其归类为新闻稿。如果它有标题、主图片、价格，但是没有主要内容，那么它可能是一个产品页面。

- 通过页面中出现的特定标签识别页面
> 可以寻找类似于 <div id="related-products"> 这样的元素来识别产品页面

```
class Website: 
	"""所有文章/网页的共同基类"""
    def __init__(self, name, url, titleTag):
		self.name = name
		self.url = url
		self.titleTag = titleTag

class Product(Website):
	def __init__(self, name, url, titileTag,productNumber,price):
		Website.__init__(name,url,titleTag)
		self.productNumber = productNumber
		self.price = price

class Article(Website):
	def __init__(self, name, url, titileTag,bodyTag,dateTag):
		Website.__init__(name,url,titleTag)
		self.bodyTag = bodyTag
		self.dateTag = dateTag

```
























