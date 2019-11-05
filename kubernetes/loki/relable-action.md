# Relable 操作示例

### action 
#### replace
replace 将 source_label 指定的标签替换为 target_label，value保持不变

示例:
```yaml
# 示例 Label
# __meta_kubernetes_namespace

  relabel_configs:
  - action: replace
    source_labels:
    - __meta_kubernetes_namespace
    target_label: namespace
```

结果:
```yaml
# __meta_kubernetes_namespace
namespace
```

示例:
```yaml
# 示例Label
# __meta_kubernetes_namespace: kube-system
# __service__: coredns

  relabel_configs:
  - action: replace
    replacement: $1
    separator: /
    source_labels:
    - __meta_kubernetes_namespace
    - __service__
    target_label: job
```
从 source_labels 获取要操作的 labels 列表，并获取 label 的值，使用 separtor 指定的分隔符拼凑出$1，而后将值赋予给 target_label指定的新 label


结果:
```
job: kube-system/coredns
```

#### drop
drop 查看source_labels定义的标签的值是否匹配 regex 表达式，如果匹配，则删除相应的label

示例:
```yaml
# 示例Label
# __service__: ''

  relabel_configs:
  - action: drop
    regex: ''
    source_labels:
    - __service__
```

结果: 当 `__service__` 的值为空时，执行删除操作

#### labelmap
labelmap 将 regex 匹配结果分组中将第一组($1)做为key，结果作为 value，生成新的label

示例:
```yaml
# 示例 Lable
# __meta_kubernetes_pod_label_app: myapp
# __meta_kubernetes_pod_label_type: acts

  relabel_configs:
  - action: labelmap
    regex: __meta_kubernetes_pod_label_(.+)
```

结果:
```
# __meta_kubernetes_pod_label_app: myapp 
app: myapp

# __meta_kubernetes_pod_label_type: acts
type: acts
```



