# Chapter10 Crawling thru forms and logins
## submit a form
最简单的例子就是输入用户名和密码，再提交。
> http://pythonscraping.com/pages/files/form.html
```
 <form method="post" action="processing.php">
     First name: <input type="text" name="firstname"><br>
     Last name: <input type="text" name="lastname"><br>
     <input type="submit" value="Submit">
     </form>
```
- 查看源代码，确认两个输入字段的名称，分别是“firstname”和“lastname"。这是之后被post到服务器上的可变参数的名称。
- 表单的操作发生在“processing.php”,post请求发生在这个页面上。
```
params = {'firstname': 'Ryan', 'lastname': 'Mitchell'}
r = requests.post("http://pythonscraping.com/pages/processing.php", data=params)
print(r.text)
```
## 单选按钮、复选框和其他输入
还有滚动条、邮箱、日期等。
- 字段名称：通过源代码寻找name属性。
- 字段值：需要确定数据格式。

## 提交文件和图像
没看懂在干啥。

## 处理登陆和cookie
这里我向欢迎页面发送了一个登录参数，它的作用就像登录表单的处理器。然后我从请求 结果中获取 cookie，打印登录状态的验证结果，然后再通过 cookies 参数把 cookie 发送到 简介页面。
```
import requests

params = {'username': 'Ryan', 'password': 'password'}
r = requests.post('http://pythonscraping.com/pages/cookies/welcome.php', params)
print('Cookie is set to:')
print(r.cookies.get_dict())
print('Going to profile page...')
r = requests.get('http://pythonscraping.com/pages/cookies/profile.php', 
                 cookies=r.cookies)
print(r.text)
```
或者不使用cookie而是用Requests库里的session函数。
```
import requests

session = requests.Session()

params = {'username': 'username', 'password': 'password'}
s = session.post('http://pythonscraping.com/pages/cookies/welcome.php', params)
print("Cookie is set to:")
print(s.cookies.get_dict())
print('Going to profile page...')
s = session.get('http://pythonscraping.com/pages/cookies/profile.php')
print(s.text)
```
### HTTP接入认证
使用requests库里的auth模块。
```
import requests
from requests.auth import AuthBase
from requests.auth import HTTPBasicAuth

auth = HTTPBasicAuth('ryan', 'password')
r = requests.post(
    url='http://pythonscraping.com/pages/auth/login.php', auth=auth)
print(r.text)
```