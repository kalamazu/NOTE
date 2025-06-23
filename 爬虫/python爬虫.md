### 流程
- 先由 urllib 模块的 request 方法打开 URL 得到网页 HTML 对象。
- 使用浏览器打开网页源代码分析网页结构以及元素节点。
- 通过 Beautiful Soup 或则正则表达式提取数据。
- 存储数据到本地磁盘或数据库。

[python教程](https://c.biancheng.net/python/)




User-Agent 字符串头
[UA检测](https://useragent.buyaocha.com/#google_vignette)

[http检测](http://httpbin.org/)




#URL
采用ASCII码
- 冒号：用于分隔协议和主机组件，斜杠用于分隔主机和路径
- `?`：用于分隔路径和查询参数等。
- `=`用于表示查询参数中的键值对。
- `&`符号用于分隔查询多个键值对。
不在URL字符范围内的，需要进行URL编码



#爬虫模板
```python
# 程序结构
class xxxSpider(object):
    def __init__(self):
        # 定义常用变量,比如url或计数变量等
       
    def get_html(self):
        # 获取响应内容函数,使用随机User-Agent
   
    def parse_html(self):
        # 使用正则表达式来解析页面，提取数据
   
    def write_html(self):
        # 将提取的数据按要求保存，csv、MySQL数据库等
       
    def run(self):
        # 主函数，用来控制整体逻辑
       
if __name__ == '__main__':
    # 程序开始运行时间
    spider = xxxSpider()
    spider.run()
```


#正则表达式

| 元字符     | 内容                  |
| ------- | ------------------- |
| .       | 匹配除换行符以外的任意字符\|     |
| \w      | 匹配所有普通字符(数字、字母或下划线) |
| \s      | 匹配任意的空白符            |
| \d      | 匹配数字                |
| \n      | 匹配一个换行符             |
| ^       | 匹配字符串的开始位置          |
| $       | 匹配字符串的结尾位置          |
| a\|b    | 匹配字符 a 或字符 b        |
| [...]   | 匹配字符组中的字符           |
| [^ ...] | 匹配除了字符组中字符的所有字符     |
|         |                     |


| 量词    | 用法     |
| ----- | ------ |
| *     | 重复未知次数 |
| +     | 重复1到多次 |
| ？     | 0或1    |
| {n}   | n次     |
| {n，m} | n到m次   |
