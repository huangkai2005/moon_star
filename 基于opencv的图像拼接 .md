# 基于opencv的图像拼接

```python
import cv2
import cv2 as cv
import  numpy as np
import time
import  matplotlib.pyplot as plt
import matplotlib
"""
图像拼接输出的是多个输出图像的并集,通常用到四个步骤：
1.特征提取(Feature Extraction):检测输入图像的特征点
2.图像配准(Image Registration):建立图像之间的几何对应关系，使它们可以在一个共同的参照系中进行变换,比较和分析
3.图像变形(Warping):图像变形指的是将一副图像的图像重投影,并将图像放置在更大的画布上
4.图像融合(Blending):图像融合是通过改变边界附近的图像灰度级，去除这些缝隙，创建混合图像，从而在图像之间实现平滑过渡。混合模式用于将两层图像,
融合在一起
"""
test="D:\project.test\Feature matching_test/"
test2="C:\project\(baseed-opencv)Image stitching/"
def cv_show(img,x):
    cv.imshow('names',img)
    cv.waitKey(0)
    cv.imwrite(test2+f'test_RANSAC{x}.png',img)
    cv.destroyAllWindows()
img1=cv.imread(test2+'test1.png')
img2=cv.imread(test2+'test2.png')
img1=cv.resize(img1,(0,0),fx=0.4,fy=0.4)
img2=cv.resize(img2,(0,0),fx=0.4,fy=0.4)
h1,w1=img1.shape[:2];h2,w2=img2.shape[:2]
top_size,bottom_size,left_size,right_size=(0,0,abs(w1-w2),0)
img2 = cv2.copyMakeBorder(img2,top_size,bottom_size,left_size,right_size,borderType=cv2.BORDER_REPLICATE)#复制法 replicate就是复制的意思
print(w1)
print(w2)
#检测A,B图像的SIFT关键特征点，并计算描述子
def _sift(img):
    sift=cv.SIFT.create()
    (kps,res)=sift.detectAndCompute(img,None)
    #将结果转换位numpy数组
    #kps=np.float32([kp.pt for kp in kps])
    #返回特征点集，及对应特征描述
    return (kps,res)

MIN=10#统计学结果
starttime=time.time()
#获取图像SIFT关键特征点，并计算特征描述子
kp1,res1=_sift(img1)
kp2,res2=_sift(img2)
#加权处理

#建立字典及flanny匹配器
indexParams = dict(algorithm = 1, trees = 5)
searchParams = dict(checks=50)
flann = cv2.FlannBasedMatcher(indexParams,searchParams)

# 使用 检测来自1，2图的SIFT特征匹配对，K=2

matches=flann.knnMatch(res1,res2,k=2)
good=[]

#过滤特征点
for i,(m,n) in enumerate(matches):
    if(m.distance<0.75*n.distance):
        good.append(m)

#单适应匹配后获得透视变换矩阵，用这个逆矩阵来对第二幅图像进行透视变换，匹配对数大于10，计算透视变换矩阵
if len(good)>MIN:
    src_pts = np.float32([(kp1[m.queryIdx]).pt for m in good]).reshape(-1, 1, 2)
    ano_pts = np.float32([kp2[m.trainIdx].pt for m in good]).reshape(-1, 1, 2)
    M, mask = cv2.findHomography(src_pts, ano_pts, cv2.RANSAC, 5.0)
    warpImg = cv2.warpPerspective(img2, np.linalg.inv(M), (img1.shape[1] + img2.shape[1], img2.shape[0]))
    direct = warpImg.copy()
    direct[0:img1.shape[0], 0:img2.shape[1]] = img1
    simple = time.time()

rows,cols=img1.shape[:2]
print(rows)
print(cols)

imageA=img1.copy()
for col in range(0, cols):
    # 开始重叠的最左端
    if imageA[:, col].any() and warpImg[:, col].any():
        left = col
        print(left)
        break

for col in range(cols - 1, 0, -1):
    # 重叠的最右一列
    if imageA[:, col].any() and warpImg[:, col].any():
        right = col
        print(right)
        break
# 加权处理
res = np.zeros([rows, cols, 3], np.uint8)
for row in range(0, rows):
    for col in range(0, cols):
        if not imageA[row, col].any():  # 如果没有原图，用旋转的填充
            res[row, col] = warpImg[row, col]
        elif not warpImg[row, col].any():
            res[row, col] = imageA[row, col]
        else:
            srcImgLen = float(abs(col - left))
            testImgLen = float(abs(col - right))
            alpha = srcImgLen / (srcImgLen + testImgLen)
            res[row, col] = np.clip(imageA[row, col] * (1 - alpha) + warpImg[row, col] * alpha, 0, 255)

warpImg[0:imageA.shape[0], 0:imageA.shape[1]]=res
cv_show(warpImg,3)
final=time.time()
print(final-starttime)
h,w=warpImg.shape[:2]
img=cv.cvtColor(warpImg,cv.COLOR_BGR2GRAY)
k=0
for j in  range(w-1,-1,-1):
    res=0
    for i in range(0,h) :
        if(img[i][j]<=0 and img[i][j-1]>0):
         res+=1
    if(res>=h-1):
        k=j
        break
warpImg=warpImg[:,:k,:]
cv_show(warpImg,4)
```

