# AutoAlignForEclipse-Python
先設定 threshold 二值化影像並濾波降噪（median filter），再透過 OpenCV 找出外切矩形。

[HackMD｜完整影像处理过程](https://hackmd.io/@juian/SyHsyVN0I)

# Python 自動對齊

```python
import cv2 # 4.2.0
from scipy import ndimage # median filter
```
- [OpenCV 輪廓形狀擬合](http://www.pianshen.com/article/4590348606/)
- [median filter](https://docs.scipy.org/doc/scipy/reference/generated/scipy.ndimage.median_filter.html#scipy.ndimage.median_filter)：影像處理中，通常需要先進行降噪。

```python
def img_process(img, threshold, mf_size, extend):
    # 影像二值化
    ret, thresh = cv2.threshold(img, threshold, 255, cv2.THRESH_BINARY)
    # median filter
    thresh = ndimage.median_filter(thresh, size=mf_size, mode="nearest")
    # 外切矩形（不一定是正方形）
    x, y, w, h = cv2.boundingRect(thresh)
    # 往外延伸 extend 個像素
    crop_img = img[y - extend : y + h + extend, x - extend : x + w + extend]
    return crop_img
```
```python 
def crop(file, output):
    # read image
    img = cv2.imread(file, cv2.IMREAD_GRAYSCALE)
    # 調用 img_process
    crop_img = img_process(img=img, threshold=10, mf_size=5, extend=20)
    # 條件太鬆導致裁到很多背景
    # threshold 和 median filter 的 size 都變兩倍
    # 1700000 是後驗估計的，如果條件太鬆裁到很多背景，那麼裁出来的 size 就會遠遠大於平均值
    i = 2
    while crop_img.size > 1700000:
        crop_img = img_process(img=img, threshold=10 * i, mf_size=5 * i, extend=20)
        i += 1
        if i > 3:
            print(file, "too many background pixels!")
            break
    # write image
    cv2.imwrite(output, crop_img, [cv2.IMWRITE_JPEG_QUALITY, 100])
```
單張影像
```python
crop("original/021_95%.jpg", "crop/crop_021_95%.jpg")
```
資料夹
```python
import os

for f in os.listdir("original"):
    if ".jpg" in f:
        try:
            crop("original/" + f, "crop/crop_" + f)
        except Exception as e:
            print("crop_" + f, e)
```
食分大的偏食可能會得到矩形的結果，但是不礙事。以我的影像為例，C2 前的影像全部向上向左對齊，C3 後的影像全部向下向右對齊，就能在正確的位置對齊了。

![](https://i.imgur.com/3qhRSY7.jpg)


**Tips**：每一個圖層都轉存出來：`檔案` > `轉存` > `圖層轉存檔案`

# 成果
> 偏食階段（5 分鐘拍一張）每張停留 0.1 秒，環食階段（1 秒拍一張）每張停留 1/30 秒

> 利用 clideo 壓縮影片

![](https://i.imgur.com/NPTcmeO.gif)
