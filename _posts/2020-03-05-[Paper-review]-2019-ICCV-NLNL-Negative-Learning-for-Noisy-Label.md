# [2019 ICCV] NLNL: Negative Learning for Noisy Label

## 0.Abstract

* Image classification에서 기존의 supervised learning은 이미지(image)에 할당된 레이블(class, label)에 속하도록 학습
  * *이 이미지는 이 레이블에 속해야 한다(input image belongs to this label)* 라고 학습
  * 여기서는 **Positive Learning (PL)**이라고 정의
* **하지만** 데이터셋에 노이즈가 껴있으면 성능 저하 발생
* **Noise data로부터 오는 영향을 줄이는 것이 목표**
* 여기서 **Negative Learning (NL)**이라는 학습 방법 소개
  * *이 이미지는 이 레이블에 속하지 않는다(input image does not belong to this complementary label)* 라고 학습
  * 모델에 correct information을 최대한 많이 전달하는 것이 목표
  * Noisy data가 있을 때, complementary label이 true label이 될 확률(incorrect information)이 매우 낮음
  * 하지만 PL로 학습하면 확실히 incorrect information이 전달 되기 때문에 모델 성능 저하에 영향을 미침
* NL로 학습 후 noisy data filtering을 통해 학습 성능을 더욱 향상 시킴
  * Selective NL(SelNL)
  * Selective PL(SelPL)
  * Selective NL & PL(SelNLPL)
  * SelNLPL + Semi-supervised Learning



## 1. Method

### 1.1. Negative Learning(NL)

* Complementary label generation
  * 이미지와 레이블이 $$(x, y)$$와 같이 주어졌을 때, $$y$$를 자기자신을 제외한 임의의 다른 레이블(Complementary label) $$\overline{y}$$로 다시 할당 해주는 작업
* 기존 PL에서의 CE(Cross Entropy) Loss가 다음과 같다면,

![Positive Learning Loss](https://daehyun-bae.github.io/img/post/200305_nlnl_0.PNG)

* NL에서의 CE Loss는 다음과 같다.

![Negative Learning Loss](https://daehyun-bae.github.io/img/post/200305_nlnl_1.PNG)

* NL에서는 $$x$$의 레이블이 $$\overline{y}$$ 가 아니라고 주입식 학습(?)을 시켜야하기 때문에 $$(1-p_k)$$의 형태로 바뀜 ($$p_k$$가 낮아져야 전체 Loss 값이 낮아짐)

* Noise 30%의 데이터셋에서 학습한 결과 PL을 썼을 때 NL을 썼을 때보다 training loss가 더 낮게 나타났지만, 결국 overfitting이 발생하여 test loss는 NL을 썻을 때보다 높게 나타남



### 1.2. Selective NL(SelNL)

* NL로 학습한 후에 training data에서 confidence > $$\frac1c$$ 인 데이터들만 모아서 다시 NL 학습
* Noisy data 를 어느정도 걸러내는 효과



### 1.3. Selective PL(SelPL)

* Clean data만 있다고하면 사실 PL이 NL보다 빨리 수렴하고 정확도도 더 높다.
* NL->SelNL 학습 후에 다시 confidence threshold($$\gamma$$ = 0.5)보다 높은 데이터만 추려내서 PL 학습 진행



### 1.4. Selective NL and PL(SelNLPL)

* NL -> SelNL -> SelPL 학습



### 1.5. Semi-supervised Learning

* SelNLPL로 학습된 모델을 사용하여 noisy data를 필터링 한다.
* Clean data로 다시 CNN 모델을 학습한다.
* 학습된 모델에 noisy data를 넣어 나오는 값을 해당 데이터의 pseudo label로 사용한다.
* Clean data + Pseudo label noisy data를 사용하여 다시 CNN 모델 학습
* SelNLPL로 학습된 모델을 사용하면 꽤 정확하게 noise data를 걸러낼 수 있다.



## 2. Experiments

### Experimental settings

#### Dataset

* CIFAR-10, CIFAR-100, Fashion MNIST, MNIST

#### Noise

* **Symmetric noise**
  * 완전 랜덤하게 label을 바꿔치는 것
  * Ground truth를 포함하느냐 (*symm-inc*) 포함하지 않느냐(*symm-exc*)의 차이가 있음
* **Asymmetric noise**
  * 실제 에러(real error)와 유사하게 label를 바꾸는 방법
  * (**CIFAR-10**) Truck -> Automobile, Bird -> Plane, Deer -> Horse, ...
  * (**Fashion MNIST**) Boot -> Sneaker, Coat -> Dress, Pullover -> Shirt, ...

#### Etc.

* SGD with momentum 0.9
* Weight decay $$10^{-4}$$
* Batch size 128
* Learning rate 0.02 -> 0.02 -> 0.1 (NL->SelNL->SelPL)
* 720 training epoch each stage

