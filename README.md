* [Kubernetes で ingress controller を使って https (TLS) 接続をするチュートリアル](http://blog.lesson-time.com/tutorial-for-k8s-tls-setting/)

```
$kubectl apply -f deployment.yaml -n sandbox
```

```
$kubectl get deployment -n sandbox -o wide
NAME       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES                                           SELECTOR
test-app   1         1         1            1           2m        test-app     asia.gcr.io/geometric-sled-187609/hello:latest   run=test-app,version=latest
```

```
$kubectl get pods -n sandbox -o wide 
NAME                        READY     STATUS    RESTARTS   AGE       IP          NODE
test-app-5d4d99f68d-q2vg6   1/1       Running   0          2m        10.40.1.6   gke-sandbox-ingress-default-pool-b47b28eb-0mj2
```

```
$kubectl expose deploy/test-app -n sandbox --type=NodePort
```

```
$ kubectl get svc -n sandbox
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
test-app   NodePort   10.43.249.11   <none>        8080:30770/TCP   40s
```

```
$kubectl apply -f ingress.yaml -n sandbox

$kubectl get ing -n sandbox
NAME           HOSTS     ADDRESS   PORTS     AGE
test-ingress   *                   80        26s
```

```
$kubectl describe ing -n sandbox
Name:             test-ingress
Namespace:        sandbox
Address:          35.227.236.205
Default backend:  test-app:8080 (10.16.0.11:8080)
TLS:
  foo-secret terminates
Rules:
  Host  Path  Backends
  ----  ----  --------
  *     *     test-app:8080 (10.16.0.11:8080)
Annotations:
  url-map:                k8s-um-sandbox-test-ingress--811a75ec4cf50d72
  https-target-proxy:     k8s-tps-sandbox-test-ingress--811a75ec4cf50d72
  ssl-cert:               k8s-ssl-sandbox-test-ingress--811a75ec4cf50d72
  https-forwarding-rule:  k8s-fws-sandbox-test-ingress--811a75ec4cf50d72
  target-proxy:           k8s-tp-sandbox-test-ingress--811a75ec4cf50d72
  backends:               {"k8s-be-32542--811a75ec4cf50d72":"HEALTHY"}
  forwarding-rule:        k8s-fw-sandbox-test-ingress--811a75ec4cf50d72
Events:
  Type    Reason   Age               From                     Message
  ----    ------   ----              ----                     -------
  Normal  ADD      11m               loadbalancer-controller  sandbox/test-ingress
  Normal  CREATE   10m               loadbalancer-controller  ip: 35.227.236.205
  Normal  Service  3m (x4 over 10m)  loadbalancer-controller  default backend set to test-app:32542
```

```
#$openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./tls.key -out ./tls.crt -subj "/CN=test-app.com"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=foo.bar.com"

```

```
#$kubectl apply -f secret.yaml -n sandbox
```

```
#kubectl create secret tls foo-secret --key ./tls.key --cert ./tls.crt -n sandbox
kubectl create secret tls foo-secret --key /tmp/tls.key --cert /tmp/tls.crt -n sandbox
```

* [Ingressにstatic-ipを指定してやった on GKE and GCE - Qiita](https://qiita.com/tinjyuu/items/fd7a97b0b81963dcc7f2)
```
$gcloud compute addresses create test-ip --global

$gcloud compute addresses list --global
NAME     REGION  ADDRESS         STATUS
test-ip          35.227.236.205  RESERVED
