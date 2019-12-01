# Kubernetes-Docker
Azure(centos)에 Docker/Kubernetes 배포 과정 정리


AKS를 쓰지 않고 처음부터 해보는 도커, 쿠버네티스 실습

실습하면서 헷갈리고 사이트마다 다른건 (*팡.)을 붙이겠다.

후... 5일동안 구글, 깃, 스택오버플로우 100개 넘는 블로그,사이트 모두 에러 찾고 .... 겨우 성공 ... 그냥 AKS를 쓰자 

원리는 완벽히 알 수 있음... 
## 환경
- Azure
- CentOS7
- Docker 최신버전
- 쿠버네티스 최신버전 
- 3개의 가상머신(master, work1, work2)// 같은 네트워크여야 한다.
- node.js

## 실습 시나리오

![image](https://user-images.githubusercontent.com/49789734/69837033-2004d680-1290-11ea-9204-1631342d9d05.png)

- master는 컨트롤러가 된다 (관리)
- 직접적인 pod처리는 work에서 (실행)

## 설치 및 등록 

### [master,work1,work2 3곳에서 하기(공통)]
<hr>
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
  
  
  ![8](https://user-images.githubusercontent.com/49789734/69836959-a40a8e80-128f-11ea-96f8-4a997a6a167f.png)


####  4. 도커 설치 및 적용
  
   ```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce
systemctl start docker && systemctl enable docker
  ```
  ![4](https://user-images.githubusercontent.com/49789734/69836960-a5d45200-128f-11ea-85cf-49b96ff4ed66.png)

#### 5. 쿠버네티스 설치 적용
  
   ```shell
   //파일 
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
  
  
   ```shell
 yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes// 설치
 
 systemctl enable kubelet && systemctl start kubelet//실행
  ```


### [master에서만]
<hr>
#### 1. 쿠버네티스 초기화

배포모듈에 따라서 다르다 

- kubeadm init
- kubeadm init --apiserver-advertise-address=172.22.4.5 
- kubeadm init --apiserver-advertise-address=172.22.4.5 --pod-network-cidr=10.244.0.0/16 
- 등등 

#### 2. 조인키 저장(work들 조인에 쓰임)



`kubeadm join 172.22.4.5:6443 --token gtna4q.x3j0cp16t1n0tltu --discovery-token-ca-cert-hash sha256:ccf40ff0eddbe0ed6`
- init 성공시 발급 저장하자 
- 재발급도 가능, 24시간 수명

#### 3. 이미지 받기

```shell
kubeadm config images pull
```

#### 4. 명령 root, 일반 다름 (*팡.)

- 일반에도 주기
 ```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
  ```
- 루트 진행시 
 ```shell
export KUBECONFIG=/etc/kubernetes/admin.conf
  ```


### 5. 배포 설치 (*팡.)

- 정말 여러가지가 있다. 몇개월 마다 바뀌니깐 사이트 보지말고 공식 홈페이지에서 바뀐 url 받자 

`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml`
- (2019.11.27까지는 된다. 언제 안될지 모름 그땐 공식사이트 참조) 

### [work에서만]
<hr>
아까 받은 조인키

`kubeadm join 172.22.4.5:6443 --token gtna4q.x3j0cp16t1n0tltu --discovery-token-ca-cert-hash sha256:ccf40ff0eddbe0ed6538eecaefd6259bd59db8b752bc5316d863d0a7091142d9`

성공하면 kubectl get nodes라는 메세지가 나온다.

이미 사용중 포트가 뜨면 kubeadm하고 실행 

### 정상적 연결 확인 (위 과정 끝내고 master에서 실행) 

 ```shell
 kubectl get no

 kubectl get componentstatuses

 kubectl describe pod etcd-master 

 kubectl get pods --all-namespaces
 ```
 
 ![1](https://user-images.githubusercontent.com/49789734/69838210-e2a34780-1295-11ea-90a5-cc903203b1f9.png)



## 실습


### node.js or 도커에 쓰일 파일들 설치

 - 나는 node.js 앱 설치(인터넷에서 확인!)
 
 - https://november11tech.tistory.com/159 노드 파일 등록은 여기 참조 index.js, express 등 
 
### 도커 이미지화 및 배포

 ```shell
 DockerFile 등록 마찬가지로 위 사이트( 제일 중요 !!! 이거 이해하면 도커 끝 (*팡!))
 
 docker build -t hk3/test3 . (경로/앱이름 '.'이거 꼭 해야한다.) // 컨테이너 만들기, 이미지화 하기 

 docker image list | grep hk3/test3 // 설치확인
 
 docker run -p 3003:3003 -d hk3/test3 // 도커 실행 등록한 포트에 맞추기 , 이미지 실행

 docker ps -a //실행 확인 
 ```
 > 성공시 hostip:3003 들어가면 나와야한다.

 #### 도커는 컨테이너로 만들어버려서 정말 편리하다. 하지만 이러한 컨테이너들이 많아지면 관리하기 어려워 지는데 
 #### 이러한 문제를 해결하는 것이 `쿠버네티스` 또한 다른 가상머신과도 공유까지 가능 

### 쿠버네티스 환경 배포


#### 1. 배포파일 작성

`vi deployments.yaml`

 ```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-web-app1
  labels:
    app: node-web-app1
spec:
  selector:
    matchLabels:
      app: node-web-app1
  replicas: 3  // 스케일링 부분 !!!
  template:
    metadata:
      labels:
        app: node-web-app1
    spec:
      containers:
      - name: node-web-app
        image: november11/node-web-app1
        imagePullPolicy: Never
        ports:
        - containerPort: 3005
        
 ```        

#### 2. deployment 적용 및 파드 확인 

 ```shell
kubectl apply -f deployments.yaml

kubectl get pod 
 ```     
![2](https://user-images.githubusercontent.com/49789734/69838211-e2a34780-1295-11ea-97cd-aadce101828a.png)

> Pending 나오면(*팡.)

 ```shell
kubectl taint nodes --all node-role.kubernetes.io/master-
 ```     
#### 3. 서비스파일 작성 (*팡.)(*팡.)  

`vi services.yaml`

```shell
apiVersion: v1
kind: Service
metadata:
  name: node-web-app1
spec:
  selector:
    app: node-web-app1
  ports:
  - protocol: "TCP"
    port: 3006
    targetPort: 3006
  type: LoadBalancer
  externalIPs:
  -52.141.3.115

 ```          
> externalIPs: 아이피  // 진짜 이부분 어디에도 안나옴!! (꿀팀)



#### 4. 서비스 배포 적용 및 파드 확인 

```shell
kubectl apply -f services.yaml

kubectl get services
```
![3](https://user-images.githubusercontent.com/49789734/69838212-e2a34780-1295-11ea-8988-2c98a3c3bfa1.png)
> 서비스가 작동 되면 성공!

![성공](https://user-images.githubusercontent.com/49789734/69837816-fa79cc00-1293-11ea-8085-bbdb70ad08bd.png)

#### 5. 안될때 (*팡.)

```shell
kubectl taint nodes --all node-role.kubernetes.io/master-

kubectl describe nodes node1 | grep -i taint

kubectl run testsvr --image=nginx --replicas=7

kubectl get pods -o wide | grep testsvr
```


