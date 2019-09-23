## 示例

```python
#!/usr/bin/python3
# -*- coding: UTF-8 -*-
import xlwt
import xlrd
import json
import glob
import os

title = {
  "IP": {
    "idx": 0,
    "width": 20
  },    
  "描述": {
    "idx": 1,
    "width": 15
  },
  "备份时间": {
    "idx": 2,
    "width": 20 
  },
  "备份大小": {
    "idx": 3,
    "width": 10
  },
  "文件名": {
    "idx": 4,
    "width": 20
  },
  "状态": {
    "idx": 5,
    "width": 10
  },
  "远程备份机": {
    "idx": 6,
    "width": 20
  },
  "远程备份方式": {
    "idx": 7,
    "width": 15
  },
  "远程备份路径": {
    "idx": 8,
    "width": 20
  },
  "压缩": {
    "idx": 9,
    "width": 10
  }
}

def get_files(backup_path):
  if os.path.exists(backup_path):
    files = glob.glob(backup_path + "*.log")
  else:
    print("%s no such file or directory." % backup_path)
    sys.exit(2)

  return files


wb = xlwt.Workbook(encoding = 'utf-8')
ws = wb.add_sheet('back info')

# write title
for k in title:
  ws.write(0,title[k]['idx'],k)
  ws.col(title[k]['idx']).width = 256 * title[k]['width']


files = get_files('/root/python/')
hight = 1

for file in files:
  with open(file) as f_obj:
    logs = json.load(f_obj) 
    width = 0
    for k in title:
      ws.write(hight, width, logs[k])
      width += 1
    
    else:
      hight += 1

wb.save('/tmp/test.xls')
```


