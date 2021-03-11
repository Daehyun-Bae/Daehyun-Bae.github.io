---
layout: post
title:  Tensorflow object detection API 사용법(1)
excerpt: Tensorflow object detection API tutorial guide
date:   2018-08-18
categories: [Tensorflow]
comments: false
---

# Object Detection API 사용하기(1)

랩실에서 진행했던 인공지능 교육 실습 자료 정리

Object dection 실습 조교를 맡게 되었다.

실습 여건 상 모델을 처음부터 구현하고 학습시킬 시간이 부족하기 때문에 수강하시는 분들이 object detection model을 실행시켜보기만 하더라도 나중에 유용하게 사용하실 수 있을 것 같아서 Tensorflow에서 제공하는 object detection API 사용방법을 준비하였다. 

실습 준비를 하면서 많은 삽질을 해서, 혹시 Object detection API를 사용하고자 하는 분들에게 조금이나마 도움이 되었으면하는 마음으로 기록

*본 포스트는 [2018년 8월 기준으로 작성된 글](https://blog.naver.com/bdh0727/221341342386)을 이전하며 작성하였습니다. 현재 버전과는 다소 차이가 있을 수 있습니다.*

## 환경 구성

----

실습환경

* OS: Ubuntu 16.04
* CPU: i7-7700K
* GPU: NVIDIA GeFore GTX 1080
* Python package
  * protobuf
  * pillow
  * lxml
  * tensorflow(>1.4)
  * Cython

실습에 사용했던 코드는 공식 제공 코드를 쉽게 실행시킬 수 있도록 약간 변형하였다.

*혹시 코드가 필요하신 분들은 아래 깃허브를 참고하시면 됩니다.*
 [Github-Object_Detection_API_utils](https://github.com/Daehyun-Bae/Object_Detection_API_utils)

*Pre-trained model이나 실습에 사용한 dataset 등은 아래 구글 드라이브에 올려두었습니다.*
[Google drive-Object_Detection_API_dataset](https://drive.google.com/drive/folders/1mwz5Lk-CfPM7q4Yb3X8jThlZzUYsH9by?usp=sharing)



## 0. 준비 과정

----

먼저 tensorflow에서 제공하는 Github repository를 다운받는다.

```
git clone http://github.com/tensorflow/models
```

다운로드 후 `models/research` 경로로 이동한다.

```
cd models/research
```

Object detection API에서는 프로토콜 버퍼를 이용한다.
`protoc` 명령어로 다음 경로에 있는 `*.proto` 파일들을 컴파일 해준다.

```
// From models/research/
protoc object_detection/protos/*.proto --python_out=.
```

그리고 파이썬 환경변수를 설정해준다.

```
export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
```

작은 따옴표( ' )가 아니라 물결표 밑에 있는 악센트 부호( ` )에 주의...

다음 명령어로 제대로 컴파일이 완료되었는지 확인할 수 있다.

```
python3 object_detection/builders/model_builder_test.py
```

![model builder test](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-1\install_chk.png)



### 1. Pre-trained model 가져오기 및 실행

----

구글에서는 이미 다양한 dataset에 대해 학습이 완료된 pre-trained model을 제공하고 있다.

![pre-train model examples](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-1\pre-train-models.png)

[Tensorflow1 object detection API model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf1_detection_zoo.md)

실습에서는 MSCOCO dataset에 대해 학습한 resnet101 기반의 faster-RCNN 모델(faster_rcnn_resnet101_coco)을 사용하였다.

위의 링크에서 해당 모델을 다운 받은 뒤 압축을 풀면 몇가지 파일들이 나온다.

![faster_rcnn_resnet101_coco files](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-1\rcnn-resnet101.png)

실제로 detection 을 수행하기 위해 필요한 파일은 `frozen_inference_graph.pb` 파일 하나뿐이다. `model.ckpt`는 나중에 여기서 fine-tuning을 할 때 사용한다.

**Pre-trained model 실행에 필요한 파일(실습 파일 참고)**

* `frozen_inference_graph.pb`: 학습이 완료된 모델의 그래프를 export한 파일
* `mscoco_label_map.pbtxt`: MSCOCO dataset에 대한 label 정보(label_map/mscoco_label_map.pbtxt)
* `object_detection_run.py`: pre-trained model을 실행시키는 코드 (tensorflow에서 제공하는 예제코드를 조금 변형)

`object_detection_run.py` 에서 실행에 필요한 각종 파일 경로와 값을 설정하고 실행한다.

```python
# Pre-trained graph model PATH
MODEL_NAME = 'faster_rcnn_resnet101'
PATH_TO_CKPT = os.path.join(MODEL_NAME, 'froaen_inference_graph.pb')

# Label information
PATH_TO_LABELS = os.path.join('label_map', 'mscoco_label_map.pbtxt')
NUM_CLASSES = 90

# Testing path
PATH_TO_TEST_IMAGES_DIR = 'test_images/test_coco'
TEST_IMAGE_PATHS = [os.path.join(PATH_TO_TEST_IMAGES_DIR, img) for img in os.listdir(PATH_TO_TEST_IMAGES_DIR)]
TEST_SAVE_PATH = 'test_result/{}'.format(MODEL_NAME)
```

* MODEL_NAME = pre-trained model (frozen_inference_graph.pb)이 저장 된 폴더명
* PATH_TO_CKPT = **frozen_inference_graph.pb** 파일 경로
* PATH_TO_LABELS = Label map 파일 (**mscoco_label_map.pbtxt**) 경로
* NUM_CLASSES = Class 개수 (MSCOCO에서는 90개)
*  PATH_TO_TEST_IMAGES_DIR = Detection test에 사용 될 이미지 Directory
* TEST_IMAGE_PATHS = test image들의 경로
* TEST_SAVE_PATH = test 결과를 저장할 directory

```
python3 object_detection_run.py
```



### 실행 결과

----

inference 코드를 실행하면 지정했던 경로에 테스트 결과 이미지가 저장된다.

![example input](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-1\test-input.png)

![example output](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-1\test-output.png)



> Reference
>
> [Tensorflow Object Detection API Home](https://github.com/tensorflow/models/tree/master/research/object_detection)
>
> [Tensorflow Object Detection API Install Guide](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md)
>
> [조대협님의 블로그](http://bcho.tistory.com/1192)
>
> [솔라리스의 인공지능 연구실](http://solarisailab.com/archives/2422)