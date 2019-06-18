인사말(HELLO WORLD)
--------
안녕하세요. 이사작전.com의 개발자 플랫폼공작소입니다. 이 프로젝트는 안정적입니다. 만약 당신이 이 프로젝트를 이용한다면, 당신은 말길을 알아먹는 서버(ASR)를 5분 만에 동작시킬 수 있을 것입니다. 물론 이게 왜 동작하는지 이해하기 위해서는 선행학습이 좀 필요하지만요. (ASR, AUTOMATIC SPEECH RECOGNITION)

> 궁금한 것이 있다면 언제든 질문 남겨주세요.

프로젝트 목표(GOAL)
--------
이것은 도커를 이용하여 실시간 + 대용량 전송량을 처리할 수 있는 칼디-스트림-서버(+REST API)를 구축함을 목표로 합니다.

구체적으로 무엇이 가능한지?(PROPOSE)
--------
- 로컬에서 음성파일을 스트림 서버에 전송하면 말길을 알아듣고 해석을 시작합니다.
- jQuery Ajax를 통해서 음성파일의 경로를 칼디-스트림-서버에 전송하면 JSON형식으로 해석한 것을 받을 수 있습니다. (현재 점검 중...)

이 프로젝트는 선행학습이 필요합니담 :)
---------
1. 이 도커파일은 자동적으로 노예(worker)와 주인(master)서버를 구축합니다. 그것이 무엇인지 확인해보세요.
- http://ebooks.iospress.nl/volumearticle/37996
- https://github.com/alumae/kaldi-gstreamer-server

2. https://github.com/kaldi-asr/kaldi 에서 yesno혹은 voxforge 등의 세팅과 데모 구동을 "반드시" 체험하세요.

중요한 내용입니다. 집중하세요
---------
이 프로젝트를 실행하기 위해선 두 가지가 필요합니다.
칼디 모델과 그 칼디 모델을 설명하는 yaml 파일입니다.

아래의 경로에서 예제, 칼디 모델을 다운로드 받고 이것이 무엇인지 감을 잡아보세요.
- https://phon.ioc.ee/~tanela/tedlium_nnet_ms_sp_online.tgz
(tip : 이것은 영어 칼디 모델입니다.) 

아래의 경로에서 yaml 예제를 확인함으로써 그것들이 무엇인지 감을 잡아보세요.
- https://github.com/alumae/kaldi-gstreamer-server/blob/master/sample_worker.yaml
- https://github.com/alumae/kaldi-gstreamer-server/blob/master/estonian_worker.yaml
- https://github.com/alumae/kaldi-gstreamer-server/blob/master/sample_english_nnet2.yaml
- 그리고 당신만의 yaml 파일을 작성하는 방법을 공부해보세요.

모든 준비가 끝났습니다. 
이제 프로젝트를 본격적으로 설치하고 데모를 구동시킬 시간입니다.

너의 환경은? (Your Setting)
--------------
- OS : UBUNTU 18.04 (윈도우 파워쉘이나 vmware는 사용하지 마세요. GPU나 마이크 잡으실 때 고생하십니다.)
- PYTHON : 2.7 (version)

도커를 설치해봅시다.
--------------
Please, refer to https://docs.docker.com/engine/installation/.

이미지를 얻어봅시다.
-------------

* Pull the image from Docker Hub (~ 900MB):

`docker pull jcsilva/docker-kaldi-gstreamer-server`

* Or you may build your own image (requires git):

`docker build -t kaldi-gstreamer-server:1.0 https://github.com/jcsilva/docker-kaldi-gstreamer-server.git`

In the next sections I'll assume you pulled the image from Docker Hub. If you have built your own image, simply change *jcsilva/docker-kaldi-gstreamer-server:latest* by your image name when appropriate.

사용방법
----------
It's possible to use the same docker in two scenarios. You may create the master and worker on the same host machine. Or you can create just a worker and connect it to an already existing master. These two situations are explained below. 

* Instantiate master server and worker server on the same machine:

Assuming that your kaldi models are located at /media/kaldi_models on your host machine, create a container:

```
docker run -it -p 8080:80 -v /media/kaldi_models:/opt/models jcsilva/docker-kaldi-gstreamer-server:latest /bin/bash
```

And, inside the container, start the service:

```
 /opt/start.sh -y /opt/models/nnet2.yaml
```

You will see that 2 .log files (worker.log and master.log) will be created at /opt of your containter. If everything goes ok, you will see some lines indicating that there is a worker available. In this case, you can go back to your host machine (`Ctrl+P and Ctrl+Q` on the container). Your ASR service will be listening on port 8080.

For stopping the servers, you may execute the following command inside your container:
```
 /opt/stop.sh
```

* Instantiate a worker server and connect it to a remote master:

Assuming that your kaldi models are located at /media/kaldi_models on your host machine, create a container:

```
docker run -it -v /media/kaldi_models:/opt/models jcsilva/docker-kaldi-gstreamer-server:latest /bin/bash
```

And, inside the container, start the service:

```
/opt/start.sh -y /opt/models/nnet2.yaml -m master-server.com -p 8888
```

It instantiates a worker on your local host and connects it to a master server located at master-server.com:8888. 

You will see that a worker.log file will be created at /opt of your container. If everything goes ok, you will see some lines indicating that there is a worker available.

For stopping the worker server, you may execute the following command inside your container:
```
 /opt/stop.sh
```

테스트 방법
-------

First of all, please, check if your setup is ok. It can be done using your browser following these steps:

1. Open a websocket client in your browser (e.g: [Simple-WebSocket-Client](https://github.com/hakobera/Simple-WebSocket-Client) or http://www.websocket.org/echo.html).
 
2. Connect to your master server: `ws://MASTER_SERVER/client/ws/status`. If your master is on local host port 8080, you can try: `ws://localhost:8080/client/ws/status`.

3. If your setup is ok, the answer is going to be something like: `RESPONSE: {"num_workers_available": 1, "num_requests_processed": 0}`.

After checking the setup, you should test your speech recognition service. For this, there are several options, and the following list gives some ideas:

1. You can download [this client](https://github.com/alumae/kaldi-gstreamer-server/blob/master/kaldigstserver/client.py) for your host machine and execute it. When the master is on the local host, port 8080 and you have a wav file sampled at 16 kHz located at /home/localhost/audio/, you can type: ```python client.py -u ws://localhost:8080/client/ws/speech -r 32000 /home/localhost/audio/sample16k.wav```

2. You can use [Kõnele](http://kaljurand.github.io/K6nele/) for testing the service. It is an Android app that is freely available for downloading at Google Play. You must configure it to use your ASR service. Below you'll find some screenshots that may help you in this configuration. First, you should click on **Kõnele (fast recognition)**. Then, change the **WebSocket URL**. In my case, I connected to a master server located at ws://192.168.1.10:8080/client/ws/speech. After that, open a **notepad-like** application and change your input method to **Kõnele speech keyboard** and you'll see a **yellow button** instead of your traditional keyboard. Press this button and enjoy!


<img src="img/1.png" alt="Kõnele configuration" width="200px"/>
&nbsp;
<img src="img/2.png" alt="Kõnele configuration" width="200px"/>
&nbsp;
<img src="img/3.png" alt="Kõnele configuration" width="200px"/>
&nbsp;
<img src="img/4.png" alt="Kõnele configuration" width="200px"/>
&nbsp;
<img src="img/5.png" alt="Kõnele configuration" width="200px"/>
&nbsp;
<img src="img/6.png" alt="Kõnele configuration" width="200px"/>

3. A Javascript client is available at http://kaljurand.github.io/dictate.js/. You must configure it to use your ASR service.

실용적인 예제
-----------------

This section describes a tested example. You may repeat all the steps and, in the end, you'll have an english ASR system working on your machine. For this example, I advise you to use a machine with at least 4GB RAM.

On the host machine, we are going to work on the directory /media/kaldi_models. I'll assume you have all permissions necessary to execute the following commands.

1) Download a valid kaldi model:
```
cd /media/kaldi_models
wget https://phon.ioc.ee/~tanela/tedlium_nnet_ms_sp_online.tgz
tar -zxvf tedlium_nnet_ms_sp_online.tgz
```

2) Copy an example yaml file to /media/kaldi_models:
```
wget https://raw.githubusercontent.com/alumae/kaldi-gstreamer-server/master/sample_english_nnet2.yaml -P /media/kaldi_models
```

3) Update file contents:
```
find /media/kaldi_models/ -type f | xargs sed -i 's:test:/opt:g'
sed -i 's:full-post-processor:#full-post-processor:g' /media/kaldi_models/sample_english_nnet2.yaml
```

4) Instantiate master and worker on the same machine:
```
docker run -it -p 8080:80 -v /media/kaldi_models:/opt/models jcsilva/docker-kaldi-gstreamer-server:latest /bin/bash
```

5) Inside the docker container, start the service:
```
/opt/start.sh -y /opt/models/sample_english_nnet2.yaml
```

6) On your host machine, download a client example and test your setup with a given audio:
```
wget https://raw.githubusercontent.com/alumae/kaldi-gstreamer-server/master/kaldigstserver/client.py -P /tmp
wget https://raw.githubusercontent.com/jcsilva/docker-kaldi-gstreamer-server/master/audio/1272-128104-0000.wav -P /tmp
wget https://raw.githubusercontent.com/alumae/kaldi-gstreamer-server/master/test/data/bill_gates-TED.mp3 -P /tmp
python /tmp/client.py -u ws://localhost:8080/client/ws/speech -r 32000 /tmp/1272-128104-0000.wav
python /tmp/client.py -u ws://localhost:8080/client/ws/speech -r 8192 /tmp/bill_gates-TED.mp3
```

OBS: For running the client example, you must install ws4py version 0.3.2. This can be installed using `pip  install --user ws4py==0.3.2`. You may also need simplejson and pyaudio. They may also be installed using pip.

You should get these transcriptions:

* Audio bill_gates-TED.mp3:

and i was a kid the disaster we worry about most was a nuclear war. that's why we had a barrel like this down our basement filled with cans of food and water. when the nuclear attack came we were supposed to go downstairs hunker down and eat out of that barrel. today the grea/opt risk of global catastrophe. doesn't look like this instead it looks like this. if anything kills over ten million people in the next few decades it's most likely to be a highly infectious virus rather than a war. not missiles that microbes now part of the reason for this is that we have invested a huge amount in nuclear deterrence we've actually invested very little in a system to stop an epidemic. we're not ready for the next epidemic.

* Audio 1272-128104-0000.wav:

mr coulter is the apostle of the middle classes and we're glad to welcome his gospel.

참조 (Reference & Credits)
--------
* [kaldi](http://www.kaldi.org)
* [gst-kaldi-nnet2-online](https://github.com/alumae/gst-kaldi-nnet2-online)
* [kaldi-gstreamer-server](https://github.com/alumae/kaldi-gstreamer-server)
* [Kõnele](http://kaljurand.github.io/K6nele/)

프로젝트 원본 (ORIGINAL)
--------
이 프로젝트의 원본은 다음과 같습니다.
- https://github.com/alumae/kaldi-gstreamer-server
- https://github.com/jcsilva/docker-kaldi-gstreamer-server
