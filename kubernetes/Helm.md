# Helm 学习
## Architecture
`Helm` 是一个用于管理 `Kubernetes` 工作负载的工具, 其将一系列的声明式文件以固定的格式组织成一个包, 称为 `charts`,  `Helm` 可以执行以下操作
  * 从头创建一个新的 `charts`
  * 将 `charts` 打包成 `charts` 归档格式(tgz)文件
  * 与 `charts` `repository` 交互
  * 将 `charts` 安装或卸载到现有的 `Kubernetes` 集群中
  * 管理使用 `Helm` 安装的 `charts` 的发布周期

对于 `Helm`, 有三个重要概念：
  1. `charts` 是创建 `Kubernetes` 应用程序所必须的一组信息
  2. `config` 包含的配置信息可以合并到 `charts` 中以用来创建应用程序
  3. 结合特定的配置, 每个 `release` 都是一个运行的 `charts` 实例

## 组件
**Helm Client**
`helm client` 是用户可以执行的的命令行客户端，其可以实现以下功能：
  * 本地 `charts` 开发
  * 管理 `charts` 仓库
  * 与 `Tiller Server` 进行交互
    * 发送要安装的 `charts`
    * 查询 `release` 的相关信息
    * 请求更新或者卸载存在的 `release`

**Tiller Server**
`Tiller server` 是一个运行在集群内的服务端程序, 与 `helm` 客户端交互，并与 `Kubernetes API` 连接, `Tiller server` 负责以下事项：
  * 监听来自 `helm` 客户端的请求
  * 结合 `charts` 和 `config` 来构建 `release`
  * 将 `charts` 安装到 `Kubernetes` 中, 然后追踪后续版本
  * 通过与 `Kubernetes` 交互来升级和卸载 `charts`

**简单来说，Helm客户端负责管理charts, Tiller server负责管理release**

## Helm 命令
### helm create
* 创建 `charts`
```shell
# 命令格式
helm create NAME [flags]
# 示例
helm create myapp
# 文件组织格式
myapp
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```



## 模板开发
### 函数
* `quote`: 引用, 将传递的字符串加双引号
* `upper`: 大写转换，将传递的字符串转换为大写
* `repeat NUM`: 重复，将传递的参数重复指定次数
* `default`: 默认值, 当给定值为空时, 赋予默认值
* `indent NUM`: 告诉模板引擎应该缩进几个空格，需顶格写，不然会在当前缩进的前提下再缩进给定的量
* `title`: 转换第一个字符为大写

#### 元素符
* eq
* ne
* lt
* gt
* and
* or
* not

#### 流程控制
控制结构为模板提供了生成流程的能力，helm的模板语言提供一下控制结构：

* `if`/`else`：条件判断
* `with`：定义范围
* `range`：循环

除此之外，还提供了一些用于生命和使用命名模板的操作

* `define`:  在模板中声明一个新的模板
* `template`: 导入命名模板
* `block`: 声明一种特殊的可填写模板区域

##### if/else
* 条件判断基本语法
```template
{{ if PIPELINE }}
  # Do something
{{ else if OTHER PIPELINE }}
  # Do something else
{{ else }}
  # Default case
{{ end }}
```

如果判断返回值为以下值，则返回结果为 `false`
  * 布尔值：false
  * 数字0
  * 空字符串
  * nil(空或者null)
  * 空集合(map, slice, tuple, dict, array)
除此之外的其他情况，返回结果将被认为true

##### 示例：
* 模板文件: templates/configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ .Release.Name }}-configmap
data:
    myvalue: "Hello World"
    drink: {{ .Values.favorite.drink | default "tea" | quote }}
    food: {{ .Values.favorite.food | upper | quote }}
    {{if eq .Values.favorite.drink "coffee"}}
    mug: true
    {{end}}
```

* 值文件: myapp/values.yaml
```yaml
favorite:
  drink: coffee
  food: pizza
```

* 测试结果
```bash
helm install --dry-run --debug ./myapp
```
```yaml
...
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: telling-chimp-configmap
data:
    myvalue: "Hello World"
    drink: "coffee"
    food: "PIZZA"

    mug: true
...
```
> 可以看到生成的YAML中产生了一些空格，这是因为模板引擎在运行的时候，只删除`{{}}`里的内容，但是还是会留下之后的空格
> 模板引擎支持一些语法可以修剪多余的空格，通过使用 `{{-` 或者 `-}}` 来删除空格，`{{-` 删除左边的库空格，`-}}` 删除右边的空格
>
> 注意：`-`指令与其余部分之间必须要有空格

* 例：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ .Release.Name }}-configmap
data:
    myvalue: "Hello World"
    drink: {{ .Values.favorite.drink | default "tea" | quote }}
    food: {{ .Values.favorite.food | upper | quote }}
    {{- if eq .Values.favorite.drink "coffee"}}
    mug: true
    {{- end}}
```
* 测试结果
```bash
helm install --dry-run --debug ./myapp
```
```yaml
...
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: telling-chimp-configmap
data:
    myvalue: "Hello World"
    drink: "coffee"
    food: "PIZZA"
    mug: true
...
```
##### WITH
* 语法格式
```
{{ with PIPELINE }}
  # restricted scope
{{ end }}
```

* `WITH` 允许你将给定的范围设置为`.`
* 例：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ .Release.Name }}-configmap
data:
    myvalue: "Hello World"
    {{- with .Values.favorite }}
    drink: {{ .drink | default "tea" | quote }}
    food: {{ .food | upper | quote }}
    {{- end }}
```
* 输出结果
```yaml
# Source: myapp/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: loitering-lamb-configmap
data:
    myvalue: "Hello World"
    drink: "coffee"
    food: "PIZZA"
```

##### range
* 语法格式
```
{{- range PIPELINE }}
{{ . }}
{{- end }}
```

* 示例：
```
# values.yaml
favorite:
  drink: coffee
  food: pizza
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
```
```
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
  {{- with .Values.favorite }}
  drink: {{ .drink | default "tea" | quote }}
  food: {{ .food | upper | quote }}
  {{- end }}
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}
```
* 结果输出
```bash
~]# helm install --dry-run --debug  ./
...
# Source: myapp/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: quieting-zebu-configmap
data:
    myvalue: "Hello World"
    drink: "coffee"
    food: "PIZZA"
    toppings: |-
      - "Mushrooms"
      - "Cheese"
      - "Peppers"
      - "Onions"
...
```
`|-`在YAML文件中，标记使用多行字符串
* `range` 可以很方便的生成列表
* 例如：
```
sizes: |-
    {{- range list "small" "medium" "large" }}
    - {{ . }}
    {{- end }}
```
生成结果
```
sizes: |-
    - small
    - medium
    - large
```


### Variables
分配变量需要特殊的赋值运算符: `:=`
* 例：
```
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ .Release.Name }}-configmap
data:
    myvalue: "Hello World"
    # 在with语法开始之前，引入其他对象赋值于变量，避免with语法生效时重新设置`.`导致引用失效
    {{- $relname := .Release.Name -}}
    {{- with .Values.favorite }}
    drink: {{ .drink | default "tea" | quote }}
    food: {{ .food | upper | quote }}
    release: {{ $relname }}
    {{- end }}
```
* 生成结果
```
# Source: myapp/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: exhaling-dragonfly-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "PIZZA"
  release: exhaling-dragonfly
```

* 变量在`range` 中的应用
```
toppings: |-
  {{- range $index, $topping := .Values.pizzaToppings }}
    {{ $index }}: {{ $topping }}
  {{- end }}
```

* 生成的结果
```
toppings: |-
    0: mushrooms
    1: cheese
    2: peppers
    3: onions
```

* 键值对生成
```
apiVersion: v1
kind: ConfigMap
metadata:
    name: {{ .Release.Name }}-configmap
data:
    myvalue: "Hello World"
    {{- range $key, $val := .Values.favorite }}
    {{ $key }}: {{ $val | quote }}
    {{- end}}
```
* 生成的结果
```
# Source: myapp/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: funny-lionfish-configmap
data:
  myvalue: "Hello World"
  drink: "coffee"
  food: "pizza"
```
