# re模块：

> re模块在古老的`Python 1.5`版中引入，用于替换那些已过时的regex模块和regsub模块---这两个模块在`Python 2.5`版中一移除，而且此后导入这两个模块中的任意一个都会触发`ImportError`异常。
>
> re模块支持更强大而且更统用的Perl风格（perl 5风格）的正则表达式，该模块允许多个线程共享同一个已编译的正则表达式对象，也支持命名子组

## re模块的核心函数与方法：

---------------------------------------------------------------------------------------------------	
* __仅仅是re模块函数__

|函数/方法|描述|
|:--|:---|
|compile(pattern, flags = 0)|使用任何可选的标记阿莱编译正则表达式的模式，然后返回一个正则表达式对象|

* __re模块函数和正则表达式对象的方法__

|函数/方法|描述|
|:--:|:---|
|match(pattern, string, flags=0)|尝试使用带有可选标记的正则表达式的模式来匹配字符串。如果匹配成功，就返回匹配对象，否则就返回`None`|
|search(pattern, string, flags=0)|使用可选标记搜索字符串中第一次出现的正则表达式模式，并返回一个匹配列表|
|findall(pattern, string[,flags])|查找字符串中所有（非重复）出现的正则表达式模式，并返回一个匹配列表|
|finditer(pattern, string[,flags])|与`findall()`函数相同，但返回的不是一个列表，而是一个迭代器。对于每一次匹配，迭代器都会返回一个匹配对象|
|split(pattern, string, max=0)|根据正则表达式的模式分隔符，`split`函数将字符串分割为列表，然后返回成功匹配的列表，分割最大操作`max`次（默认分割所有匹配成功的位置）|
|sub(pattern, repl, string, count=0)|使用`repl`替换所有正则表达式的模式在字符串中出现的位置，除非定义`count`否则就将替换所有出现的位置（另见`subn()`函数），该函数返回替换操作的数目|
|purge()|清楚隐式编译的正则表达式模式|
|group(num=0)|返回整个匹配对象，或者编号为`num`的特定子组|
|groups(default=None)|返回一个包含所有匹配子组的元组（如果没有匹配成功，就返回一个空元组）|
|groupdict(default=None)|返回一个包含所有匹配的命名子组的字典，所有的子组名称作为字典的键（如果没有匹配成功，就返回一个空字典）|

* __常用的模块属性（用于大多数正则表达式函数的标记）__

|函数/方法|描述|
|:--:|:---|
|re.I、 re.IGNORECASE|不区分大小写匹配。|
|re.L、 re.LOCALE|根据所使用的本地语言环境通过`\w`、`\W`、`\b`、`\B`、`\s`、`\S`实现匹配|
|re.M、 re.MULTILINE|`^`和`$`分别匹配目标字符串中行的起始和结尾，而不是严格匹配整个字符串的起始和结尾(多行字符串作为目标字符串时)|
|re.S、 re.DOTALL|`"."`(点号)通常匹配除`\n`(换行符)之外的所有单个字符；该标记表示`"."`(点号)能够匹配全部字符|
|re.X、 re.VERBOSE|通过反斜线转义，否则所有空格加上`#`（以及该行中后续所有文字）都被忽略，除非在一个字符类中或者允许注释并且提高可读性|


* 标记的使用：

> 这些标记也可以作为参数适用于大多数re模块函数。如果想要在方法中使用这些标记，他们必须已经集成到已编译的正则表达式的对象之中，或者需要使用直接嵌入到正则表达式的(?F)标记，其中F是一个或多个i（用于re.I/IGNORECASE）、m（用于re.M/MULTILINE）等，如果想要同时使用多个，就把它们放在一起，而不是使用按位或操作，例如：(?im)可以同时表示 re.IGNORECASE和re.MULTILINE

---------------------------------------------------------------------------------------------------	

### 匹配对象以及 group()和groups()方法

> 当处理正则表达式时，除了正则表达式对象外，还有一个对象类型：匹配对象，这些成功调用了`match()`或者`search()`返回的对象。匹配对象有两个主要的方法：`group()`和`groups()`

* group():
	* 要么返回整个匹配对象，要么根据要求返回特定子组。

* groups()：
	* 返回一个包含唯一或者全部子组的元组，如果没有子组的要求，当`group()`仍然返回整个匹配时，`groups()`返回一个空元组。

* re.match(): 试图从字符串的起始部分对模式进行匹配，如果匹配成功，就返回一个匹配对象，如果匹配失败，就返回None，匹配对象的`group()`方法能够用于显示那个成功的匹配。

```
m = re.match('foo', 'foo')
if m:
	m.group()

# 输出: 'foo'
```

* re.search()：
	* 当要搜索的模式出现在一个字符串中间的概率，远大于出现在字符串起始位置的概率时，可以选择使用`search()`，`search()`的工作方式与`match()`完全一致，不同之处在于`search()`会用它的字符串参数，在任意位置对给定正则表达式模式搜索第一次出现的匹配对象。如果搜索到成功的匹配，就会返回一个匹配对象，否则就返回`None`

	* `search()`函数不但会搜索字符串中第一次出现的位置，而且严格地对字符串从左到右搜索。

```
# match()与search()对比
c = 'abcfood'

m = re.match('foo', c)
if m:
	m.group()
else:
	print('Matching Failed.')

# 输出:  Matching Failed.

m = re.search('foo', c)
if m:
	m.group()
else:
	print('Matching Failed.')

# 输出： 'foo'

```

* 分组：

```
c = 'abc-123'

m = re.match('(\w+)-(\d+)', c)

m.group(0) --> 'abc-123'
m.group(1) --> 'abc'
m.group(2) --> '123'
m.groups() --> ('abc', '123')

```


* re.findall()
	* `findall()`查询某个字符串中正则表达式模式全部非重复出现的情况，`findall()`总是返回一个列表，如果`findall()`没有找到匹配的部分，就返回一个空列表，如果匹配成功，列表将包含所有匹配成功部分(从左到右按出现顺序排列)
```
c = 'my car, you car, other car.'
m = re.findall('car', c)
print(m)  -->  ['car', 'car', 'car']
```

<!-- * re.finditer()
	* `finditer()`是一个与`findall()`类似但比`findall()`更省内存的变体，明显的不同在于`finditer()`在匹配对象中迭代，，而`findall()`返回的是一个列表。
```
s = 'This and that'
re.i
```
 -->

* re.sub() -- re.subn()
	* `sub()`与`subn()`两者几乎一样，都是将字符串中匹配正则表达式的部分进行某种形式的替换，不同的是`subn()`还会额外返回一个表示替换的总数。
```
re.sub('a', 'b', 'result: aac')
'result: bbc'

re.subn('a', 'b', 'result: aac')
('result: bbc', 2)

```

* re.split()
	* `re`模块的`split()`方法与字符串的`split()`方法类似，但是与分割一个固定的字符串想比，它们基于正则表达式的模式分割字符串，还可以指定`max`参数指定最大分割次数。

```
re.split(':', 'str1:str2:str3')
['str1', 'str2', 'str3']

re.split(':', 'str1:str2:str3', 1)
['str1', 'str2:str3']

DATA = (
	'Mountain View, CA, 94040',
	'Sunnyvale, CA',
	'Los Altos, 94023',
	'Cupertino, 95014',
	'Palo Alto CA',
)

for i in DATA:
	print(re.split(r', | (?= (?:\d{5}|[A-Z]{2}))', i))

['Mountain View', 'CA', '94040']
['Sunnyvale', 'CA']
['Los Altos', '94023']
['Cupertino', '95014']
['Palo Alto CA']
``` 

### 扩展符号
* Python的正则表达式支持大量的扩展符号，通过使用`(?iLmsux)`系列选项，用户可以直接在正则表达式里面指定一个或多个标记。

* `re.I/IGNORECASE`
```
re.findall(r'(?i)yes', 'Yes? yEs! YES.')
['Yes', 'yEs', 'YES']

re.findall(r'(?i)th\w+', 'The quickest way is through this tunnel.')
['The', 'through', 'this']

re.findall(r'(?im)(^th[\w ]+', """
This line is the first,
another line,
that line, it's the best
""")
['This line is the first', 'that line']

# (?im) == re.I 与 re.M的集合，
re.M/MULTILINE：多行字符串作为目标字符串时，^和$不作为整个字符串的起始和结束，而作为整个字符串每一行的起始和结束。
```

* `re.S/DOTALL`
```
re.findall(r'th.+', '''
The first line,
the second line,
the third line
''')
['the second line', 'the third line']

re.findall(r'(?s)th.+', '''
The first line,
the second line,
the third line
''')
['the second line\nthe third line\n']
```

* `re.X/VERBOSE`
```
re.search(r'''(?x)
	\((\d{3})\)	# 区号
	[ ]			# 空白符
	(\d{3})		# 前缀
	-			# 横线
	(\d{4})		# 终点数字
''', '(800) 555-1212').groups()

('800', '555', '1212')
```