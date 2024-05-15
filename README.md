# 基于传统图形学的车牌识别系统

```python
#导入工具包
import os

import cv2
import cv2 as cv
import numpy as np
import matplotlib.pyplot as plt
import  matplotlib
matplotlib.use('Qt5Agg')  # 或者 'TkAgg'
from matplotlib.ticker import MultipleLocator, FormatStrFormatter
test="C:\project/tmplate_match/"   #方便读取地址
temp_test="C:\project/test001/"
template = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
            'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I','J', 'K', 'L', 'M', 'N','O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W',
            'X',
            '陕','藏','云','贵','川','渝','琼','桂','粤','沪','黑','吉','辽','蒙','晋','冀','津','澳','港','新','宁','青','甘','湘','鄂','豫','鲁','赣','闽'
,'皖','浙','京','苏','Y','Z']
#自定义函数
def cv_show(img) :
    cv.imshow('names',img)
    cv.waitKey(0)
    cv.destroyAllWindows()
def plt_show(img):
    # 设置坐标轴的起点和终点
    img0=img.copy()
    origin = (50, img0.shape[0] - 50)
    x_end = (img0.shape[1] - 50, img0.shape[0] - 50)
    y_end = (50, 50)

    # 绘制坐标轴
    cv2.line(img0, origin, x_end, (255, 0, 0), 2)  # X轴，蓝色
    cv2.line(img0, origin, y_end, (0, 255, 0), 2)  # Y轴，绿色

    # 设置刻度
    for i in range(50, img0.shape[1] - 50, 50):  # X轴刻度
        cv2.line(img0, (i, img0.shape[0] - 48), (i, img0.shape[0] - 52), (255, 255, 255), 2)
        cv2.putText(img0, str((i - 50) // 50), (i - 10, img0.shape[0] - 30), cv2.FONT_HERSHEY_SIMPLEX, 0.5,
                    (255, 255, 255), 2)

    for i in range(50, img0.shape[0] - 50, 50):  # Y轴刻度
        cv2.line(img0, (48, i), (52, i), (255, 255, 255), 2)
        cv2.putText(img0, str((img0.shape[0] - 50 - i) // 50), (20, i + 5), cv2.FONT_HERSHEY_SIMPLEX, 0.5,
                    (255, 255, 255), 2)
    cv_show(img0)

#高斯模糊
def gauss(img):
    img = cv2.GaussianBlur(img, (5, 5), 3)
    return img
#sobel算子
def  sobel(img):
    sobelx=cv.convertScaleAbs(cv.Sobel(img,cv.CV_64F,1,0,ksize=3))
    return sobelx

#读取与预处理图像
img=cv.imread(test+"project_test000.jpg")
img=cv.resize(img,(0,0),fx=0.2,fy=0.2)
img1=cv.cvtColor(img,cv.COLOR_BGR2GRAY)
imgc=img.copy()
h,w=img[:,:,0].shape
print(f"{h} {w}")
cv_show(img)
cv_show(img1)
plt_show(img)
#形态学操作

#高斯及其灰度显示
imgc_gray=cv.cvtColor(imgc,cv.COLOR_BGR2GRAY)
imgc_gray=gauss(imgc_gray)
cv_show(imgc_gray)

#sobel强化边缘信息
imgcc=sobel(imgc_gray)
cv_show(imgcc)
#转为单纯黑白的二值化图像
imgcc=cv.threshold(imgcc,0,255,cv2.THRESH_OTSU)[1]

cv_show(imgcc)

#形态学操作

#形态学操作核心：卷积核
kernel=cv.getStructuringElement(cv.MORPH_RECT,(30,10))

#产生块状图，取出车牌信息
imgcc=cv.morphologyEx(imgcc,cv.MORPH_CLOSE,kernel,iterations=1)

cv_show(imgcc)

kernelx=cv2.getStructuringElement(cv.MORPH_RECT,(50,1))
kernely=cv2.getStructuringElement(cv.MORPH_RECT,(1,20))

#x方向进行闭操作
imgcc=cv.dilate(imgcc,kernelx)
imgcc=cv.erode(imgcc,kernelx)

#y方向进行开运算
imgcc=cv.erode(imgcc,kernely)
imgcc=cv.dilate(imgcc,kernely)

#中值滤波(去除噪音
imgcc=cv.medianBlur(imgcc,51)

cv_show(imgcc)
plt_show(imgcc)
contours,hierarchy=cv.findContours(imgcc,cv.RETR_EXTERNAL,cv.CHAIN_APPROX_SIMPLE)

for i in contours:
    j=cv.boundingRect(i)
    x=j[0]
    y=j[1]
    weight=j[2]
    height=j[3]
    if(weight>(height*3.5)) and (weight<(height*5)):
        imgcc=img[y:y+height,x:x+weight]
        cv_show(imgcc)

gray=cv.cvtColor(imgcc,cv.COLOR_BGR2GRAY)
gray=gauss(gray)
ret, imgccc = cv2.threshold(gray, 0, 255, cv2.THRESH_OTSU)
cv_show(imgccc)

#膨胀切割
kernel=cv.getStructuringElement(cv.MORPH_RECT,(2,2))
imgccc=cv.dilate(imgccc,kernel)
cv_show(imgccc)

#查找轮廓
contours,hierarchy=cv.findContours(imgccc,cv.RETR_EXTERNAL,cv.CHAIN_APPROX_SIMPLE)
words=[]
word_img=[]
#对所有轮廓逐一操作
for i in contours:
    word=[]
    ret=cv.boundingRect(i)
    x=ret[0]
    y=ret[1]
    weight=ret[2]
    height=ret[3]
    word.append(x)
    word.append(y)
    word.append(weight)
    word.append(height)
    words.append(word)
words=sorted(words,key=lambda s:s[0],reverse=False)
i=0
plt_show(imgccc)
#word中存放轮廓的起始点和宽高
for word in words:
    #筛选字符的轮廓
    if (word[3] > (word[2] * 1.5)) and (word[3] < (word[2] * 3.5)) and (word[2] > 25):
        i+=1
        img_s=imgccc[word[1]:word[1] + word[3], word[0]:word[0] + word[2]]
        word_img.append(img_s)
        print(i)
print(words)
for i,j in enumerate(word_img):
    plt.subplot(1,7,i+1)
    plt.imshow(word_img[i],cmap='gray')
plt.show()






# 读取一个模板地址与图片进行匹配，返回得分
def template_score(template, image):
    # 将模板进行格式转换
    template_img = template.copy()
    template_img = cv2.cvtColor(template_img, cv2.COLOR_RGB2GRAY)
    # 模板图像阈值化处理——获得黑白图
    ret, template_img = cv2.threshold(template_img, 0, 255, cv2.THRESH_OTSU)
    #     height, width = template_img.shape
    #     image_ = image.copy()
    #     image_ = cv2.resize(image_, (width, height))
    image_ = image.copy()
    # 获得待检测图片的尺寸
    height, width = image_.shape
    # 将模板resize至与图像一样大小
    template_img = cv2.resize(template_img, (width, height))
    # 模板匹配，返回匹配得分
    result = cv2.matchTemplate(image_, template_img, cv2.TM_CCOEFF)
    return result[0][0]

# 对分割得到的字符逐一匹配
def template_matching(word_img):
    results = []
    results_img=[]
    index=0
    x = 0
    for  word_imgs in word_img:
        #汉字
        op = word_imgs.copy()
        print(x)
        if index == 0:
            res=0

            nans=word_imgs.copy()
            for i in range(34,67):
              r = cv.imread(temp_test + f"{i}.png").copy()
              ans=template_score(r,nans)
              if ans>=res:
                  res=ans
                  x=i
                  op=r.copy()
            results.append(template[x])
            results_img.append(op)
        #字母
        elif index == 1:
            res=0
           # op=word_imgs.copy()
            #cv_show(op)
            for i in range(10,34):
              r = cv.imread(temp_test + f"{i}.png").copy()
              #cv_show(r)
              ans=template_score(r,word_imgs)
              if ans>res:
                  res=ans
                  x=i
                  op=r.copy()
            for i in range(67,69):
              r = cv.imread(temp_test + f"{i}.png").copy()
              ans=template_score(r,word_imgs)
              if ans>res:
                  res=ans
                  op=r.copy()
            results.append(template[x])
            results_img.append(op)
        #匹配数字与字母
        else:
            res=0
            #op=word_imgs.copy()
            for i in range(0,35):
              r = cv.imread(temp_test + f"{i}.png").copy()
              ans=template_score(r,word_imgs)
              if ans>res:
                  x=i
                  res=ans
                  op=r.copy()
            for i in range(67,69):
              r = cv.imread(temp_test + f"{i}.png").copy()
              ans=template_score(r,word_imgs)
              if ans>res:
                  res=ans
                  x=i
                  op=r.copy()
            results.append(template[x])
            results_img.append(op)
        cv_show(op)
        index += 1
    return results,results_img



res,ans=template_matching(word_img)
#cv_show(res[1])
print(res)


```
