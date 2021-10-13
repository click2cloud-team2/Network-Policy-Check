
<h1>Kubernetes policy basic test</h1>
<br>
<b>1) Under namespaces- default access test</b>
Create pods in a namespace and try to access one pod service from another
<pre>
demo@centaurus:~/arktos$ cluster/kubectl.sh create ns policy-demo
namespace/advanced-policy-demo created
demo@centaurus:~/arktos$ cluster/kubectl.sh create deployment --namespace=policy-demo nginx --image=nginx
deployment.apps/nginx created
demo@centaurus:~/arktos$ cluster/kubectl.sh expose --namespace=policy-demo deployment nginx --port=80
service/nginx exposed
demo@centaurus:~/arktos$ cluster/kubectl.sh get svc -A
NAMESPACE              NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default                kubernetes           ClusterIP   10.0.0.1     <none>        443/TCP                  51m
default                kubernetes-default   ClusterIP   10.0.0.121   <none>        443/TCP                  50m
kube-system            kube-dns             ClusterIP   10.0.0.10    <none>        53/UDP,53/TCP            51m
kube-system            kube-dns-default     ClusterIP   10.0.0.252   <none>        53/UDP,53/TCP,9153/TCP   50m  
policy-demo            nginx                ClusterIP   10.0.0.150    <none>       80/TCP                   18s

demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q 10.0.0.150:80 -O -
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ # 
</pre>
<pre>
<b>Expected:</b> You should see a response from "nginx"
<b>Findings:</b> By using clusterIP of nginx in "wget -q nginx -O -" we got response from "nginx"
</pre>
<b>2)Enable Isolation for the namespace</b>
<pre>
demo@centaurus:~/arktos$ cluster/kubectl.sh create -f - <<EOF
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: default-deny
>   namespace: policy-demo
> spec:
>   podSelector:
>     matchLabels: {}
> EOF
networkpolicy.networking.k8s.io/default-deny created
demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 10.0.0.150 -O -
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
</pre>
<pre>
<b>Expected:</b> You should see a "wget: download timed out" i.e., no response from "nginx"
<b>Findings:</b> By using clusterIP of nginx in "wget -q nginx -O -" we got response from "nginx"
</pre>

<b>3)Enable access for "nginx"</b>
<pre>
demo@centaurus:~/arktos$ cluster/kubectl.sh create -f - <<EOF
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: access-nginx
>   namespace: policy-demo
> spec:
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
demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 10.0.0.150 -O -
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
</pre>
<pre>
<b>Expected :</b> You should see a response from "nginx"
<b>Findings: </b>By using clusterIP of nginx in "wget -q nginx -O -" we got response from "nginx"
</pre>
<hr>

<h1>Kubernetes policy advanced test</h1>

<b>1) Ingress and egress default test</b>

Create pods in a namespace and try to access one pod service from another
<pre>
demo@centaurus:~/arktos$ cluster/kubectl.sh create ns advanced-policy-demo
namespace/advanced-policy-demo created
demo@centaurus:~/arktos$ cluster/kubectl.sh create deployment --namespace=advanced-policy-demo nginx --image=nginx
deployment.apps/nginx created
demo@centaurus:~/arktos$ cluster/kubectl.sh expose --namespace=advanced-policy-demo deployment nginx --port=80
service/nginx exposed
demo@centaurus:~/arktos$ cluster/kubectl.sh get svc -A
NAMESPACE              NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
advanced-policy-demo   nginx                ClusterIP   10.0.0.65    <none>        80/TCP                   107s
default                kubernetes           ClusterIP   10.0.0.1     <none>        443/TCP                  3h30m
default                kubernetes-default   ClusterIP   10.0.0.121   <none>        443/TCP                  3h30m
kube-system            kube-dns             ClusterIP   10.0.0.10    <none>        53/UDP,53/TCP            3h30m
kube-system            kube-dns-default     ClusterIP   10.0.0.252   <none>        53/UDP,53/TCP,9153/TCP   3h30m  

demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 10.0.0.65 -O -
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ # wget -q --timeout=5 google.com -O -
wget: bad address 'google.com'
/ # wget -q --timeout=5 142.250.76.196 -O -
wget: download timed out
/ #
</pre>
<pre>
<b>Expected : </b> You should see a response in both the cases
<b>Findings: </b>i) By using clusterIP of nginx in "wget -q nginx -O -" we got response from "nginx"
	  ii) For access to "google.com", we didn't get response
</pre>
<hr>
<b>2) Deny all ingress traffic </b>
<pre>
demo@centaurus:~/arktos$ cluster/kubectl.sh create -f - <<EOF
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
demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 10.0.0.65 -O -
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ # wget -q --timeout=5 google.com -O -
wget: bad address 'google.com'
/ # wget -q --timeout=5 142.250.76.196 -O -
wget: download timed out
/ #
Session ended, resume using 'kubectl attach access-dddf48dd6-tqsbg -c access -i -t' command when the pod is running
deployment.apps "access" deleted
</pre>
<pre>
<b>Expected : </b> For "nginx" access, we should not get response, while response should come for outbound access of "google.com"
<b>Findings: </b> i) By using clusterIP of nginx in "wget -q nginx -O -" we got response from "nginx"
	  ii) For access to "google.com", we didn't get response
</pre>
<hr>

<b>
3)Allow ingress traffic to Nginx </b>

<pre>
demo@centaurus:~/arktos$ cluster/kubectl.sh create -f - <<EOF
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
demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 10.0.0.65 -O -
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
<p><em>Thank you for using nginx.</em></p>
</body>
</html>
/ #
Session ended, resume using 'kubectl attach access-dddf48dd6-4ttnf -c access -i -t' command when the pod is running
deployment.apps "access" deleted
demo@centaurus:~/arktos$ cluster/kubectl.sh create -f - <<EOF
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
</pre>
<pre>
<b>Expected :</b> For "nginx" access, we should get response now
<b>Findings:</b> By using clusterIP of nginx in "wget -q nginx -O -" we got response from "nginx"
</pre>
<hr>
<b> 4) Deny all egress traffic </b>
<pre>
demo@centaurus:~/arktos$ cluster/kubectl.sh create -f - <<EOF
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
demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # nslookup nginx
;; connection timed out; no servers could be reached

/ # nslookup 10.0.0.65
;; connection timed out; no servers could be reached

/ # wget -q --timeout=5 google.com -O -
wget: bad address 'google.com'
/ # wget -q --timeout=5 142.250.76.196 -O -
wget: download timed out
/ #
Session ended, resume using 'kubectl attach access-dddf48dd6-v9f4n -c access -i -t' command when the pod is running
deployment.apps "access" deleted
</pre>
<pre>
<b>Expected :</b> We should get "wget: bad address 'google.com'"
<b>Findings:</b> We also got "wget: bad address 'google.com'" response
</pre>
<hr>
<b>5) Allow DNS egress traffic</b>
<pre>
demo@centaurus:~/arktos$ cluster/kubectl.sh label namespace kube-system name=kube-system
error: 'name' already has a value (kube-system), and --overwrite is false
demo@centaurus:~/arktos$ cluster/kubectl.sh create -f - <<EOF
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
    - namespaceSelector:
        matchLabels:
>     - namespaceSelector:
>         matchLabels:
>           name: kube-system
>     ports:
>     - protocol: UDP
>       port: 53
>
> EOF
networkpolicy.networking.k8s.io/allow-dns-access created
demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # nslookup nginx
;; connection timed out; no servers could be reached

/ # nslookup google.com
;; connection timed out; no servers could be reached

/ # nslookup 10.0.0.65
;; connection timed out; no servers could be reached

/ # nslookup 142.250.76.196
;; connection timed out; no servers could be reached

/ #
Session ended, resume using 'kubectl attach access-dddf48dd6-tspr2 -c access -i -t' command when the pod is running
deployment.apps "access" deleted
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
demo@centaurus:~/arktos$ cluster/kubectl.sh create -f - <<EOF
> apiVersion: networking.k8s.io/v1
> kind: NetworkPolicy
> metadata:
>   name: allow-egress-to-advance-policy-ns
>   namespace: advanced-policy-demo
> spec:
>   podSelector:
>     matchLabels: {}
>   policyTypes:
        matchLabels:
          app: nginx
>   - Egress
>   egress:
>   - to:
>     - podSelector:
>         matchLabels:
>           app: nginx
> EOF
networkpolicy.networking.k8s.io/allow-egress-to-advance-policy-ns created
demo@centaurus:~/arktos$ cluster/kubectl.sh run --namespace=advanced-policy-demo access --rm -ti --image busybox /bin/sh
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
/ # create -f - <<EOF
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
> ^C
/ # wget -q --timeout=5 nginx -O -
wget: bad address 'nginx'
/ # wget -q --timeout=5 10.0.0.65 -O -
wget: download timed out
/ # wget -q --timeout=5 google.com -O -
wget: bad address 'google.com'
/ # wget -q --timeout=5 142.250.76.196 -O -
wget: download timed out
/ #
</pre>
<pre>
<b>Expected : </b>
For "nginx" access, we should get response and 
for "google.com" we should get "wget: download timed out"
<b>Findings: </b>
For i) we got "wget: bad address 'nginx'"
For ii) we got " wget: bad address 'google.com'"
</pre>

