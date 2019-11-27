# Kubernetes-Docker
Azure(centos)에 Docker/Kubernetes 배포... 정리


AKS를 쓰지 않고 처음부터 해보는 도커, 쿠버네티스 실습

후... 5일동안 구글, 깃, 스택오버플로우 모두 에러 찾고 .... 겨우 성공 ... 그냥 AKS를 쓰자 

## 환경
- Azure
- CentOS7
- Docker 최신버전
- 쿠버네티스 최신버전 
- 3개의 가상머신(master, work1, work2)// 같은 네트워크여야 한다.
- node.js

## 기본설정 

### master,work1,work2 3곳에서 하기(공통)

#### 1. vi /etc/hosts 호스트 추가

 
  ```shell
  172.91.2 master
  172.91.2 work1
  172.91.3 work2
  ```
#### 2. su - 
  
  - 루트계정에서 실행 
  
  - 패스워드 입력하기
  
#### 3. 방화벽 차단 및 통신 원활하게 열기
 
   ```shell
 setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

modprobe br_netfilter
  ```
  
####  4. 도커 설치 및 적용
  
   ```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce
systemctl start docker && systemctl enable docker
  ```
  
#### 5. 쿠버네티스 설치 적용
  
   ```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo

[kubernetes]

name=Kubernetes

baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64

enabled=1

gpgcheck=1

repo_gpgcheck=1

gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg

exclude=kube*

EOF
  ```
  
