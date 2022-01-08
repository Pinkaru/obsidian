author:: null
source:: [No route to host from some Kubernetes containers to other containers in same cluster](https://stackoverflow.com/questions/58860745/no-route-to-host-from-some-kubernetes-containers-to-other-containers-in-same-clu)
clipped:: [[2022-01-04]]
published:: 

#clippings

This is a Kubespray deployment using calico. All the defaults are were left as-is except for the fact that there is a proxy. Kubespray ran to the end without issues.

Access to Kubernetes services started failing and after investigation, there was **no route to host** to the *coredns* service. Accessing a K8S service by IP worked. Everything else seems to be correct, so I am left with a cluster that works, but without DNS.

Here is some background information: Starting up a busybox container:

```

Server:     169.254.25.10
Address:    169.254.25.10:53

** server can't find kubernetes.default: NXDOMAIN

*** Can't find kubernetes.default: No answer
```

Now the output while explicitly defining the IP of one of the CoreDNS pods:

```

;; connection timed out; no servers could be reached
```

Notice that telnet to the Kubernetes API works:

```

Connected to 10.233.0.1
```

**kube-proxy logs:** 10.233.0.3 is the service IP for coredns. The last line looks concerning, even though it is INFO.

```
$ kubectl logs kube-proxy-45v8n -nkube-system
I1114 14:19:29.657685       1 node.go:135] Successfully retrieved node IP: X.59.172.20
I1114 14:19:29.657769       1 server_others.go:176] Using ipvs Proxier.
I1114 14:19:29.664959       1 server.go:529] Version: v1.16.0
I1114 14:19:29.665427       1 conntrack.go:52] Setting nf_conntrack_max to 262144
I1114 14:19:29.669508       1 config.go:313] Starting service config controller
I1114 14:19:29.669566       1 shared_informer.go:197] Waiting for caches to sync for service config
I1114 14:19:29.669602       1 config.go:131] Starting endpoints config controller
I1114 14:19:29.669612       1 shared_informer.go:197] Waiting for caches to sync for endpoints config
I1114 14:19:29.769705       1 shared_informer.go:204] Caches are synced for service config 
I1114 14:19:29.769756       1 shared_informer.go:204] Caches are synced for endpoints config 
I1114 14:21:29.666256       1 graceful_termination.go:93] lw: remote out of the list: 10.233.0.3:53/TCP/10.233.124.23:53
I1114 14:21:29.666380       1 graceful_termination.go:93] lw: remote out of the list: 10.233.0.3:53/TCP/10.233.122.11:53
```

All pods are running without crashing/restarts etc. and otherwise services behave correctly.

IPVS looks correct. CoreDNS service is defined there:

```

IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.233.0.1:443 rr
  -> x.59.172.19:6443           Masq    1      0          0         
  -> x.59.172.20:6443           Masq    1      1          0         
TCP  10.233.0.3:53 rr
  -> 10.233.122.12:53             Masq    1      0          0         
  -> 10.233.124.24:53             Masq    1      0          0         
TCP  10.233.0.3:9153 rr
  -> 10.233.122.12:9153           Masq    1      0          0         
  -> 10.233.124.24:9153           Masq    1      0          0         
TCP  10.233.51.168:3306 rr
  -> x.59.172.23:6446           Masq    1      0          0         
TCP  10.233.53.155:44134 rr
  -> 10.233.89.20:44134           Masq    1      0          0         
UDP  10.233.0.3:53 rr
  -> 10.233.122.12:53             Masq    1      0          314       
  -> 10.233.124.24:53             Masq    1      0          312
```

Host routing also looks correct.

```

default via x.59.172.17 dev ens3 proto dhcp src x.59.172.22 metric 100 
10.233.87.0/24 via x.59.172.21 dev tunl0 proto bird onlink 
blackhole 10.233.89.0/24 proto bird 
10.233.89.20 dev calib88cf6925c2 scope link 
10.233.89.21 dev califdffa38ed52 scope link 
10.233.122.0/24 via x.59.172.19 dev tunl0 proto bird onlink 
10.233.124.0/24 via x.59.172.20 dev tunl0 proto bird onlink 
x.59.172.16/28 dev ens3 proto kernel scope link src x.59.172.22 
x.59.172.17 dev ens3 proto dhcp scope link src x.59.172.22 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
```

I have redeployed this same cluster in separate environments with flannel and calico with iptables instead of ipvs. I have also disabled the docker http proxy after deploy temporarily. None of which makes any difference.

Also: kube\_service\_addresses: 10.233.0.0/18 kube\_pods\_subnet: 10.233.64.0/18 (They do not overlap)

What is the next step in debugging this issue?