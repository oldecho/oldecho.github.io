---
title: python 3 安装 zbarlight
date: 2018-05-02 15:28:07
link:
categories:
- python
tags:
- python
- zbarlight
---

python 二维码识别库 zbarlight 安装小记

开发环境，python 3.6（64 位）（看到其他小伙伴提到了 32 / 64 位的问题，此处 32 位未做测试）

1. clone or download [zbarlight](https://github.com/Polyconseil/zbarlight) 和 [zbar](https://github.com/ZBar/ZBar) 到本地

2. 将 zbar 里的 include 文件夹 copy 放到 zbarlight 下的 src 文件夹内

3. 修改 zbarlight 根目录下的 setup.py 文件

   ```python
   ext_modules=[
           Extension(
               ...
               include_dirs = ['src\include'],
           ),
       ],
   ```

4. 将 zbarlight 文件夹复制到 python site-packages 下

到此可以正常使用，示例：

```python
def check_qr(file_path):
    """
    检查二维码
    """
    qr = 0
    with open(file_path, 'rb') as image_file:
        image = Image.open(image_file)
        image.load()
    codes = zbarlight.scan_codes('qrcode', image)
    print('QR codes: %s' % codes)
    if codes is not None:
        qr = 1
    return qr
```

[参考博客](https://www.jianshu.com/p/f968527e1d5e) 