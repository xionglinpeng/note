# 1



```shell
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' catalogue-db-dep.yaml
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' carts-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' catalogue-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' front-end-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' loadtest-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' orders-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' orders-db-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' payment-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' queue-master-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' rabbitmq-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' session-db-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' shipping-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' user-db-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' user-dep.yaml 
[root@localhost manifests]# sed -i 's/extensions\/v1beta1/apps\/v1/g' carts-db-dep.yaml
```







```shell
spec:
  selector:
    matchLabels:
      app: my-nginx
  replicas: 1
  template:
    metadata:
      labels:
        name: user-db
    spec:
      containers:
```







Linux查看CPU信息





```shell
[root@work1 ~]# cat /proc/cpuinfo 
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 94
model name	: Intel(R) Core(TM) i7-6700HQ CPU @ 2.60GHz
stepping	: 3
cpu MHz		: 2591.998
cache size	: 6144 KB
physical id	: 0
siblings	: 2
core id		: 0
cpu cores	: 2
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 22
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc eagerfpu pni pclmulqdq ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single fsgsbase avx2 invpcid rdseed clflushopt flush_l1d
bogomips	: 5183.99
clflush size	: 64
cache_alignment	: 64
address sizes	: 39 bits physical, 48 bits virtual
power management:

processor	: 1
vendor_id	: GenuineIntel
cpu family	: 6
model		: 94
model name	: Intel(R) Core(TM) i7-6700HQ CPU @ 2.60GHz
stepping	: 3
cpu MHz		: 2591.998
cache size	: 6144 KB
physical id	: 0
siblings	: 2
core id		: 1
cpu cores	: 2
apicid		: 1
initial apicid	: 1
fpu		: yes
fpu_exception	: yes
cpuid level	: 22
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc eagerfpu pni pclmulqdq ssse3 cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx rdrand hypervisor lahf_lm abm 3dnowprefetch invpcid_single fsgsbase avx2 invpcid rdseed clflushopt flush_l1d
bogomips	: 5183.99
clflush size	: 64
cache_alignment	: 64
address sizes	: 39 bits physical, 48 bits virtual
power management:
```

