
# Question: 
Implement Ingress to route traffic to two different Services from the same Host and path.





# Ans :
 Attache Helm chart with the solution.
./ing-question
```sh
ing-question/
├── Chart.yaml
├── charts
├── templates
│   ├── backends-ingres.yaml
│   ├── backends-service.yaml
│   ├── deploymeny.yaml
│   ├── ingress.yaml
│   └── service.yaml
└── values.yaml
```



## Pods

```sh
 ~ $ k get pods --show-labels=true
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
app-a-64bbcfb888-mcmp6   1/1     Running   0          16m   app=app-a,component=backend,pod-template-hash=64bbcfb888
app-b-5ff8984986-zzdg4   1/1     Running   0          16m   app=app-b,component=backend,pod-template-hash=5ff8984986
 ~ $ 
```

## Services
``` yaml
 ~ $ k describe svc 
Name:              app-a
Namespace:         ing-test
Labels:            app=app-a
                   app.kubernetes.io/managed-by=Helm
Annotations:       meta.helm.sh/release-name: ing-test
                   meta.helm.sh/release-namespace: ing-test
Selector:          app=app-a
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.0.205.150
IPs:               10.0.205.150
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.6.10:8080
Session Affinity:  None
Events:            <none>


Name:              app-b
Namespace:         ing-test
Labels:            app=app-b
                   app.kubernetes.io/managed-by=Helm
Annotations:       meta.helm.sh/release-name: ing-test
                   meta.helm.sh/release-namespace: ing-test
Selector:          app=app-b
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.0.76.98
IPs:               10.0.76.98
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.27.124:8080
Session Affinity:  None
Events:            <none>


Name:              backend
Namespace:         ing-test
Labels:            app.kubernetes.io/managed-by=Helm
                   component=backend
Annotations:       meta.helm.sh/release-name: ing-test
                   meta.helm.sh/release-namespace: ing-test
Selector:          component=backend
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.0.96.38
IPs:               10.0.96.38
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.27.124:8080,10.244.6.10:8080
Session Affinity:  None
Events:            <none>
```

## Imgresses
```sh

 ~ $ k get ing
NAME      CLASS   HOSTS                     ADDRESS          PORTS   AGE
app-a     nginx   kubernetes.app-a.com      20.101.202.250   80      19m
app-b     nginx   kubernetes.app-b.com      20.101.202.250   80      19m
backend   nginx   kubernetes.backends.com   20.101.202.250   80      19m
 ~ $ 
 ```



 Get response for each Ingress :
```sh
k get ing  -o custom-columns=HOST_NAME:.spec.rules[0].host --no-headers=true
kubernetes.app-a.com
kubernetes.app-b.com
kubernetes.backends.com
 ~ $ for host in $(k get ing  -o custom-columns=HOST_NAME:.spec.rules[0].host --no-headers=true) ; do curl $host ; done
APP-A
APP-B
APP-B
 ~ $
```


Each time a request is sent to  `kubernetes.backends.com ` Ingress a diffrent service is reponding.
```hs
 ~ $ for i in {1..10} ; do curl kubernetes.backends.com   ; done
APP-A
APP-A
APP-B
APP-A
APP-B
APP-B
APP-B
APP-B
APP-A
APP-A
 ~ $
```

