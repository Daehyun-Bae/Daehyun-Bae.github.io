---
layout: post
title:  원격 서버의 Docker와 로컬 Pycharm 연동하기
excerpt: 코드 작성은 로컬에서, 실행은 원격 서버의 리소스를 사용하고 싶다면?
date:   2019-03-02
categories: [Development]
tags: [Docker, Pycharm, Environment]
comments: true
---

# 원격 서버의 Docker와 로컬의 Pycharm 연동하기

연구실 후배의 추천으로 최근 Docker(도커)를 활용한 개발 환경에 익숙해지려고 한다.

서버 활용도가 점점 높아지면서 도커를 잘 활용해보려고 한다. 도커를 이용하면 코드를 실행할 환경 설정을 서버마다 일일이 하지 않아도 되고, 각자의 개발 환경을 분리하여 관리할 수 있기 때문에 특히 연구실처럼 다수의 인원이 함께 서버를 사용하는 경우에 환경 세팅에 필요한 시간을 크게 줄일 수 있다.

[(참고) Pycharm과 도커 설치 안내 페이지](http://drmola.com/bbs_free/318738)

그럼에도 불구하고, 로컬에서 코드를 작성하고 서버에서 실행을 할 때 다음과 같은 과정을 거친다.

1. 로컬에서 코드 작성
2. 로컬의 도커 이미지에서 코드 실행 확인
3. 같은 도커 이미지를 (리모트)서버에서 Pull
4. 작성한 코드를 서버에 전송
5. 서버에서 코드 실행

도커 이미지만 옮기면 환경 세팅이 끝나기 때문에 편리하지만  여전히 서버에서 도커 이미지를 가져오거나 코드를 수정할 때마다 서버로 옮기는 과정, 코드 실행을 위해 터미널로 서버에 접속하는 과정이 은근 번거로웠다.

개발은 로컬에서 편리하게 하고 코드 실행은 서버의 리소스를 이용할 수 있다면 굉장히 편할 것이다.

몇 번의 삽질 끝에 위의 과정을 더 간단하게 단축시키는 방법을 알아내었다.

1. 로컬(Pycharm)에서 코드 작성
2. 로컬에서 코드 실행 &rarr; 실제로는 원격 서버의 도커 이미지에서 실행



### 사전 준비

---

글 작성 시점 기준 실행 환경은 다음과 같다.

로컬

* OS: Ubuntu 18.04 LTS
* Pycharm: Professional 2018.3

서버(remote)

* OS: Ubuntu 16.04 LTS
* Docker: 18.09.1



### Remote Docker 세팅

---

서버에 있는 도커 이미지를 가져 올 수 있게 세팅하는 과정이다.

Pycharm에서 `Ctrl+Alt+S` 또는 `File > Settings`로 환경설정에 들어간 뒤 `Docker` 항목을 선택한다. 아래 그림과 같이 새로운 도커 연결 설정을 만들어준다.

![Docker setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\step1.png)

* `Name`: 환경설정 이름(서버 ip 주소 등 구분 가능한걸로)
* `Connect to Docker daemon with`: `TCP socket` 선택 후 `API URL`에는 `tcp://xxx.xxx.xxx.xxx:2375` 형식으로 원격 서버의 ip 주소를 입력한다. (2375는 도커 데몬의 기본 포트 번호)

입력을 완료하고 나면 자동으로 연결 시도 후 아래에 Connection succesful이 뜨는 것을 확인할 수 있다.



### Deployment 설정

---

로컬에서 작성한 코드가 서버에서 자동으로 동기화를 시켜주는 역할을 한다.

> Pycharm professional 버전에서만 된다고 하는데 Student 계정을 이용하면 무료로 education 버전에서도 사용할 수 있다.

마찬가지로 환경설정 메뉴(Settings)에서 `Deployment` 항목의 + 를 눌러서 새로운 설정을 만들어준다.

![Docker setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\dev1.png)



`Connection` 탭에서 연결하려는 서버 정보를 입력한다.

* `Type`: SFTP
* `Host, User name, Password`: 각각 서버 ip, 계정 이름, 비밀번호 등을 입력하면 된다.

입력 완료 후 `Test connection` 버튼을 눌러 연결 확인이 가능하다.

연결이 되었으면 옆의 `Mapping` 탭에서 서버의 어느 디렉토리와 연결할 것인지를 설정해 준다.

![Docker setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\dev2.png)

* `Local path`: 서버에 업로드하고 싶은 **로컬 프로젝트 경로**
* `Deployment path`: 업로드한 프로젝트를 저장할 **서버 경로**

도커를 사용하지 않는 환경이라도 로컬과 서버의 코드를 동기화 시키는 기능이라 유용하게 사용할 수 있을 것 같다.

> `Tools > Deployment > Automatic Upload (Always)` 를 설정해 놓으면 코드에 변경사항이 있을 때마다 자동으로 동기화해 준다.



### Interpreter 설정

---

프로젝트의 인터프리터를 설정하는 과정

앞의 **Remote Docker 세팅**에서 지정한 도커 이미지에 있는 인터프리터를 가져오는 과정이다.

![Interpreter setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\intpre1.png)

`Settings > Project Interpreter` 항목에서 오른쪽 위의 톱니바퀴를 눌러 `Add`를 선택한다.

인터프리터 추가 화면에서 Docker를 선택하고 `Server`에는 앞서 설정한 도커 서버를 고른다. `Image name`에서 서버에 있는 도커 이미지 중 원하는 도커 이미지를 선택한다.

이 때 추가로 `Path mapping` 설정을 해주어야하는데 이 부분이 헷갈려서 한참 헤맸었다. 옆의 폴더 모양의 아이콘을 눌러 *로컬 프로젝트 경로*와 *도커 컨테이너 내부의 프로젝트 경로*를 매핑해 준다.

![Interpreter setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\intpre2.png)



### 코드 실행 구성 설정(Volume Binding)

---

테스트를 위해 적당한 테스트 코드를 작성한 뒤, `Run > Edit Configurations...`를 눌러 설정 창을 연다.

오른쪽 상단에서도 선택할 수 있다.

![Interpreter setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\config1.png)

**+** 기호를 눌러 새로운 설정을 만들어준다.

여기가 가장 헷갈렸는데 로컬, 서버, 도커 컨테이너 내부의 path가 모두 총집합하는 곳이다.

![Interpreter setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\config2.png)

* `Script path`: 실행하려는 코드의 경로 (로컬)
* `Execution > Docker container settings`: -v (volume bining) 옵션을 통해 서버에 있는 파일 시스템을 도커 컨테이너와 연결할 수 있다.

폴더 모양 아이콘을 누르면 아래와 같은 창이 뜨는데,

`Container path`: 도커 컨테이너 내에서의 프로젝트 경로 (`/project` 등)

`Host path`: 서버에서 프로젝트가 저장되어 있는 경로, 앞서 Deployment에서 설정한 로컬 프로젝트와 동기화하는 경로를 입력해주면 된다.

만약 프로젝트 폴더 외에 서버의 다른 경로에 있는 폴더와 연동하려면 같은 방법으로 추가할 수 있다. (예를 들어 서버에 저장되어있는 dataset 폴더)

![Interpreter setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\config3.png)



### 실행 결과

---

간단하게 GPU 상태 정보를 출력하는 코드를 로컬에서 작성해서 서버의 도커 컨테이너에서 실행해 보았다.

```python
# sample_code.py
import os

print(os.system('nvidia-smi'))
```

로컬(Pycharm)에서 실행하면 원격 서버에서 미리 설정해둔 도커 이미지를 통해 컨테이너를 생성하고, 컨테이너 내부의 코드 실행 결과를 다시 로컬에서 확인할 수 있다.

![Interpreter setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\test.png)





### Outro

---

로컬 / 서버 / 도커 컨테이너 세 곳의 경로를 같이 신경 써야하기 때문에 예제 코드로 실습해 보고도 원래 코드에 적용하려니 또 헤맸었다.

대략적인 관계도를 그림으로 그려봤는데 다른 분들에게도 도움이 되었으면 좋겠다.

`Interpreter`: Local  path - Docker container path

`Deployment`: Local path - Remote server path

`Configuration`: Remote server path - Docker container path

![Interpreter setting](https://daehyun-bae.github.io\img\post\210317-pycharm-docker\summary.png)





#### Reference

> [https://youtrack.jetbrains.com/issue/PY-33489](https://youtrack.jetbrains.com/issue/PY-33489): 전반적인 설명 순서를 참고한 글
>
> [https://goodtogreate.tistory.com/entry/Pycharm으로-TensorFlow-원격-빌드하기](https://goodtogreate.tistory.com/entry/Pycharm으로-TensorFlow-원격-빌드하기): 원격 실행과 Deployment 사용 관련 정보를 검색하다가 발견한 글