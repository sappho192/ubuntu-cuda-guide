# 요약
연구실 서버의 기존 운영체제였던 16.04에서 20.04로 업그레이드하면서 진행했던 내용들을 참고용으로 남겨놓습니다.

## 우분투(리눅스) 재설치
- 우분투 공식 홈페이지에서 서버용 우분투 (LTS 권장)를 받은 후 빈 USB메모리와 Rufus를 이용해 우분투 설치 미디어를 만든다.
- 재설치할 본체를 끄고 (중요) 우분투 설치 USB메모리를 꽂은 다음 전원을 켠다
- 부팅되는 동안 계속 DEL 키를 눌러서 바이오스에 진입한다.
- Secure Boot을 끈다 (안 끄면 NVIDIA 드라이버 설치 때 **굉장히** 귀찮아짐)
- 부팅 순서 목록에 가서 우분투가 설치된 USB 메모리를 최상위에 둔 다음 설정을 저장하고 재부팅을 한다
- 우분투 설치를 진행한다  

### 우분투 설치 과정
- 설치 언어는 영어 권장
- APT 패키지 미러 사이트는 `http://kr.archive.ubuntu.com`에서 `http://mirror.kakao.com`으로 바꿔준다 (앞 미러는 가끔 헤까닥 하는 경우가 있어서)
  - 바꾸는 방법 [[링크](https://teddylee777.github.io/linux/ubuntu%EC%97%90%EC%84%9C-apt-get%EC%98%A4%EB%A5%98%EC%8B%9C-mirror%EC%82%AC%EC%9D%B4%ED%8A%B8-%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8%EB%B0%A9%EB%B2%95/)]
- 네트워크 장치들(eth0, enp0s0 같은거) 나오는 화면에서 랜선이 꽂혀있는 장치(안 꽂혀있는건 Not connected라고 나온다)를 선택하고 Automatic(DHCP)에서 Manual로 바꾼 후 IP 설정을 해준다

## 우분투 초기 설정
- 우분투 설치때 인터넷 설정을 해줬다면 아래 단계는 넘어가도됨
- `/etc/netplan` 안에 yml 파일이 하나 있는데 cp 명령어를 이용해 하나 백업해둔 다음 (보통 `cp a.yml a.yml.bak` 같은 식으로 백업함) vim으로 아래와 비슷하게 되어있는지 확인한다.
- 만약 이더넷 장치(eno1, enp0s3 등)가 여러 개 나열되어있으면 본인이 IP설정을 해둔 장치만 남겨놓고 나머진 지운다. 그러지 않으면 부팅이 오래걸림
```shell
# This is the network config written by 'subiquity'
network:
  ethernets:
    # 아래 enp0s3 이름 대신 실제 장치이름을 넣는다
    enp0s3:
      # 아래 999 대신에 실제 서버IP와 게이트웨이 주소를 넣는다
      addresses: [999.999.999.999/24]
      gateway4: 999.999.999.1
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
  version: 2
```
출처: https://www.manualfactory.net/13079

- 재부팅 한 다음 로그인하고 `sudo apt update`, `sudo apt upgrade`로 운영체제를 업데이트한다

## NVIDIA 드라이버, CUDA Toolkit, cuDNN 설치

### A) 만약 CUDA 업그레이드를 위해 재설치를 하는 경우라면!
해당하지 않으면 바로 B)로 넘어가고, 해당하면 터미널에서 아래 명령어들을 입력해서 기존에 깔려있던 NVIDA 드라이버와 CUDA Toolkit, CuDNN을 모두 없애준다.

```bash
sudo apt-get purge nvidia* 
sudo apt-get autoremove
sudo apt-get autoclean
sudo rm -rf /usr/local/cuda*
```
`/usr/local/` 폴더에 cuda 뭔가가 남아있음 지운다

### B) 나머지 단계

https://nirsa.tistory.com/332 여기를 참고했긴 한데 중간에 다운받는 파일 유형이나 설치 방법이 조금씩 다르니 주의할 것  

```shell
sudo apt update 
sudo apt install -y ubuntu-drivers-common
ubuntu-drivers devices
```
이 때 `nvidia-driver-470` 같은 게 나올텐데, 버전이 가장 높으면서 `nvidia-driver-470-server` 처럼 뒤에 `server`가 붙어있는 것을 설치해야 한다. 안 붙어있는걸 설치하면 GUI 데스크탑 패키지가 설치되어서 서버 컴퓨터 앞에서 작업할 때 귀찮아짐... 만약 실수로 안 붙어있는걸 설치했다면 해결방법은 알아서 찾아보세요. 있긴함
```
sudo apt install nvidia-driver-470-server
```
설치 후 **반드시 재부팅**한다. 그런 다음 `nvidia-smi`를 입력해보면 드라이버가 온전히 설치된 것이 확인될 것이다.

그 다음 cuDNN을 설치해야 한다.  
아래 링크로 들어가서 본인의 운영체제와 시스템에 맞는 옵션을 선택하면, `Installation Instructions:`로 시작하는 가이드가 나온다.   
그걸 따라 설치하면 된다.
https://developer.nvidia.com/cudnn-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu

예시
```
wget https://developer.download.nvidia.com/compute/cudnn/9.1.1/local_installers/cudnn-local-repo-ubuntu2004-9.1.1_1.0-1_amd64.deb
sudo dpkg -i cudnn-local-repo-ubuntu2004-9.1.1_1.0-1_amd64.deb
sudo cp /var/cudnn-local-repo-ubuntu2004-9.1.1/cudnn-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cudnn
```

아래 방법은 cuDNN 9부터는 해당되지 않는 내용이지만, 일단 남겨는 놓습니다.
### 예전 방법

**이 시점에서 https://developer.nvidia.com 에 접속해서 NVIDIA 계정을 만든다. 계정이 있어야 cuDNN 다운로드가 가능함**

CUDA Toolkit은 자신의 드라이버가 지원하면서 cuDNN도 지원이 되면서 최신인 버전을 골라야한다는 번거로움이 있다. 다만 기본적으로 `드라이버가 구형 + Toolkit&cuDNN이 신형`인 조합은 문제없이 동작한다는 점에 착안하여,  
그냥 위에서 드라이버를 적당히 최신으로 설치해뒀다면 https://developer.nvidia.com/cudnn 에서 최신 cuDNN을 먼저 확인한 다음 (`Download cuDNN v8.3.2 for CUDA 11.5` 처럼 적혀있다) 거기에 적힌 CUDA Toolkit 버전 (11.5 같은)을 https://developer.nvidia.com/cuda-toolkit-archive 여기에서 가이드를 따라가면서 설치해준다. 대충 아래와 같이 선택한 다음 가이드에 적힌 터미널 명령어들을 한 줄씩 복사해서 엔터치면 된다.  

![cudatoolkit-options.png](/files/cudatoolkit-options.png)  

cuDNN도 받은 다음에 `sudo dpkg -i cudnn어쩌구` 같은 식으로 설치해준다.  

그런 다음 컴퓨터를 재부팅한다.

#### NVCC 업데이트
아래 명령어를 터미널에서 입력하여 /etc/bash.bashrc 상의 환경변수에 새로 설치한 CUDA의 경로를 추가한다.
이때 `쿠다버전`은 `/usr/local`에 있는 `cuda-11.4` 와 비슷한 이름의 쿠다 폴더가 있으니 그 숫자를 똑같이 적으면 된다.
```Shell
$ sudo sh -c "echo 'export PATH=$PATH:/usr/local/cuda-쿠다버전/bin' >> /etc/bash.bashrc"
$ sudo sh -c "echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-쿠다버전/lib64' >> /etc/bash.bashrc"
$ sudo sh -c "echo 'export CUDADIR=/usr/local/cuda-쿠다버전' >> /etc/bash.bashrc"
```

`sudo vim /etc/bash.bashrc`로 추가한 내용이 잘 들어가있는지 확인한다.

현재 터미널을 닫고, 새 터미널을 연 다음 `nvcc -V`를 입력했을때 CUDA 버전이 방금 추가한 쿠다 폴더의 버전과 동일하면 끝!

## 마무리
`sudo apt update`, `sudo apt upgrade`로 운영체제를 업데이트한다.

## 팁
sudo user 만들기 - [링크](https://linuxize.com/post/how-to-create-a-sudo-user-on-ubuntu/)
