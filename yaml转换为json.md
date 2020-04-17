## 从 YAML 文件中读取数据转换为字典或者 json 格式

##### 示例文件
```
# 1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myap
  name: myap
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myap
  template:
    metadata:
      labels:
        app: myap
    spec:
      containers:
      - image: myapp:v1
        livenessProbe:
          tcpSocket:
            port: 80
        name: myap
        readinessProbe:
          tcpSocket:
            port: 80
```

* 转换为 dict
```
# -*- coding: utf-8 -*-
#

import yaml     # 需要安装 pyyaml 模块

example_file = '1.yaml'
yf = open(example_file, 'r')

#
dict_context = yaml.safe_load(yf.read())
print(dict_context)

# 输出
# {'apiVersion': 'apps/v1', 'kind': 'Deployment', 'metadata': {'labels': {'app': 'myap'}, 'name': 'myap'}, 'spec': {'replicas': 1, 'selector': {'matchLabels': {'app': 'myap'}}, 'template': {'metadata': {'labels': {'app': 'myap'}}, 'spec': {'containers': [{'image': 'myapp:v1', 'livenessProbe': {'tcpSocket': {'port': 80}}, 'name': 'myap', 'readinessProbe': {'tcpSocket': {'port': 80}}}]}}}}
```

* 转换为 json
```
# -*- coding: utf-8 -*-
#

import yaml     # 需要安装 pyyaml 模块
import json

example_file = '1.yaml'
yf = open(example_file, 'r')

# 先转换为 dict
dict_context = yaml.safe_load(yf.read())
print(dict_context)

# 再转换为 json
json_context = json.dumps(dict_context)
print(json_context)
# 输出
# {"apiVersion": "apps/v1", "kind": "Deployment", "metadata": {"labels": {"app": "myap"}, "name": "myap"}, "spec": {"replicas": 1, "selector": {"matchLabels": {"app": "myap"}}, "template": {"metadata": {"labels": {"app": "myap"}}, "spec": {"containers": [{"image": "myapp:v1", "livenessProbe": {"tcpSocket": {"port": 80}}, "name": "myap", "readinessProbe": {"tcpSocket": {"port": 80}}}]}}}}
```

### 将 json 数据以 yaml 格式写入文件

```
import yaml

json_data = '{"apiVersion": "apps/v1", "kind": "Deployment", "metadata": {"labels": {"app": "myap"}, "name": "myap"}, "spec": {"replicas": 1, "selector": {"matchLabels": {"app": "myap"}}, "template": {"metadata": {"labels": {"app": "myap"}}, "spec": {"containers": [{"image": "myapp:v1", "livenessProbe": {"tcpSocket": {"port": 80}}, "name": "myap", "readinessProbe": {"tcpSocket": {"port": 80}}}]}}}}'

# 转换为
with open('2.yaml', 'w+') as yf:
    json_data = yaml.safe_load(json_data)
    yf.write(yaml.dump(json_data, default_flow_style=False))
```