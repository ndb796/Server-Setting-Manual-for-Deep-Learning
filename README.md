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
User={사용자명}
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
