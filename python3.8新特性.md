#### 1、海象运算符(赋值表达式)

新增语法`:=`可在表达式内部为变量赋值。它被昵称为“海象运算符”因为它很像是海象的眼睛和长牙。

在这个示例中，赋值表达式可以避免调用 `len()` 两次:

```python
a = [1,2,3,4,5,6,7,8,9,0,'*']
if (n := len(a)) > 10:
    print(f"List is too long ({n} elements, expected <= 10)")

# List is too long (11 elements, expected <= 10)
```

此运算符也适用于配合 while 循环计算一个值来检测循环是否终止，而同一个值又在循环体中再次被使用的情况:

```python
while (block := f.read(256)) != '':
    process(block)
```

列表推导式中，在筛选条件中计算一个值，而同一个值又在表达式中需要被使用:

```python
[clean_name.title() for name in names
 if (clean_name := normalize('NFC', name)) in allowed_names]
```

#### 2、仅限位置形参

新增了一个函数形参语法 `/` 用来指明某些函数形参必须使用仅限位置而非关键字参数的形式。

在下面的例子中，形参 *a* 和 *b* 为仅限位置形参，*c* 或 *d* 可以是位置形参或关键字形参，而 *e* 或 *f* 要求为关键字形参:

```python
def f(a, b, /, c, d, *, e, f):
    print(a, b, c, d, e, f)
```

以下均为合法的调用:

```python
f(10, 20, 30, d=40, e=50, f=60)
```

但是，以下均为不合法的调用:

```python
f(10, b=20, c=30, d=40, e=50, f=60)   # b cannot be a keyword argument:b不能是关键字参数
f(10, 20, 30, 40, 50, f=60)           # e must be a keyword argument:e必须是关键字参数
```

在不需要形参名称时排除关键字参数。 例如，内置的 `len()`函数的签名为 `len(obj, /)`。 这可以排除如下这种笨拙的调用形式:

```python
len(obj='hello')  # 因为obj关键字参数削弱可读性
```

另一个益处是将形参标记为仅限位置形参将允许在**未来修改形参名而不会破坏客户的代码**。 例如，在 `statistics` 模块中，形参名 *dist* 在未来可能被修改。 这使得以下函数描述成为可能:

```python
def quantiles(dist, /, *, n=4, method='exclusive')
    ...
```

#### 3、f-string支持 `=` 用于自动记录表达式和调试文档

增加 `=` 说明符用于 f-string。 形式为 `f'{expr=}'` 的 f-字符串将扩展表示为表达式文本，加一个等于号，再加表达式的求值结果。 例如：

```python
>>> user = 'eric_idle'
>>> member_since = date(1975, 7, 31)
>>> f'{user=} {member_since=}'
"user='eric_idle' member_since=datetime.date(1975, 7, 31)"
```

f-string, 空表达式不被允许，`lambda` 和赋值表达式 `:=` 必须显式地加上圆括号。

f-string中，单个左花括号 `'{'` 标示一个替换字段，以一个 Python 表达式打头，表达式之后可能有一个以叹号 `'!'` 标示的**转换字段**。之后还可能带有一个以冒号 `':'` 标示的**格式说明符**。

 每个表达式会在格式化字符串字面值所包含的位置按照从左至右的顺序被求值。

转换符 `'!s'` 即对结果调用 `str()`，`'!r'` 为调用 `repr()`，而 `'!a'` 为调用 `ascii()`。

>str() 函数将对象转化为适于人阅读的形式。
>
>repr() 函数将对象转化为供解释器读取的形式。
>
>ascii() 函数类似 repr() 函数, 返回一个表示对象的字符串, 但是对于字符串中的非 ASCII 字符则返回通过 repr() 函数使用 \x, \u 或 \U 编码的字符。

格式说明符详情见：<https://docs.python.org/zh-cn/3.8/library/string.html#formatspec>

**双重花括号 `'{{'` 或 `'}}'` 会被替换为相应的单个花括号。**

```python
>>> user = 'eric_idle'
>>> member_since = date(1975, 7, 31)
>>> delta = date.today() - member_since
>>> f'{user=!s}  {delta.days=:,d}'
'user=eric_idle  delta.days=16,075'
```

```python
>>> print(f'{theta=}  {cos(radians(theta))=:.3f}')
theta=30  cos(radians(theta))=0.866
```

#### 4、其他语言特性修改

- 在之前版本中 `continue`语句不允许在 `finally` 子句中使用，这是因为具体实现存在一个问题。 在 Python 3.8 中此限制已被取消。
- 添加 \N{name} 转义符在 正则表达式 中的支持:
```python
>>> notice = 'Copyright © 2019'
>>> copyright_year_pattern = re.compile(r'\N{copyright sign}\s*(\d{4})')
>>> int(copyright_year_pattern.search(notice).group(1))
2019
```
>其中“\N{copyright sign}” 即表示版权符号“©”

- 字典推导式已与字典字面值实现同步，会先计算键再计算值:

对执行顺序的保证对赋值表达式来说很有用，因为在键表达式中赋值的变量将可在值表达式中被使用:

```python
>>> names = ['Martin von Löwis', 'Łukasz Langa', 'Walter Dörwald']
>>> {(n := normalize('NFC', name)).casefold() : n for name in names}
{'martin von löwis': 'Martin von Löwis',
 'łukasz langa': 'Łukasz Langa',
 'walter dörwald': 'Walter Dörwald'}
```

- ### math

添加了新的函数 `math.dist()`用于计算两点之间的欧几里得距离;
添加了新的函数 `math.prod()` 作为的 `sum()` 同类，该函数返回 'start' 值 (默认值: 1) 乘以一个数字可迭代对象的积;
>sum()为求和函数， math.prod()为求积函数

```python
>>> prior = 0.8
>>> likelihoods = [0.6, 3, 0.5]
>>> math.prod(likelihoods, start=prior)
0.72
```

*更多python3.8特性与改动请移步官方文档：<https://docs.python.org/zh-cn/3.8/whatsnew/3.8.html#assignment-expressions>*