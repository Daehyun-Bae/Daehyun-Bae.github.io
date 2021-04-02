---
layout: post
title:  Tensorflow object detection API 사용법(2)
excerpt: Tensorflow object detection API tutorial guide 2 - fine-tuning
date:   2019-05-15
categories: [Development]
tags: [Tensorflow]
comments: true
---

# Object Detection API 사용하기 (2): Fine-tunig

거의 1년 가까이 지나서야 두번째 세션에 대한 내용을 정리하였다.

올해도 인공지능 실습 교육 조교를 맡아서 새로 코드를 정리하며 미뤄두었던 정리 시작

[지난 글](https://daehyun-bae.github.io/articles/2018-08/Tensorflow-Object-Detection-API-%EC%82%AC%EC%9A%A9%EB%B2%95-(1))에서는 Object detection API를 사용하기 위한 환경을 셋업하고 미리 학습된 모델을 실행시키는 방법에 대해 다루었다.

> 본 포스트는 [2019년 5월 기준으로 작성된 글](https://blog.naver.com/bdh0727/221537759295)을 이전하며 작성하였습니다. 
>
> 현재 버전과는 다소 차이가 있을 수 있습니다.
>
> 혹시 코드가 필요하신 분들은 아래 깃허브를 참고하시면 됩니다.
>
>  [Github-Object_Detection_API_utils](https://github.com/Daehyun-Bae/Object_Detection_API_utils)
>
> Pre-trained model이나 실습에 사용한 dataset 등은 아래 구글 드라이브에 올려두었습니다.
>
> [Google drive-Object_Detection_API_dataset](https://drive.google.com/drive/folders/1mwz5Lk-CfPM7q4Yb3X8jThlZzUYsH9by?usp=sharing)

### 0. Fine-tuning

---

지난번 실습 때 사용했던 모델은 MSCOCO라는 데이터셋에 미리 학습된 모델이다.

새로운 데이터에서 객체를 검출하려면 어떻게 해야할까?

MSCOCO 데이터셋이 워낙 많은 클래스를 갖고 있기 때문에 웬만한 물체는 찾을 수 있지만 한계가 있을 것이다.

이 때 사용자가 원하는 데이터셋에 맞게 추가 학습을 진행하는 것이 fine-tuning이다.

![intro-image](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-2\intro.jpg)
*이 고양이가 무슨 고양인지 알고싶다!*



### 1. 학습 준비

---

이번 실습에는 Oxford-IIIT Pets라는 데이터셋을 이용하여 기존 모델을 fine-tuning하는 방법을 다뤘다.

해당 데이터셋은 총 37종류의 강아지와 고양이 사진이 종류별로 약 200장씩 구성되어 있는 데이터셋이다.

실습에 사용되는 파일은 다음과 같다.

![intro-image](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-2\train-file.png)

`train.py` 코드를 통해 추가 학습을 진행 할 수 있다. 마찬가지로 공식 페이지에서 제공하는 코드를 약간 변형하여 사용하였다.

코드에서 수정해야할 부분은 다음과 같다.

```python
# train.py
TRAIN_DIR = 'train_res_pet' # path to save training result
PIPELINE_CONFIG_PATH = 'piipeline_pet.config' # path to training configuration file
```

* TRAIN_DIR: 학습 결과를 저장할 경로
* PIPELINE_CONFIG_PATH: 학습에 필요한 정보가 담겨있는 .config 파일 경로

학습에 필요한 정보는 `pipeline_pet.config`에 정의 되어있다.

해당 파일을 열어 학습에 필요한 파라미터를 수정한다.

```json
model {
  faster_rcnn {
    num_classes: 37
    image_resizer {
      keep_aspect_ratio_resizer {
        min_dimension: 600
        max_dimension: 1024
      }
    }
    feature_extractor {
      type: 'faster_rcnn_resnet101'
      first_stage_features_stride: 16
    }
...
```

model{...} 은 학습을 수행할 모델에 대한 정보가 정의되어 있다.

크게 수정할 부분은 없다.

```json
train_config: {
  batch_size: 1
  optimizer {
    momentum_optimizer: {
      learning_rate: {
        manual_step_learning_rate {
          initial_learning_rate: 0.0003
          schedule {
            step: 7000
            learning_rate: .00003
          }
        }
      }
      momentum_optimizer_value: 0.9
    }
    use_moving_average: false
  }
  gradient_clipping_by_norm: 10.0
  fine_tune_checkpoint: "faster_rcnn_resnet101/model.ckpt"
  from_detection_checkpoint: true
  num_steps: 20000
  data_augmentation_options {
    random_horizontal_flip {
    }
  }
}
```

다음은 학습과 관련된 내용을 정의하는 부분이다.

batch size를 비롯해 optimizer나 learning rate scheduling도 가능하다.

> batch size를 1보다 크게 사용할 때는 위의 모델 정의 부분에서  `keep_aspect_ratio_resizer` 대신에 `fixed_shape_resizer` 를 사용해야한다.
>
> ```json
> model {
>   faster_rcnn {
>     num_classes: 37
>     image_resizer {
>       fixed_shape_resizer {
>         height: 600
>         width: 1024
>       }
>     }
> ...
> ```

learning rate schedule은 다음과 같이 설정할 수 있다.

```json
initial_learning_rate: 0.0003
          schedule {
            step: 7000
            learning_rate: .00003
          }
```

위의 예시는 시작 learning rate를 0.0003으로 하고, step count가 7000이 되면 0.00003으로 바꾸겠다는 의미이다.

마지막으로 미리 학습된 모델의 weight 파일 경로와 학습 횟수를 지정해 준다.

```json
fine_tune_checkpoint: "faster_rcnn_resnet101/model.ckpt"
from_detection_checkpoint: true
num_steps: 20000
```

* `fine_tune_checkpoint`: pre-trained model의 체크포인트 파일 경로
* `num_step`: 학습 횟수

fine tuning이라고는 하지만 제대로 성능을 낼 수 있을 정도로 학습시켜보니 약 70,000 회는 되어야했다. *(약 5,000 회에 30분 정도니까 약 7시간...)*

실습 시간에 처음부터 7만번이나 학습하면 그 날 수업은 그대로 끝이다. 실제로 실습을 진행할 때는 미리 어느 정도 학습을 시켜놓은 모델을 제공하여, 추가 학습에 걸리는 시간을 단축시켰다.

50,000 번 학습한 모델은 pet_dataset_pre_trained 폴더에 저장해 두었다. (google drive에서 다운받을 수 있습니다.)

마지막으로 학습 데이터에 대한 경로를 지정해 준다.

```json
train_input_reader: {
  tf_record_input_reader {
    input_path: "pet_dataset_pre_trained/pet_train.record"
  }
  label_map_path: "label_map/pet_label_map.pbtxt"
}
```

* `input_path`: 학습 데이터 경로
* `label_map_path`: 학습할 데이터의 label 목록 파일의 경로

TFrecord라는 데이터 타입을 사용한다. 이 TFrecord를 생성하는 실습은 다음 포스팅에서 정리해야겠다. 

학습 준비를 마치고 `train.py`를 통해 학습을 진행한다.

```bash
python3 train.py
```

학습이 완료 되면 미리 설정해두었던 경로에 다음과 같은 파일들이 생긴다.

![intro-image](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-2\after-training.png)
*학습 중간중간에 저장한 checkpoint 파일들과 configuration 파일, 학습 로그를 기록한 event. 파일 등이 보인다.*



### 2. Export Inference Graph

---

학습이 끝난 후 실제로 모델을 실행시켜 학습이 잘 되었는지 확인을 해보자.

이를 위해서는 `inference graph`라는 것을 추출하는 과정이 필요하다.

graph로 표현되어 있는 tensorflow model에 학습에 사용했던 checkpoint 파일의 weight를 덮어씌워 고정시키는 작업이라고 생각하면 된다.

`export_inference_graph.py`의 코드를 다음과 같이 수정하고 실행한다.

```python
# export_inference_graph.py
PIPELINE_CONFIG_PATH = 'pipeline_pet.config'
OUTPUT_DIR = 'export_pet'
CKPT_PREFIX = 'pet_dataset_pre_trained/model.ckpt-50000'
```

* `PIPELINE_CONFIG_PATH`: 학습에 사용했던 configuration 파일 경로
* `OUTPUT_DIR`: Inference graph를 저장할 경로
* `CKPT_PREFIX`: 학습된 모델의 checkpoint 파일 경로

실행 후에는 다음과 같은 파일이 생성된다.

![intro-image](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-2\inference-graph.png)



### 3. 학습 결과

---

이제 새로 생성된 `frozen_inference_graph.pb`를 이용하여 다시 테스트 이미지에 적용해보자.

앞서 첫번째 실습에서 사용했던 `object_detection_run.py`를 사용한다.

이 때 mscoco가 아닌 pet dataset에 대해 학습하였기 때문에 label map도 바꿔 주어야한다.

```python
# object_detection_run.py
# Pre trained graph model PATH
MODEL_NAME = 'export_pet'
PATH_TO_CKPT = os.path.join(MODEL_NAME, 'frozen_inference_graph.pb')
# Path to label map
PATH_TO_LABELS = os.path.join('label_map', 'pet_label_map.pbtxt')
NUM_CLASSES = 37

PATH_TO_TEST_IMAGES_DIR = 'test_images/test_pet'
TEST_IMAGE_PATHS = [os.path.join(PATH_TO_TEST_IMAGES_DIR, img) for img in os.listdir(PATH_TO_TEST_IMAGES_DIR)]
TEST_SAVE_PATH = 'test_result/{}'.format(MODEL_NAME)
```

실행결과는 다음과 같다.

![intro-image](https://daehyun-bae.github.io\img\post\210311-tf-obj-det-api-2\inference-result.jpg)
*아까 전의 고양이는 뱅갈 고양이였다!*

---

#### Reference

>[Oxford-IIIT Pets dataset](http://www.robots.ox.ac.uk/~vgg/data/pets/)
>
>[Tensorflow Object Detection API Home](https://github.com/tensorflow/models/tree/master/research/object_detection)
>
>[Tensorflow Object Detection API Install Guide](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md)
>
>[조대협님의 블로그](http://bcho.tistory.com/1192)
>
>[솔라리스의 인공지능 연구실](http://solarisailab.com/archives/2422)
>
>[원본 포스팅](https://blog.naver.com/bdh0727/221537759295)

