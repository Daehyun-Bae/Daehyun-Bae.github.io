```
layout: post
title:  Tensorflow object detection API 사용법(2)
excerpt: Tensorflow object detection API tutorial guide 2 - fine-tuning
date:   2019-05-15
categories: [Tensorflow]
comments: true
```

# Object Detection API 사용하기 (2): Fine-tunig

거의 1년 가까이 지나서야 두번째 세션에 대한 내용을 정리하였다.

올해도 인공지능 실습 교육 조교를 맡아서 새로 코드를 정리하며 미뤄두었던 정리 시작

*본 포스트는 [2019년 5월 기준으로 작성된 글](https://blog.naver.com/bdh0727/221537759295)을 이전하며 작성하였습니다. 현재 버전과는 다소 차이가 있을 수 있습니다.*

---

[지난 글](https://daehyun-bae.github.io/articles/2018-08/Tensorflow-Object-Detection-API-%EC%82%AC%EC%9A%A9%EB%B2%95-(1))에서는 Object detection API를 사용하기 위한 환경을 셋업하고 미리 학습된 모델을 실행시키는 방법에 대해 다루었다.

*혹시 코드가 필요하신 분들은 아래 깃허브를 참고하시면 됩니다.*

 [Github-Object_Detection_API_utils](https://github.com/Daehyun-Bae/Object_Detection_API_utils)

*Pre-trained model이나 실습에 사용한 dataset 등은 아래 구글 드라이브에 올려두었습니다.*

[Google drive-Object_Detection_API_dataset](https://drive.google.com/drive/folders/1mwz5Lk-CfPM7q4Yb3X8jThlZzUYsH9by?usp=sharing)



### 0. Fine-tuning 실습

지난번 실습 때 사용했던 모델은 MSCOCO라는 데이터셋에 미리 학습된 모델이다.

새로운 데이터에서 객체를 검출하려면 어떻게 해야할까?

MSCOCO 데이터셋이 워낙 많은 클래스를 갖고 있기 때문에 웬만한 물체는 찾을 수 있지만 한계가 있을 것이다.

이 때 사용자가 원하는 데이터셋에 맞게 추가 학습을 진행하는 것이 fine-tuning이다.

![intro-image](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-2\intro.jpg)*이 고양이가 무슨 고양인지 알고싶다!*