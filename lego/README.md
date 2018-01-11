# 参考サイト
* [GKE で TLS 証明書を自動管理(cert-manager DNS-01 編) - Qiita](https://qiita.com/apstndb/items/3a39a1e6acacbbc30765)
* [GKE でサービスを HTTPS と HTTP/2 に対応する(kube-lego 編) - Qiita](https://qiita.com/apstndb/items/2fef0a80d4510516cb1f)
* [Google Container Engine上でLet's Encryptによる証明書を自動更新する（kube-lego） - Qiita](https://qiita.com/esplo/items/f386be580871aa63bdcf)
* [ブログをGKEでの運用に移行した | tsub's blog](https://blog.tsub.me/post/operate-blog-server-on-gke/)

# 設定

```
$wget https://raw.githubusercontent.com/jetstack/kube-lego/master/examples/gce/lego/00-namespace.yaml
$wget https://raw.githubusercontent.com/jetstack/kube-lego/master/examples/gce/lego/configmap.yaml
$wget https://raw.githubusercontent.com/jetstack/kube-lego/master/examples/gce/lego/deployment.yaml
```

```
$kubectl apply -f 00-namespace.yaml
$kubectl apply -f configmap.yaml
$kubectl apply -f deployment.yaml
```

```
$kubectl get deployment,replicaset,pod,configmap --namespace kube-lego
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/kube-lego   1         1         1            1           39s

NAME                      DESIRED   CURRENT   READY     AGE
rs/kube-lego-7d5bc89cb5   1         1         1         39s

NAME                            READY     STATUS    RESTARTS   AGE
po/kube-lego-7d5bc89cb5-w8m59   1/1       Running   0          39s

NAME           DATA      AGE
cm/kube-lego   2         44s
```


```
$kubectl apply -f ../ingress.yaml

$kubectl get ing -n sandbox
NAME           HOSTS             ADDRESS         PORTS     AGE
test-ingress   www.hiroki.work   35.227.228.21   80, 443   5h
```

# ERROR 
* [Unknown user error · Issue #225 · jetstack/kube-lego](https://github.com/jetstack/kube-lego/issues/225)
* [Current version may not be compatible with K8s 1.8.4-gke.0 · Issue #290 · jetstack/kube-lego](https://github.com/jetstack/kube-lego/issues/290)
```
$kubectl logs -f --namespace kube-lego $(kubectl get pod --namespace kube-lego -l app=kube-lego -o name)
E0111 13:13:37.551101       1 reflector.go:201] github.com/jetstack/kube-lego/pkg/kubelego/watch.go:105: Failed to list *v1beta1.Ingress: ingresses.extensions is forbidden: User "system:serviceaccount:kube-lego:default" cannot list ingresses.extensions at the cluster scope: Unknown user "system:serviceaccount:kube-lego:default"
```

# Misc
Non-production use caseと書いてある 
https://raw.githubusercontent.com/jetstack/kube-lego/master/README.md
