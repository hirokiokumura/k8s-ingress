# 参考サイト
* [Kubernetes で ingress controller を使って https (TLS) 接続をするチュートリアル](http://blog.lesson-time.com/tutorial-for-k8s-tls-setting/)
* [contrib/ingress/controllers/nginx/examples/tls at master · kubernetes/contrib](https://github.com/kubernetes/contrib/tree/master/ingress/controllers/nginx/examples/tls)
* [Ingressにstatic-ipを指定してやった on GKE and GCE - Qiita](https://qiita.com/tinjyuu/items/fd7a97b0b81963dcc7f2)
* [Ingress での HTTP 負荷分散の設定  |  Kubernetes Engine のドキュメント  |  Google Cloud Platform](https://cloud.google.com/kubernetes-engine/docs/tutorials/http-balancer?hl=ja)
* [Kubernetes Engineでの Jenkins の設定  |  ソリューション  |  Google Cloud Platform](https://cloud.google.com/solutions/jenkins-on-container-engine-tutorial?hl=ja)
* [Ingress on GKE/GCE](https://www.slideshare.net/shoutayoshikai/ingress-on-gkegce)
* [GCPUG Tokyo+Osakaで話す予定だった話 – Daisuke Maki – Medium](https://medium.com/@lestrrat/gcpug-tokyo-osaka%E3%81%A7%E8%A9%B1%E3%81%99%E4%BA%88%E5%AE%9A%E3%81%A0%E3%81%A3%E3%81%9F%E8%A9%B1-8076bf314777)
* [GKEにデプロイしHTTPロードバランシングする - ソフトウェア開発備忘録](http://yuki-toida.hatenablog.com/entry/2017/09/21/000000)

# Command
## クラスタ作成
```
gcloud container clusters create sandbox \
--machine-type=n1-standard-1 \
--zone=asia-northeast1-a \
--num-nodes=1 \
--network=dev \
--subnetwork=dev-asia-northeast1  \
--disk-size=100 \
--cluster-version=1.8.5-gke.0 \
--username=admin \
--password=1234567890123456
```

## 静的IPアドレス取得
```
$gcloud compute addresses create test-ip --global

$gcloud compute addresses list --global
NAME     REGION  ADDRESS         STATUS
test-ip          35.227.228.21  RESERVED
```

## namespaceとdeploymentを作成
```
$kubectl apply -f deployment.yaml -n sandbox
```

### deployment確認
```
$kubectl get pod -n sandbox -o wide
NAME                        READY     STATUS    RESTARTS   AGE       IP           NODE
test-app-5d4d99f68d-d72sn   1/1       Running   0          32m       10.16.0.11   gke-sandbox-default-pool-576b7af2-0twc
```

### pod確認
```
$kubectl get deployment -n sandbox -o wide
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES                                           SELECTOR
test-app   1         1         1            1           32m       test-app     asia.gcr.io/geometric-sled-187609/hello:latest   run=test-app,version=latest
```

## service作成
```
$kubectl expose deploy/test-app -n sandbox --type=NodePort

or

$kubectl apply -f service.yaml -n sandbox
```

### service確認
```
$kubectl get service -n sandbox -o wide
NAME       TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE       SELECTOR
test-app   NodePort   10.19.249.226   <none>        8080:31785/TCP   33m       run=test-app
```

## 証明書
### 作成
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=foo.bar.com"
```

### Secret作成
```
kubectl create secret tls foo-secret --key /tmp/tls.key --cert /tmp/tls.crt -n sandbox
```

## ingress作成
```
$kubectl apply -f ingress.yaml -n sandbox

$kubectl get ing -n sandbox
NAME           HOSTS     ADDRESS         PORTS     AGE
test-ingress   *         35.227.228.21   80, 443   33m
```

### ingress確認
* GCPの[Kubernetes Enging] > [検出と負荷分散]にingress作成までのインジケータが表示されるので進捗確認ができる

```
$kubectl  describe ing -n sandbox                                                   (gke_geometric-sled-187609_asia-northeast1-a_sandbox/default)
Name:             test-ingress
Namespace:        sandbox
Address:          35.227.228.21
Default backend:  test-app:8080 (10.16.0.11:8080)
TLS:
  foo-secret terminates
Rules:
  Host  Path  Backends
  ----  ----  --------
  *     *     test-app:8080 (10.16.0.11:8080)
Annotations:
  https-forwarding-rule:  k8s-fws-sandbox-test-ingress--8b2072c9ee9834f5
  url-map:                k8s-um-sandbox-test-ingress--8b2072c9ee9834f5
  backends:               {"k8s-be-31785--8b2072c9ee9834f5":"HEALTHY"}
  forwarding-rule:        k8s-fw-sandbox-test-ingress--8b2072c9ee9834f5
  target-proxy:           k8s-tp-sandbox-test-ingress--8b2072c9ee9834f5
  https-target-proxy:     k8s-tps-sandbox-test-ingress--8b2072c9ee9834f5
  ssl-cert:               k8s-ssl-sandbox-test-ingress--8b2072c9ee9834f5
Events:
  Type    Reason   Age               From                     Message
  ----    ------   ----              ----                     -------
  Normal  ADD      21m               loadbalancer-controller  sandbox/test-ingress
  Normal  CREATE   20m               loadbalancer-controller  ip: 35.227.228.21
  Normal  Service  2m (x6 over 20m)  loadbalancer-controller  default backend set to test-app:31785
```

