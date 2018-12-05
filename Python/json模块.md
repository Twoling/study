### json模块
> JSON（JavaScript Object Notation）格式最初是为JavaScript开发的，但随后成了一种格式，被包括Python在内的众多语言采用

* json.dump()
	* 接受两个实参： 要存储的数据以及可用于存储数据的文件对象

栗子：

```
import json

numbers = [2, 3, 5, 7, 11, 13]

filename = 'number.json'
with open(filename, 'w') as f_obj:
	json.dump(numbers, f_obj)
```

* json.load():

栗子:

```
import json

filename = 'number.json'
with open(filename) as f_obj:
	numbers = json.load(f_obj)

print(numbers)
```

* json.dumps():
	* dumps是将dict转化成str格式

* json.loads():
	* loads是将str转化成dict格式