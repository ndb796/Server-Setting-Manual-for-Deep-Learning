## 사전 준비물
* 나만의 서버용 워크스테이션
* Ubuntu 18.04 OS
* 인터넷 연결 with public IP address
* 외장 그래픽 카드 such as GTX, Tesla, etc

## Jupyter Notebook 이용 방법
* Ubuntu 18.04 OS에는 기본적으로 python 3이 설치되어 있음
* pip 및 jupyter 설치
```
sudo apt-get update
sudo apt-get install python3-pip
sudo pip3 install notebook
```
* jupyter 접속 비밀번호 설정
```
python3
>> from notebook.auth import passwd
>> passwd()
# 비밀번호 설정한 뒤에 SHA1 값 기록하기
```
* jupyter 환경 설정 파일 만들기
```
jupyter notebook --generate-config
sudo vi /home/{사용자명}/.jupyter/jupyter_notebook_config.py
```
* jupyter 환경 설정하기
```
c = get_config()
c.NotebookApp.password = u'sha1:{해시 값}'
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.notebook_dir = '/'
```
* jupyter 실행해보기
```
sudo jupyter-notebook --allow-root
```
## Jupyter Notebook에 HTTPS 적용하기
* SSL 키 생성하기
```
cd /home/{사용자명}
mkdir ssl
cd ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout "cert.key" -out "cert.pem" -batch
```
* jupyter 환경 설정하기
```
sudo vi /home/{사용자명}/.jupyter/jupyter_notebook_config.py
# 다음의 내용 입력하기
c.NotebookApp.certfile = u'/home/{사용자명}/ssl/cert.pem'
c.NotebookApp.keyfile = u'/home/{사용자명}/ssl/cert.key'
```
* (https://{Host}:8888) 같은 형태로 주피터 노트북에 접속
## Jupyter Notebook을 시스템 서비스로 등록하기
```
# 실행 중인 Jupyter 서버 종료하기
# jupyter-notebook 실행 파일 경로 찾기
which jupyter-notebook
# jupyter.service 작성하기
sudo vi /etc/systemd/system/jupyter.service
# Jupyter Notebook 서비스 작성하기
[Unit]
Description=Jupyter Notebook Server

[Service]
Type=simple
User=root
ExecStart=/usr/bin/sudo /usr/local/bin/jupyter-notebook --allow-root --config=/home/{사용자명}/.jupyter/jupyter_notebook_config.py

[Install]
WantedBy=multi-user.target
# Jupyter 서비스 구동시키기
sudo systemctl daemon-reload
sudo systemctl enable jupyter
sudo systemctl start jupyter
# 서비스 상태에서 오류가 발생하는 경우 cat /var/log/syslog로 로그 확인하기
# Jupyter 서비스 상태 확인하기
sudo systemctl status jupyter
# 오류 발생시 jupyter_notebook_config.py에서 IP 주소 확인하기
# Jupyter 서비스 재시작 방법
sudo systemctl restart jupyter
```
## 장치 환경 확인
* 연결된 장치 확인: lspci -k
## 드라이버 자동 설치
```
sudo apt-add-repository -r ppa:graphics-drivers/ppa
sudo apt update
sudo ubuntu-drivers autoinstall
sudo reboot
nvidia-smi
```
## 기초 라이브러리 설치
```
sudo apt install git
pip3 install matplotlib
pip3 install pandas
```
## PyTorch 설치
```
pip3 install torch torchvision
```
## 설치 이후 실행할 MNIST 예제 코드
* [MNIST 예제](https://www.kaggle.com/scottclowe/testing-gpu-enabled-notebooks-mnist-pytorch)
* 테슬라 K20으로 위 예제와 비슷한 속도로 학습이 되는 것을 확인할 수 있음.
## 학습 도중 그래픽 카드 튕기는 문제
* 학습 도중에 'Unable to determine the device handle for GPU 0000:0A:00.0: GPU is lost.'와 같은 Cuda 관련 오류가 출력이 되고, 학습에 실패하는 경우가 있음. 이럴 때는 다양한 이유가 있지만, 그래프 카드의 과열 혹은 파워 때문일 수도 있음. 일반적으로 그래픽 카드 2장과 케이스 쿨러까지 동작시키기 위해서는 800W 정도의 파워가 필요함. 그래서 이러한 문제가 발생했을 때는, 가장 먼저 그래픽 카드 발열 정도를 체크할 필요가 있음. 온도는 nvidia-smi로도 체크 가능. 본인은 간단한 모델을 학습시킬 때에도 온도가 70도 ~ 90도 이상까지 올라가는 것을 볼 수 있었음. (85도 이상은 좋지 않음.) 따라서 그래픽 카드의 발열을 잡을 수 있도록 쿨러의 위치를 변경할 필요가 있었음.
## 컴퓨터 세팅 관련 이슈
* 메인보드에 CPU는 최대한 빠르게 부착, 이후에 추가적으로 CPU 쿨러를 부착하기. 써멀 컴파운드는 발려져 있는 경우가 많은데, 한 번에 부착. (이 때 걸쇠 방식으로 부착하는 경우, 다소 장력이 강하게 부착해야 하므로 부착 전에 거리 잘 확인하기.)
* 케이스 쿨러가 3pin 짜리인 경우, 4pin 파워에 그냥 꽂아도 정상적으로 동작.
* 그래픽 카드를 바르게 잘 꽂은 뒤에, 파워 또한 8 pin 및 6 pin에 잘 꽂아야 정상적으로 동작. (하나라도 안 꽂으면 안 끼워져 있는 것과 동일.)
* 메인 그래픽 카드의 위치에 좋은 그래픽 카드를 꽂고, 나머지 위치에 안 좋은 그래픽 카드를 꽂는 방식으로 이용.
* 특정한 파워에는 6 pin만 존재함. 따라서 SATA 전원 공급을 위하여 6 pin - SATA 변환 케이블이 필요할 수 있음.
* 특정한 그래픽 카드를 대상으로 하여 동작하던 윈도우 7 PC, 다른 그래픽 카드를 꽂으면 마우스 및 키보드 인식이 안 될 수 있음. (BIOS에서는 인식.)
* PCIe x 16, PCIe x 8, PCIe x 4가 있을 때 PCIe x 4에만 그래픽 카드를 꽂으면 정상적으로 인식이 안 될 수 있음. (디스플레이가 무조건 First, Second Slot에만 있어야 하는 경우.)
* PCIe v3 x 4는 대략 4GB/s의 속도를 가짐. 따라서 SATA와 비교했을 때 느리지는 않음.
* 그래픽 카드의 크기와 PCIe 규격 등을 확인하여 물리적 간섭이 없도록 해야 함.

