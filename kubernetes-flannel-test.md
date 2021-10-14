
<h1>Kubernetes policy basic test</h1>
<br>
<b>1) Under namespaces- default access test</b>
Create pods in a namespace and try to access one pod service from another
<pre>
demo@centaurus-master:~$ kubectl create ns policy-demo
namespace/policy-demo created
demo@centaurus-master:~$ kubectl create deployment --namespace=policy-demo nginx --image=nginx
deployment.apps/nginx created
demo@centaurus-master:~$ kubectl expose --namespace=policy-demo deployment nginx --port=80
service/nginx exposed
demo@centaurus-master:~$ kubectl taint nodes $(hostname) node-role.kubernetes.io/master:NoSchedule-
I1014 05:51:56.706527    5618 request.go:668] Waited for 1.151192558s due to client-side throttling, not priority and fairness, request: GET:https://192.168.1.222:6443/apis/apiextensions.k8s.io/v1beta1?timeout=32s
node/centaurus-master untainted
demo@centaurus-master:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-q9ftl                   1/1     Running   0          25m
kube-system   coredns-558bd4d5db-xhs9l                   1/1     Running   0          25m
kube-system   etcd-centaurus-master                      1/1     Running   5          25m
kube-system   kube-apiserver-centaurus-master            1/1     Running   7          25m
kube-system   kube-controller-manager-centaurus-master   1/1     Running   1          23m
kube-system   kube-flannel-ds-lffzq                      1/1     Running   0          17m
kube-system   kube-proxy-plqtg                           1/1     Running   0          25m
kube-system   kube-scheduler-centaurus-master            1/1     Running   5          23m
policy-demo   nginx-6799fc88d8-khmj7                     1/1     Running   0          15m

demo@centaurus-master:~$ kubectl get svc -A
NAMESPACE     NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP                  27m
kube-system   kube-dns     ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   27m
policy-demo   nginx        ClusterIP   10.106.154.111   <none>        80/TCP                   17m

demo@centaurus-master:~$ kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -q nginx -O -
wget: bad address 'nginx'
/ # wget -q 10.106.154.111 -O -
wget: can't connect to remote host (10.106.154.111): Connection timed out
/ #
Session ended, resume using 'kubectl attach access -c access -i -t' command when the pod is running
pod "access" deleted
</pre>
<pre>
<b>Expected:</b> You should see a response from "nginx"
<b>Findings:</b> We didn' got response from "nginx"
</pre>

<b>2)Enable Isolation for the namespace</b>
<pre>
demo@centaurus-master:~$ kubectl create -f - <<EOF
> kind: NetworkPolicy
> apiVersion: networking.k8s.io/v1
> metadata:
>   name: default-deny
>   namespace: policy-demo
> spec:
>   podSelector:
>     matchLabels: {}
> EOF
networkpolicy.networking.k8s.io/default-deny created
demo@centaurus-master:~$ kubectl get networkpolicy
No resources found in default namespace.
demo@centaurus-master:~$ kubectl get NetworkPolicy -A
NAMESPACE     NAME           POD-SELECTOR   AGE
policy-demo   default-deny   <none>         60s

demo@centaurus-master:~$ kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 10.106.154.111 -O -
wget: download timed out
/ #

</pre>
<pre>
<b>Expected:</b> You should see a "wget: download timed out" i.e., no response from "nginx"
<b>Findings:</b> We got no response from "nginx".With clusterIP of nginx in we got "wget: download timed out" response
</pre>

<b>3)Enable access for "nginx"</b>
<pre>
demo@centaurus-master:~$ kubectl create -f - <<EOF
> kind: NetworkPolicy
> apiVersion: networking.k8s.io/v1
> metadata:
>   name: access-nginx
>   namespace: policy-demo
> spec:
  podSelector:
>   podSelector:
>     matchLabels:
>       app: nginx
>   ingress:
>     - from:
>       - podSelector:
>           matchLabels:
>             run: access
> EOF
networkpolicy.networking.k8s.io/access-nginx created
demo@centaurus-master:~$ kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 10.106.154.111 -O -
wget: download timed out
/ #
Session ended, resume using 'kubectl attach access -c access -i -t' command when the pod is running
pod "access" deleted
demo@centaurus-master:~$ kubectl run --namespace=policy-demo cant-access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ #
Session ended, resume using 'kubectl attach cant-access -c cant-access -i -t' command when the pod is running
pod "cant-access" deleted

</pre>
<pre>
<b>Expected :</b> You should see a response from "nginx" for pod with label access, else not.
<b>Findings: </b>We didn't got response from "nginx"
</pre>
<hr>

<h1>Kubernetes policy advanced test</h1>

<b>1) Ingress and egress default test</b>

Create pods in a namespace and try to access one pod service from another
<pre>
demo@centaurus-master:~$ kubectl create ns advanced-policy-demo
namespace/advanced-policy-demo created
demo@centaurus-master:~$ kubectl create deployment --namespace=advanced-policy-demo nginx --image=nginx
deployment.apps/nginx created
demo@centaurus-master:~$ kubectl expose --namespace=advanced-policy-demo deployment nginx --port=80
service/nginx exposed
demo@centaurus-master:~$ kubectl get pods -A
NAMESPACE              NAME                                       READY   STATUS    RESTARTS   AGE
advanced-policy-demo   nginx-6799fc88d8-97vqn                     1/1     Running   0          5s
kube-system            coredns-558bd4d5db-q9ftl                   1/1     Running   0          76m
kube-system            coredns-558bd4d5db-xhs9l                   1/1     Running   0          76m
kube-system            etcd-centaurus-master                      1/1     Running   5          76m
kube-system            kube-apiserver-centaurus-master            1/1     Running   7          76m
kube-system            kube-controller-manager-centaurus-master   1/1     Running   1          74m
kube-system            kube-flannel-ds-lffzq                      1/1     Running   0          67m
kube-system            kube-proxy-plqtg                           1/1     Running   0          76m
kube-system            kube-scheduler-centaurus-master            1/1     Running   5          74m
demo@centaurus-master:~$ kubectl get svc -A
NAMESPACE              NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
advanced-policy-demo   nginx        ClusterIP   10.102.84.166   <none>        80/TCP                   15s
default                kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP                  76m
kube-system            kube-dns     ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   76m

demo@centaurus-master:~$ kubectl run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 google.com -O -
wget: bad address 'google.com'
/ #
</pre>
<pre>
<b>Expected : </b> You should see a response in both the cases
<b>Findings: </b> We didn't got response for both cases
</pre>
<hr>
<b>2) Deny all ingress traffic </b>
<pre>
demo@centaurus-master:~$ kubectl create -f - <<EOF
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: default-deny-ingress
>   namespace: advanced-policy-demo
> spec:
>   podSelector:
>     matchLabels: {}
>   policyTypes:
>   - Ingress
> EOF
networkpolicy.networking.k8s.io/default-deny-ingress created
demo@centaurus-master:~$ kubectl run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 google.com -O -
wget: bad address 'google.com'
/ #
Session ended, resume using 'kubectl attach access -c access -i -t' command when the pod is running
pod "access" deleted
</pre>
<pre>
<b>Expected : </b> For "nginx" access, we should not get response, while response should come for outbound access of "google.com"
<b>Findings: </b> We didn't got response for both cases
</pre>
<hr>

<b>3)Allow ingress traffic to Nginx </b>

<pre>
demo@centaurus-master:~$ kubectl create -f - <<EOF
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: access-nginx
>   namespace: advanced-policy-demo
> spec:
>   podSelector:
>     matchLabels:
>       app: nginx
>   ingress:
>     - from:
>       - podSelector:
>           matchLabels: {}
> EOF
networkpolicy.networking.k8s.io/access-nginx created
demo@centaurus-master:~$ kubectl run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ #
Session ended, resume using 'kubectl attach access -c access -i -t' command when the pod is running
pod "access" deleted
</pre>
<pre>
<b>Expected :</b> For "nginx" access, we should get response now
<b>Findings:</b> We didn't got response from "nginx"
</pre>
<hr>
<b> 4) Deny all egress traffic </b>
<pre>
demo@centaurus-master:~$ kubectl create -f - <<EOF
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: default-deny-egress
>   namespace: advanced-policy-demo
> spec:
>   podSelector:
>     matchLabels: {}
>   policyTypes:
>   - Egress
> EOF
networkpolicy.networking.k8s.io/default-deny-egress created
demo@centaurus-master:~$ kubectl get networkpolicy -A
NAMESPACE              NAME                   POD-SELECTOR   AGE
advanced-policy-demo   access-nginx           app=nginx      2m38s
advanced-policy-demo   default-deny-egress    <none>         8s
advanced-policy-demo   default-deny-ingress   <none>         8m23s
demo@centaurus-master:~$ kubectl run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup nginx
;; connection timed out; no servers could be reached

/ # wget -q --timeout=5 google.com -O -
wget: bad address 'google.com'
/ #
Session ended, resume using 'kubectl attach access -c access -i -t' command when the pod is running
pod "access" deleted

</pre>
<pre>
<b>Expected :</b> We should get "wget: bad address 'google.com'"
<b>Findings:</b> We also got "wget: bad address 'google.com'" response
</pre>
<hr>
<b>5) Allow DNS egress traffic</b>
<pre>
demo@centaurus-master:~$ kubectl label namespace kube-system name=kube-system
metadata:
  name: allow-dns-access
  namespace: advanced-policy-demo
spec:
  podSelector:
    matchLabels: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
EOF
namespace/kube-system labeled
demo@centaurus-master:~$ kubectl create -f - <<EOF
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: allow-dns-access
>   namespace: advanced-policy-demo
> spec:
>   podSelector:
>     matchLabels: {}
>   policyTypes:
>   - Egress
>   egress:
>   - to:
>     - namespaceSelector:
>         matchLabels:
>           name: kube-system
>     ports:
>     - protocol: UDP
>       port: 53
>
> EOF
networkpolicy.networking.k8s.io/allow-dns-access created
demo@centaurus-master:~$ kubectl run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup nginx
;; connection timed out; no servers could be reached

/ # nslookup google.com
;; connection timed out; no servers could be reached

/ #
</pre>
<pre>
<b>Expected : </b>
i)For "nslookup nginx", we should get 
Server:    10.0.0.10
Address 1: 10.0.0.10 kube-dns.kube-system.svc.cluster.local

ii)For "nslookup google.com", we should get
Name:      google.com
Address 1: 2607:f8b0:4005:807::200e sfo07s16-in-x0e.1e100.net
Address 2: 216.58.195.78 sfo07s16-in-f14.1e100.net


<b>Findings: </b>
i)For "nslookup nginx", we should got
connection timed out; no servers could be reached
ii)For "nslookup google.com", we should got
connection timed out; no servers could be reached
</pre>
<hr>
<b> 6) Allow egress traffic to nginx </b>
<pre>
demo@centaurus-master:~$ kubectl create -f - <<EOF
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: allow-egress-to-advance-policy-ns
>   namespace: advanced-policy-demo
> spec:
>   podSelector:
>     matchLabels: {}
>   policyTypes:
>   - Egress
>   egress:
>   - to:
>     - podSelector:
>         matchLabels:
>           app: nginx
> EOF
networkpolicy.networking.k8s.io/allow-egress-to-advance-policy-ns created
demo@centaurus-master:~$ kubectl run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 google.com -O -
wget: bad address 'google.com'
/ #
Session ended, resume using 'kubectl attach access -c access -i -t' command when the pod is running
pod "access" deleted

</pre>
<pre>
<b>Expected : </b>
For "nginx" access, we should get response and 
for "google.com" we should get "wget: download timed out"
<b>Findings: </b>
 we didn't got response for both the cases.
</pre>

