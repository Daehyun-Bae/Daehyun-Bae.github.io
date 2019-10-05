# [2018 CVPR] Adversarial Complementary Learning for Weakly Supervised Object Localization (Self-erasing)

## 0. Abstract

- ACoL(Adversarial Complementary Learning)이라는 것을 제안
- Weak supervision으로 semantically interesting한 영역을 localizing하는 방법
- Two classifier
  - (1) Localize discriminative object regions
  - (1)을 통해 discriminative region 일부를 지운다.
  - (2) Discover new object regions
  - 각자 localizing을 거친 후 통합하여 최종 결과물 산출
- Two advantages
  - End2end training
  - Dynamically erasing --> discover complementary object regions effectively
- ILSVRC dataset top-1 localization error rate: 45.14% (SOTA)



## 1. Introduction

Weakly Supervised Object Localization (WSOL):
Image + Image-level label --> Object location 

No bbox

CAM과 같은 방식의 solution의 issue

* Relying on category-wise discriminative features (Most discriminative parts)
* Fail to localizing densely

#### Process

Input image -> (Classifier A) leverage discriminative regions -> Erasing the discriminative regions -> (Classifier B)find new complementary object regions
-> Integrate them 



## 3. Method

### 3.1. Revisiting CAM

CAM: two step processing to localize object region

Ours: select feature maps of last conv-layer directly

----

S: output feature maps from FCN

Add 1$$\times$$1 Conv layer and fed into GAP layer followed by a Softmax layer

Number of channels: K --> C (No. of classes)

Mathematically same with CAM to calculate $$y_c$$ 

Also similar localization map quality

**(Pros) OUR method can generate localization maps in the forward pass**

### 3.2. ACoL (Adversarial Complementary Learning)

Deep classification network
--> focus on unique pattern of a specific category
--> the localization map highlights a __small__ region

Identify the discriminative regions by conducting a threshold

**Training procedure**

​	Update feature map **S**

​	Extract localization map $$M^A$$

​	Discover the discriminative region **R**

​	Obtain erased feature map $$\tilde{S}$$

​	Extract localization map $$M^B$$

​	Obtain fused map $$\tilde{M}^{fuse}$$

​	Update learning parameters

Use erasing threshold $$\delta \in \{0.5, 0.9\}$$ --> 0.6 is best score

## 4. Experiments

### Experiment setup

Training dataset: ImageNet (ILSVRC 2016), CUB-200-2011

Backbone network: VGG16 / GoogLeNet
	VGG16: Remove the layers after conv5-3
	GoogleNet: Remove the layers after last inception block
	Add 2 convolution layers (3$$\times$$3, 1$$\times$$1)

Use Global Average Pooling layer

Pre-trained on ImageNet

256$$\times$$256 --> random crop (224$$\times$$224)

10 crop technique is used for classification evaluation

### Results

#### Classification

* Comparable with Vanilla Backbone Network
* Outperform with Backbone Network + GAP



#### Localization

* GoogLeNet: 56.40(CAM) --> 53.28 (Top-1)
* VGG: 57.20(CAM) --> 54.17 (Top-1)

* CAM based approach: Find small part of object

  ![Compare with CAM](https://Daehyun-Bae.github.io/img/_post/191005_acol_0.png)

#### Ablatio study

* 0.5 ~ 0.9 --> 0.6 (best performance)
* Classifier B successfully works with Classifier A
* Well designed thresholding method can improve the performance

##### Third classifier C

​	No significant improvement both classification and localization



##### Influence classification results

​	Localize using ground-truth label (???)

​	Achieve top-1 error 37.04 on ILSVRC validation set

