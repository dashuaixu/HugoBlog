+++
title = 'Python批量解压zip 得到的文件名乱码问题'
date = 2024-03-11T22:50:56+08:00
draft = false
tags = ["Python"] 
categories = [ "Python"]
+++
## 场景：
解压几万个zip包，其中得到的少部分文件名是乱码的，大部分没问题。

## 功能：
- [x] 批量解压某个文件夹内的zip包到目标文件夹

- [x] 修复乱码的文件名

## 工具包：

- import os
- import zipfile

---

批量解压：

```python
import os
import zipfile


open_path = '/Downloads/docs'
save_path = '/Downloads/temp'

def Decompression(files, file_path, save_path):
    for file_name in files:
        os.chdir(file_path)  # 转到路径
        print(file_name)
        if file_name.endswith('zip'):  # 判断是否解压文件
            try:
                zpfd = zipfile.ZipFile(file_name)  # 读取压缩文件
                os.chdir(save_path)  # 转到存储路径
                zpfd.extractall()
                zpfd.close()
            except:
                pass
        else:
            print(f'!!!!{file_name}')


def main(open_path):
    for file_path, sub_dirs, files in os.walk(open_path):  # 获取所有文件名，路径
        print(file_path, sub_dirs, files)
        Decompression(files, file_path, save_path)

main(open_path)
```



---

解压完修复显示错误文件名：

```python
import os


type = 'pdf'

dir = f"/Users/spider/Downloads/{type}s/all_{type}_hx"
for file in os.listdir(dir):
    if file.endswith(f'.{type}') or file.endswith(f'.{type}x'):
        try:
            # 对文件名用cp437解码成byte，然后用gbk编码成中文string，然后转成utf8
            new_name = file.encode('cp437').decode('gbk').encode().decode()
            os.rename(os.path.join(dir, file), os.path.join(dir, new_name))
        except:
            # 解码报错的文件说明实际显示正确，无须理会
            pass
```

