---
title: 【Image Process】Scikit Image的基本使用
date: 2026-03-05 18:50:36
tags:
  - Python
---



## 1.安装

```bash
pip install scikit-image matplotlib
```





## 2.基本使用



### 2-1.上传和查看图像

```python
from skimage import io

img = io.imread("img_path")
io.imshow(img)
io.show()
```





### 2-2.获取图像分辨率

```python
from skimage import io

img = io.imread("img_path")
print(img.shape) // (538, 354, 3)
```





### 2-3.查看像素值

```python
from skimage import io
import pandas as pd

img = io.imread("img_path")
df = pd.DataFrame(img.flatten()) // 把RGB图像的三个维度转换成一维

filePath = "file_path"
df.to_excel(filePath, index=False)
```





### 2-4.转换色彩空间



#### 1）RGB<->HSV

```python
from skimage import io, color
from pylab import *

img = io.imread("img_path")

img_hsv = color.rgb2hsv(img)
img_rgb = color.hsv2rgb(img_hsv)

figure(0)
io.imshow(img_hsv)
figure(1)
io.imshow(img_rgb)
io.show()
```



#### 2）RGB<->XYZ

```python
from skimage import io, color
from pylab import *

img = io.imread("img_path")

img_xyz = color.rgb2xyz(img)
img_rgb = color.xyz2rgb(img_xyz)

figure(0)
io.imshow(img_xyz)
figure(1)
io.imshow(img_rgb)
io.show()
```



#### 3）RGB<->LAB

```python
from skimage import io, color
from pylab import *

img = io.imread("img_path")

img_lab = color.rgb2lab(img)
img_rgb = color.lab2rgb(img_lab)

figure(0)
io.imshow(img_lab)
figure(1)
io.imshow(img_rgb)
io.show()
```





#### 4）RGB<->YUV

```python
from skimage import io, color
from pylab import *

img = io.imread("img_path")

img_yuv = color.rgb2yuv(img)
img_rgb = color.yuv2rgb(img_yuv)

figure(0)
io.imshow(img_yuv)
figure(1)
io.imshow(img_rgb)
io.show()
```



#### 5）RGB<->YIQ

```python
from skimage import io, color
from pylab import *

img = io.imread("img_path")

img_yiq = color.rgb2yiq(img)
img_rgb = color.yiq2rgb(img_yiq)

figure(0)
io.imshow(img_yiq)
figure(1)
io.imshow(img_rgb)
io.show()
```



#### 6）RGB<->YPbPr

```python
from skimage import io, color
from pylab import *

img = io.imread("img_path")

img_ypbpr = color.rgb2ypbpr(img)
img_rgb = color.ypbpr2rgb(img_ypbpr)

figure(0)
io.imshow(img_ypbpr)
figure(1)
io.imshow(img_rgb)
io.show()
```







### 2-5.保存图像



```python
from skimage import io
from skimage.draw import (line)

img = io.imread("img_path")

io.imsave("save_path", img)
```





### 2-6.创建基本图形



#### 1）直线

```python
from skimage import io
from skimage.draw import (line)

img = io.imread("img_path")

x, y = line(0, 0, 20, 20)
img[x, y] = 0

io.imshow(img)
io.show()
```



#### 2）矩形

```python
from skimage import io
from skimage.draw import (polygon)

img = io.imread("img_path")

def rectangle(x, y, w, h):
    rr, cc = [x, x + w, x + w, x], [y, y, y + h, y + h]
    return polygon(rr, cc)

rr, cc = rectangle(10, 10, 50, 50)
img[rr, cc] = 0

io.imshow(img)
io.show()
```





#### 3）圆形

```python
from skimage import io
from skimage.draw import (circle_perimeter)

img = io.imread("img_path")

x, y = circle_perimeter(100, 100, 100)
img[x, y] = 1

io.imshow(img)
io.show()
```





#### 4）贝塞尔曲线

```python
from skimage import io
from skimage.draw import (bezier_curve)

img = io.imread("img_path")

x, y = bezier_curve(0, 0, 100, 100, 200, 300, 50)
img[x, y] = 1

io.imshow(img)
```



### 2-7.执行伽马矫正

```python
from skimage import io
from skimage.exposure import (adjust_gamma)

img = io.imread("img_path")

gamma = adjust_gamma(img, 0.5)

io.imshow(gamma)
io.show()
```





### 2-8.旋转、平移和缩放图像



#### 1）旋转

```python
from skimage import io
from skimage.transform import (rotate)

img = io.imread("img_path")

img_rot = rotate(img, 20)

io.imshow(img_rot)
io.show()
```



#### 2）缩放

```python
from skimage import io
from skimage.transform import (resize)

img = io.imread("img_path")

img_res = resize(img_original, (600, 400))

io.imshow(img_res)
io.show()
```





### 2-9.确定结构相似度



```python
from skimage import io
from skimage.metrics import structural_similarity as ssim

img_original = io.imread("img_original")
img_modified = io.imread("img_modified")

score, diff = ssim(img_original, img_modified, full=True, channel_axis=-1)  # diff 为差异图像，可用于可视化

print(f"SSIM 值: {score:.4f}")
```



