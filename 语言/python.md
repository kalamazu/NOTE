
#eval
>eval(expression, globals=None, locals=None)
- **`expression`**: 一个字符串，表示要执行的 Python 表达式。
    
- **`globals`**: 可选参数，表示全局命名空间（字典形式）。如果未提供，则使用当前全局命名空间。
    
- **`locals`**: 可选参数，表示局部命名空间（字典形式）。如果未提供，则使用当前局部命名空间。

用于将字符串作为 Python 表达式进行求值并返回结果

#python字符串操作
```python
# 清理接收到的表达式，移除空白字符并替换除法符号
        aa = re.sub(r'[ \t\n\r=]+', '', a.decode()).replace('/', '//')
```

#strip
string.strip([chars])
- `string`：要处理的字符串。
- `chars`（可选）：指定要删除的字符。如果未指定，默认删除空白字符。


#关键字参数
需遵循参数顺序规则：​**必选参数 → 默认参数 → `*args` → `**kwargs`**
**注意**：`*args`和`**kwargs`之间不能有其他参数



#ord 
返回字符的ascii码值，返回int

#chr
ord的逆过程

#str
输入什么就得到什么字符串


#字符串格式化
“这是一个占位符%s” % Value


#yield
定义生成器函数，暂停函数执行，并返回一个值，同时保留函数状态，以便下次调用时能从暂停的地方继续执行