---
layout: post
title: 红外图像增强/彩色化笔记
category: 深度学习
tags: 深度学习
keywords: infrared image, enhancement, colorization
---

## 1610_TEN（IROS2016）: 红外图像超分辨率_from KAIST
《Thermal Image Enhancement using Convolutional Neural Network》 Yukyung Choi, Namil Kim, Soonmin Hwang and In So kweon  
项目地址: [https://sites.google.com/site/ykchoicv/ten](https://sites.google.com/site/ykchoicv/ten)  
pytorch复现代码: [https://github.com/ninadakolekar/ten](https://github.com/ninadakolekar/ten)

![TEN](http://markdown-ydao.oss-cn-beijing.aliyuncs.com/19-1-10/98876457.jpg)
- 用4层无下采样的卷积层就完成了图像超分辨率
- 为了降低计算量，取36x36的patch进行训练
- 发现用RGB图预训练再直接应用到红外图像上的效果最好，用MWIR预训练的甚至还不如Bi-cubic的好   
```python
class ten(nn.Module):
    def __init__(self):
        super(ten, self).__init__()
        self.model = nn.Sequential(
            nn.Conv2d(1,64,7,stride=1,padding=3),
            nn.ReLU(True),
            
            nn.Conv2d(64,32,5,stride=1,padding=2),
            nn.ReLU(True),
            
            nn.Conv2d(32,32,3,stride=1,padding=1),
            nn.ReLU(True),
            
            nn.Conv2d(32,1,3,stride=1,padding=1)
        )

    def forward(self,x):
        x = self.model(x)
        return x
```

## 1709_TIECNN（IEEE）：韩国延世大学基于CNN的热红外图像增强
《Brightness-Based Convolutional Neural Network for Thermal Image Enhancement》 KYUNGJAE LEE, JUNHYEOP LEE, JOOSUNG LEE, SANGWON HWANG, AND SANGYOUN LEE  
项目地址:  [https://sites.google.com/view/kjaelee/tiecnn](https://sites.google.com/view/kjaelee/tiecnn) (有caffe版的模型训练结果)   
- 通过残差学习，只训练残差，学习HQ和LQ图像之间的高频信息
- 基于RGB图像的brightness domain进行预训练，对于thermal image彩色化是最有效，而非gray domain
    - lightness (L in HSL)
    - intensity (I in HSI)
    - brightness (V in HSV)   
![gray](http://markdown-ydao.oss-cn-beijing.aliyuncs.com/19-1-15/32042216.jpg)
![others](http://markdown-ydao.oss-cn-beijing.aliyuncs.com/19-1-15/80223059.jpg)

![TIECNN](http://markdown-ydao.oss-cn-beijing.aliyuncs.com/19-1-15/38479648.jpg)

![structure](http://markdown-ydao.oss-cn-beijing.aliyuncs.com/19-1-15/86644372.jpg)

## 1806_IR2VI (cvpr18)：将红外图像映射成可见光图像的视觉效果   
《Enhanced Night Environmental Perception by Unsupervised Thermal Image Translation》 Shuo Liu, Vijay John, Erik Blasch, Zheng Liu, and Ying Huang   

![IR2VI](http://markdown-ydao.oss-cn-beijing.aliyuncs.com/19-1-10/71872054.jpg)
- 一个生成器：The **generator** translates an IR image to a synthetic VI image that looks similar to the real VI image   
- 两个判别器：   
  -  the **global discriminator** distinguishes translated VI images from real ones
  -  the **ROI discriminator** aims to distinguish the ROIs between translated VI image and real ones
- 在CycleGAN的基础上进行的改进
- 加入了ROI focal loss来避免错误的映射（如：原图没有森林的地方生成了森林）

**！！！！用的数据集打不开啊打不开！！！！！** [https://www.sensiac.org/external/products/list_databases](https://www.sensiac.org/external/products/list_databases)

## 1811_IE-CGAN（Neurocomputing）：南理工的单张红外图像对比度增强ConditionalGAN
《Single infrared image enhancement using a deep convolutional neural network》 Xiaodong Kuang, Xiubao Sui, Yuan Liu, Qian Chen, Guohua Gu  
github:  [https://github.com/Kuangxd/IE-CGAN](https://github.com/Kuangxd/IE-CGAN) (只有torch版本的测试模型，没有模型结构和训练方法)

![IE-CGAN](http://markdown-ydao.oss-cn-beijing.aliyuncs.com/19-1-10/9567948.jpg)   

 - 目标：
    - 提升对比度
    - 增强细节
    - 抑制背景噪声
    - 能够应用到任意分辨率的图片上进行增强   

- 数据集：   
训练使用可见光的灰度图作为高对比度的ground truth图像，使用tf中的random-contrast函数（pytorch里也有类似的torchvision.transforms.functional.adjust_contrast函数）随机降低灰度图的对比度，来获得低对比度的输入图像

- 评价方法：
    - Peak signal-to-noise ratio (PSNR)、noise quality measure (NQM)、structure similarity index (SSIM)、and multiscale structure similarity index (MSSIM) 使用了一个MATLAB库：https://github.com/sattarab/image-quality-tools
    - entropy (E)
    - image enhancement factor (IEF) ：一个鉴定去噪能力的评价指标 https://ww2.mathworks.cn/matlabcentral/fileexchange/41332-ief-image-enhancement-factor?s_tid=prof_contriblnk


网络结构：   
![IE-CGAN_stucture](http://markdown-ydao.oss-cn-beijing.aliyuncs.com/19-1-10/35003045.jpg)
