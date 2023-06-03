# 진짜진짜 쉬운 통합 운영환경 (Monitoring & Logging)

<img src="https://capsule-render.vercel.app/api?type=waving&color=auto&height=200&section=header&text=통합모니터링&fontSize=90" />

<p align='center'> kind로 클러스터부터 설치해봅시다!! </p>
<p align='center'>
  <a href="https://github.com/sangwonchoi/observability/tree/main">
    <img src="https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white" />
  </a>
  <a href="#prometheus">
    <img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white" />
  </a>
</p>

1. Mac 환경에서의 진행

순서는 다음과 같습니다.

a. Prerequiste

준비물로는 Go 1.16+와 Docker가 필요합니다.

Docker Download : https://www.docker.com/ 

```
sangwon@Sangwons-MacBook-Air ~ % brew install golang
sangwon@Sangwons-MacBook-Air ~ % go version
go version go1.19.2 darwin/amd64
```

b. kind 설치

```
sangwon@Sangwons-MacBook-Air ~ % go install sigs.k8s.io/kind@v0.19.0

sangwon@Sangwons-MacBook-Air ~ % export PATH=$PATH:$(go env GOPATH)/bin
```
### go 환경을 못잡아 kind 명령어가 쳐지지 않을때 실행

c. Singlenode Cluster 설치

벌써 준비가 끝났습니다. 이제 Kind로 쿠버네티스 클러스터를 만들어보면 됩니다.

```
sangwon@Sangwons-MacBook-Air ~ % kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.27.1) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```

d. Multinode Cluster 설치

Single Cluster에 Multi node가 선언된 Cluster를 설치해봅니다.

```
sangwon@Sangwons-MacBook-Air ~ % mkdir kind_multinode
sangwon@Sangwons-MacBook-Air ~ % cd kind_multinode
sangwon@Sangwons-MacBook-Air kind_multinode % vi kind-config,yaml
### 복사 붙여넣기 합니다.
# this config file contains all config fields with comments
# NOTE: this is not a particularly useful config file
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
# patch the generated kubeadm config with some extra settings
kubeadmConfigPatches:
- |
  apiVersion: kubelet.config.k8s.io/v1beta1
  kind: KubeletConfiguration
  evictionHard:
    nodefs.available: "0%"
# patch it further using a JSON 6902 patch
kubeadmConfigPatchesJSON6902:
- group: kubeadm.k8s.io
  version: v1beta3
  kind: ClusterConfiguration
  patch: |
    - op: add
      path: /apiServer/certSANs/-
      value: my-hostname
# 1 control plane node and 3 workers
nodes:
# the control plane node config
- role: control-plane
# the three workers
- role: worker
- role: worker
- role: worker
```
```
sangwon@Sangwons-MacBook-Air kind_multinode % kubectl get nodes -o wide
NAME                 STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION     CONTAINER-RUNTIME
kind-control-plane   Ready    control-plane   19h   v1.27.1   172.18.0.5    <none>        Debian GNU/Linux 11 (bullseye)   5.10.47-linuxkit   containerd://1.6.21
kind-worker          Ready    <none>          19h   v1.27.1   172.18.0.4    <none>        Debian GNU/Linux 11 (bullseye)   5.10.47-linuxkit   containerd://1.6.21
kind-worker2         Ready    <none>          19h   v1.27.1   172.18.0.3    <none>        Debian GNU/Linux 11 (bullseye)   5.10.47-linuxkit   containerd://1.6.21
kind-worker3         Ready    <none>          19h   v1.27.1   172.18.0.2    <none>        Debian GNU/Linux 11 (bullseye)   5.10.47-linuxkit   containerd://1.6.21
```

```
sangwon@Sangwons-MacBook-Air kind_multinode % kind create cluster --name test --config kind2-config.yaml
Creating cluster "test" ...
 ✓ Ensuring node image (kindest/node:v1.27.1) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-test"
You can now use your cluster with:

kubectl cluster-info --context kind-test

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

sangwon@Sangwons-MacBook-Air kind_multinode %  kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          docker-desktop   docker-desktop   docker-desktop
          kind-kind        kind-kind        kind-kind
*         kind-test        kind-test        kind-test
          minikube         minikube         minikube         default
```
Kind를 통해 Local 환경에서 Multi Cluster 환경을 구축해보았습니다.

```
sangwon@Sangwons-MacBook-Air ~ % kubectl config use-context kind-kind
Switched to context "kind-kind".
sangwon@Sangwons-MacBook-Air ~ % kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          docker-desktop   docker-desktop   docker-desktop
*         kind-kind        kind-kind        kind-kind
          kind-test        kind-test        kind-test
          minikube         minikube         minikube         default
```

이렇게 클러스터 단위를 Switch 하며 오갈 수 있습니다.