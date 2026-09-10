---
title: 【OpenCV】OpenCV的基本使用
date: 2026-03-06 17:19:44
tags:
  - OpenCV
---





## 1.安装

```bash
pip install cv2
```





## 2.基本使用





### 2-1.混合两张图片

```python
import cv2

img1 = cv2.imread("E:/OCR/PDF/a_pdf/0.png")
img2 = cv2.imread("E:/OCR/PDF/a_pdf/1.png")

alpha = 0.3
beta = 0.7

final_img = cv2.addWeighted(img1, alpha, img2, beta, 0.0)

cv2.imshow("final_img", final_img)
cv2.waitKey(0)
```







### 2-2.改变图像的对比度和亮度

- 对比度：最大像素强度和最小像素强度之差
- 亮度：指图像亮或暗的程度。通过给所有像素加上一个常数可以让图像变得更亮

```python
import cv2
import numpy as np

img = cv2.imread("img_path")

new_img = np.zeros(img.shape, img.dtype)

constract = 3.0 # 对比度为3
bright = 2 # 亮度为2

for y in range(img.shape[0]): # 循环图像宽度
    for x in range(img.shape[1]): # 循环图像高度
        for c in range(img.shape[2]): # 循环图像通道
            new_img[y, x, c] = np.clip(constract * img[y, x, c] + bright, 0, 255) # 限制具体图像的取值，范围限制为0~255，也就是每个通道的像素值

cv2.imshow("img", img)
cv2.imshow("new_img", new_img)
cv2.waitKey(0)
```



（特定像素值 * 对比度） + 亮度





### 2-3.往图像中添加文字

```python
cv2.putText()
```

接受参数：

- 要添加文字的图像
- 要添加的文字
- 文字在图像上的位置
- 字体类型，支持以下的字体：
  - FONT_HERSHEY_SIMPLEX
  - FONT_HERSHEY_PLAIN
  - FONT_HERSHEY_DUPLEX
  - FONT_HERSHEY_COMPLEX
  - FONT_HERSHEY_TRIPLEX
  - FONT_HERSHEY_COMPLEX_SMALL
  - FONT_HERSHEY_SCRIPT_SIMPLEX
  - FONT_HERSHEY_SCRIPT_COMPLEX
- 字体大小
- 字体颜色
- 字体粗细
- 使用的线型
  - FILLED：完全填充线
  - LINE_4：四连通线
  - LINE_8：八连通线
  - LINE_AA：防锯齿线



```python
import cv2
import numpy as np

img = cv2.imread("img_path")

cv2.putText(new_img, "This is a picture", (230, 50), cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2, cv2.LINE_AA)

cv2.imshow("img", img)
cv2.waitKey(0)
```





cv2.putText()不能在图像中添加中文，需要使用```PIL```

```python
import cv2
import numpy as np

img = cv2.imread("img_path")

if isinstance(img1, np.ndarray):
    imgPIL = Image.fromarray(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))

drawPIL = ImageDraw.Draw(imgPIL)
font = ImageFont.truetype("font/simsun.ttc", size=40, encoding="utf-8")
drawPIL.text((50, 20), "这是一张图片", (255, 255, 255), font=font)
imgPutText = cv2.cvtColor(np.asanyarray(imgPIL), cv2.COLOR_RGB2BGR)

cv2.imshow("imgPutText", imgPutText)
cv2.waitKey(0)
```





### 2-4.平滑图像



- 均值滤波器（cv2.blur()）
- 中值滤波器（cv2.medianBlur()）
- 高斯滤波器（cv2.GaussianBlur()）
- 双边滤波器（cv2.bilateralFilter()）



#### 1）均值滤波器



```python
import cv2
import numpy as np

img = cv2.imread("img_path")

img_mean = cv2.blur(img1, (5, 5))
cv2.imshow("img_mean", img_mean)
cv2.waitKey(0)
```



#### 2）中值滤波器

是基本的图像平滑滤波器之一，是一种非线性滤波器，通过求临近像素的中位数来消除图像中的黑色噪音。

```python
import cv2
import numpy as np

img = cv2.imread("img_path")

img_median = cv2.medianBlur(img1, 5)

cv2.imshow("img_median", img_median)
cv2.waitKey(0)
```





#### 3）高斯滤波器

```python
import cv2
import numpy as np

img = cv2.imread("img_path")

img_gaussian = cv2.GaussianBlur(img1, (5, 5), 0)

cv2.imshow("img_gaussian", img_gaussian)
cv2.waitKey(0)
```





#### 4）双边滤波器

```python
import cv2
import numpy as np

img = cv2.imread("img_path")

img_bilateral = cv2.bilateralFilter(img1, 9, 75, 75)

cv2.imshow("img_bilateral", img_bilateral)
cv2.waitKey(0)
```





### 2-5.改变图像的形状

- 侵蚀：物理边界的像素减少
- 扩张：物体边界的像素增加



定义邻域核（neighborhood kernel），有如下三种定义方式：

- MORPH_RECT：创建矩形核
- MORPH_CROSS：创建十字形核
- MORPH_ELLIPSE：创建椭圆形核





##### 1）侵蚀操作作用核内的最小值产生一个新的像素

以下使用一个3x1的矩阵来发现每一行的最小值。

对于第一个元素来说，计算从前一个单元开始，因为左边的单元没有值，所以当它然是空白的，这种做法称为填充（padding）。所以第一个最小值在空白、141和157之间产生。结果141最小，所以右边矩阵的第一个元素是141.

然后，核向右平移，现在要考虑的单元是141、157和65，这次的最小值是65，所以新矩阵的第二个元素是65。

第三次的时候，核要比较157、65和空白，因为没有第三单元，所以最小值是65，也就是新矩阵的最后一个元素的值。

对于每个单元执行这种操作，就会得到右边的矩阵。

![](data:image/png;base64,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)

##### 2）扩张操作作用核内的最大值产生一个新的像素

与侵蚀操作类似，只是取最大值

![](data:image/png;base64,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)







```cv2.getStructuringElement()```函数用于定义核，作为侵蚀或扩张函数的参数。

- 侵蚀或扩张类型
- 核的大小
- 使用核的起始点



```python
import cv2
import numpy as np

img = cv2.imread("img_path")

1 = 0
s2 = 10
s3 = 10

t1 = cv2.MORPH_RECT
t2 = cv2.MORPH_CROSS
t3 = cv2.MORPH_ELLIPSE


temp1 = cv2.getStructuringElement(t1, (2 * s1 + 1, 2 * s1 + 1), (s1, s1))
temp2 = cv2.getStructuringElement(t2, (2 * s2 + 1, 2 * s2 + 1), (s2, s2))
temp3 = cv2.getStructuringElement(t3, (2 * s3 + 1, 2 * s3 + 1), (s3, s3))

final1 = cv2.erode(img1, temp1)
final2 = cv2.erode(img1, temp2)
final3 = cv2.erode(img1, temp3)

cv2.imshow("final1", final1)
cv2.imshow("final2", final2)
cv2.imshow("final3", final3)
cv2.waitKey(0)
```





```python
import cv2
import numpy as np

img = cv2.imread("img_path")
d1 = 0
d2 = 10
d3 = 20

t1 = cv2.MORPH_RECT
t2 = cv2.MORPH_CROSS
t3 = cv2.MORPH_ELLIPSE


temp1 = cv2.getStructuringElement(t1, (2 * d1 + 1, 2 * d1 + 1), (d1, d1))
temp2 = cv2.getStructuringElement(t2, (2 * d2 + 1, 2 * d2 + 1), (d2, d2))
temp3 = cv2.getStructuringElement(t3, (2 * d3 + 1, 2 * d3 + 1), (d3, d3))

final1 = cv2.dilate(img1, temp1)
final2 = cv2.dilate(img1, temp2)
final3 = cv2.dilate(img1, temp3)

cv2.imshow("final1", final1)
cv2.imshow("final2", final2)
cv2.imshow("final3", final3)
cv2.waitKey(0)
```







### 2-6.实施图像阈值化



阈限化（thresholding）处理的主要原因是要做图像分割。

可以通过移除背景将物体从图像中提取出来。为此，需要先将图像转换成灰度格式，然后转换成二值格式——只有黑色与白色的图像。



提供一个参考像素值，然后将所有值大于或小于它的像素转换成黑色或白色。有如下五种阈限化：

- 二值：如果像素值大于参考像素值（阈值），就转换成白色（255），否则转换成黑色（0）
- 反向二值：如果像素值大于参考像素值（阈值），就转换成黑色（0），否则转换成白色（255）。刚好与普通二值型相反
- 截断：如果像素值大于惨开像素值（阈值），就转换成阈值，否则保持不变。
- 阈限到零：如果像素值大于参考像素值（阈值），则保持不变；否则转换成黑色（0）。
- 反向阈值到零：如果像素值大于参考像素值（阈值），就转换成黑色（0），否则保持不变。



使用```cv2.threshold()```函数做图像阈限化，参数如下：

- 要转换的图像
- 阈值
- 最大像素值
- 阈限化类型



```python
import cv2
import numpy as np

img = cv2.imread("img_path")

# 设置阈值类型
# 0 - Binary
# 1 - Binary Inverted
# 2 - Truncated
# 3 - Threshold To Zero
# 4 - Threshold To Zero Inverted

_, img0 = cv2.threshold(img, 50, 255, 0)
_, img1 = cv2.threshold(img, 50, 255, 1)
_, img2 = cv2.threshold(img, 50, 255, 2)
_, img3 = cv2.threshold(img, 50, 255, 3)
_, img4 = cv2.threshold(img, 50, 255, 4)

cv2.imshow("img0", img0)
cv2.imshow("img1", img1)
cv2.imshow("img2", img2)
cv2.imshow("img3", img3)
cv2.imshow("img4", img4)
cv2.waitKey(0)
```





### 2-7.计算梯度

使用索伯导数（Sobel derivative）做边缘检测。

边有两种方向：垂直方向和水平方向。针对这种算法，只有空间频率非常高的区域才算边。一个区域的空间频率是指这个区域内的细节丰富内容。



```cv2.Sobel()```的参数：

- 输入图像

- 输入图像的深度

  图像的深度越大，错过任一边界的概率就约低。可以根据需求试验下列所有参数，看看他们是否把边检测出来。

  深度可以是以下类型：

  - -1（与原始图像的深度相同）
  - cv2.CV_16S
  - cv2.CV_32F
  - cv2.CV_64F

- x的求导顺序

- y的求导顺序

- 核的大小

- 用于求导的缩放系数

- 作为标量加到公式中的差值

- 用于外推像素的边界类型

```python
import cv2
import numpy as np

img = cv2.imread("img_path")

cv2.GaussianBlur(img, (3, 3), 0)

gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

grad_x = cv2.Sobel(gray, cv2.CV_16S, 1, 0, ksize=3, scale=1, delta = 0, borderType=cv2.BORDER_DEFAULT)
grad_y = cv2.Sobel(gray, cv2.CV_16S, 0, 1, ksize=3, scale=1, delta = 0, borderType=cv2.BORDER_DEFAULT)
abs_grad_x = cv2.convertScaleAbs(grad_x)
abs_grad_y = cv2.convertScaleAbs(grad_y)

grad = cv2.addWeighted(abs_grad_x, 0.5, abs_grad_y, 0.5, 0)

cv2.imshow("grad", grad)

cv2.waitKey(0)
```





### 2-8.执行直方图均衡

直方图均衡用于调整图像的对比度。

我们首先画出像素强度分布的直方图，然后修改。

每张图像都关联一个累计概率函数。

直方图均衡使得这个函数具有线性趋势。

使用灰度图做直方图均衡

```cv2.equalizeHist()```

```python
import cv2
import numpy as np

img = cv2.imread("img_path")

img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
img_eqlzd = cv2.equalizeHist(img)

cv2.imshow("img", img)
cv2.imshow("img_eqlzd", img_eqlzd)
cv2.waitKey(0)
```

