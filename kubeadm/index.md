# Creating kubernetes clusters using kubeadm


GCP, AWS와 같은 Cloud 환경이 아닌 물리 서버에 kubernetes cluster 구성해보자.
[Creating a single master cluster with kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

<!--more-->

## Maser node setting 

~~~bash
$ sudo kubeadm --kubernetes-version=1.13.2 init
~~~

init을 실행하면 worker node 에서 master node로 접속할 수 있는 token hash 값을 내준다. 이것을 이용해 master node에 접속한다.

`ex) kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`

{{< admonition note "Note" >}}
만약 [ERROR Swap]: running with swap on is not supported. Please disable swap 에러가 발생한다면 swapoff를 해주면된다.
{{< /admonition >}}

~~~bash
$ sudo swapoff -a
~~~

To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:

~~~bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
~~~

## Installing a pod network add-on

문서에 보면 여러가지 network를 설정할 수 있는 것들이 존재하는데 [Canal](https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/flannel)을 사용하였다.
 
~~~bash
$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
$ kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/canal/canal.yaml
~~~

잘 작동하고 있는지 pod을 확인하자.

~~~bash
$ kubectl get pods -n kube-system

NAME                          READY   STATUS              RESTARTS   AGE
canal-vb5fw                   0/3     ContainerCreating   0          27s
coredns-86c58d9df4-mf68s      0/1     Pending             0          2s
coredns-86c58d9df4-zbxnp      0/1     Pending             0          2s
etcd-eta                      1/1     Running             0          26s
kube-apiserver-eta            1/1     Running             0          45s
kube-controller-manager-eta   1/1     Running             0          33s
kube-proxy-tww4r              1/1     Running             0          82s
kube-scheduler-eta            1/1     Running             0          24s
~~~

{{< admonition note "Note" >}}
만약 coredns가 fail이 나거나 error가 발생하면 다음 설정으로 해결할 수 있다.
{{< /admonition >}}

~~~bash
$ kubectl -n kube-system edit configmap coredns
loop 삭제
$ kubectl -n kube-system delete pod -l k8s-app=kube-dns
~~~

## Joining your nodes

아까 master node에서 init할 때 나온 token hash값을 이용해서 node들을 join 시켜보자.

worker node에서는 init 할 필요없이 바로 join 명령어만 실행 하면된다.

~~~bash
$ kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>
~~~

master에서 확인해보면,

~~~bash
$ kubectl get pods -n kube-system

NAME                          READY   STATUS    RESTARTS   AGE
canal-6gg89                   3/3     Running   0          24m
canal-7dhgw                   3/3     Running   0          24m
canal-vb5fw                   3/3     Running   0          24m
coredns-86c58d9df4-mf68s      1/1     Running   0          24m
coredns-86c58d9df4-zbxnp      1/1     Running   0          24m
etcd-eta                      1/1     Running   0          24m
kube-apiserver-eta            1/1     Running   0          25m
kube-controller-manager-eta   1/1     Running   0          24m
kube-proxy-57qh5              1/1     Running   0          24m
kube-proxy-tww4r              1/1     Running   0          25m
kube-proxy-x6rng              1/1     Running   0          24m
kube-scheduler-eta            1/1     Running   0          24m
~~~

node 개수 만큼 canal, proxy등이 추가 된것을 확인 할 수 있다.

~~~bash
$ kubectl get nodes

NAME    STATUS   ROLES    AGE   VERSION
chi     Ready    <none>   14m   v1.13.2
delta   Ready    <none>   14m   v1.13.2
eta     Ready    master   16m   v1.13.2
~~~

## pod test하기 

[조대협님 블로그 node.js 코드 참조](https://bcho.tistory.com/1261?category=731548)

Deployment를 올리고

~~~yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jaejin-node-test
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: stupid-server
        image: opiximeo/jaejintest:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
~~~

외부에서 접속하기 위해 service를 올린다.

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-node-svc
spec:
  selector:
    app: myapp
  ports:
  - port: 5920
    protocol: TCP
    targetPort: 8080
  externalIPs:
  - {your server ip}
~~~

cloud에서 service에 `type: LoadBalancer`를 설정해주면 자동으로 외부 IP를 가져오지만 cloud에서 올린게 아니라서 자동으로 가져오지는 못한다.

따라서 externalIPs 옵션을 줘서 접속할 수 있는 IP를 명시해주면 된다.

http://{your server ip}:5920


