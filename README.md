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
## 기타 잡다한 내용들
#### Pytorch 설치
<pre>
pip3 install torch torchvision
</pre>
#### 설치 완료 체크
<pre>
python3
import torch
torch.__version__
torch.version.cuda
torch.cuda.current_device()
torch.cuda.device(0)
torch.cuda.get_device_name(0)
</pre>
#### 관련 라이브러리 설치
<pre>
pip3 install matplotlib
pip3 install pandas
</pre>
#### Cuda 설치(필요 시에 설치 진행)
* Cuda: GPU의 병렬 프로그래밍 언어. Cuda가 적용된 딥 러닝 라이브러리가 cuDNN.
* 최신 Cuda인 10.0을 설치함. 자신의 그래픽 카드가 이를 지원할 수 있는지 확인.
<pre>
wget https://developer.nvidia.com/compute/cuda/10.0/Prod/local_installers/cuda_10.0.130_410.48_linux
sudo sh cuda_10.0.130_410.48_linux
</pre>
* 확인: nvcc --version
#### cuDNN 설치(필요 시에 설치 진행)
* cuDNN은 엔비디아 CUDA 딥 뉴럴 네트워크 라이브러리. 기본적인 GPU 가속화 라이브러리 요소들을 포함.
* cuDNN 다운로드: https://developer.nvidia.com/rdp/cudnn-download
<pre>
sudo tar -xzvf cudnn-10.0-linux-x64-v7.5.1.10.tgz 
cd cuda
sudo cp include/cudnn.h /usr/local/cuda/include
sudo cp lib64/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
</pre>
#### 정상 동작 검사
<pre>
cp -r /usr/src/cudnn_samples_v7/ $HOME
cd  $HOME/cudnn_samples_v7/mnistCUDNN
make clean && make
./mnistCUDNN
</pre>
## 잡다한 지식들
* SATA와 PCIe: SATA3는 속도를 500MB/s까지 낼 수 있고, 아직 많이 사용됨. PCIe는 GB 단위로 더 빠름. 따라서 GPU도 PCIe를 이용하는 경우 더 빠르지만, 사실 사용자 단에서는 큰 차이를 못 느낌.
* 그래픽 카드 PCIe: 메인보드와 그래픽 카드를 연결하기 위한 선. PCIe 4.0 x16이 많이 사용되고 있음. 하지만 8배속 짜리에 VGA를 꽃더라도, 큰 성능 차이는 없음. 애초에 PCIe 대역폭을 전부 다 쓰는 카드가 희박하기 때문임. 따라서 그래픽 카드가 2장 있다면, 그냥 16배속과 8배속에 하나씩 꽃으면 됨. 또한 x16 크기의 슬롯이 여러 개라도, 간혹 뒤편을 보면 더 짧은 경우가 있음. 뒤편에 있는 것이 진짜 배속을 의미함.
* 패시브 쿨링: 별도의 쿨러가 붙어 있지 않고, 방열판만 존재함. 장점: 무소음, 쿨러에 대한 비용이 없음. / 단점: 내부 통풍에 따른 영향이 크기 때문에, 액티브 쿨링에 비해 쿨링 효과가 적음. 별도의 시스템 쿨러(케이스 쿨러)가 필요할 수 있음.
* 액티브 쿨링: 장점: 쿨링 효율이 높기 때문에 오버클럭에 유용하고, 안정적인 장치 운용 가능. / 단점: 쿨러 비용이 추가적으로 들고, 쿨러 또한 교체를 해 줄 필요가 있음. 소음 또한 발생.
* 쿠다(CUDA): NVIDIA가 만든 병렬 컴퓨팅 플랫폼이자 API 모델. GPU의 가상 명령어 셋을 사용할 수 있도록 도와주고, CUDA 코어가 장착된 GPU에서 동작함. 최신 쿠다 버전은 10.0이고, Tesla K20은 쿠다 툴키트 10.0을 지원하는 것으로 보임.
* 성능과 메모리: 쿠다 코어가 많고, 코어의 클럭이 높을 수록 성능이 좋음. 성능: 쿠다 코어 x 코어 클럭 / 메모리 또한 중요한데, 데이터 셋이 메모리에 올라가기 때문.
