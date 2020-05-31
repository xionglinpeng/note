# 1



tree



```shell
$ yum install -y tree
```





```shell
$ tree --help
usage: tree [-acdfghilnpqrstuvxACDFQNSUX] [-H baseHREF] [-T title ] [-L level [-R]]
	[-P pattern] [-I pattern] [-o filename] [--version] [--help] [--inodes]
	[--device] [--noreport] [--nolinks] [--dirsfirst] [--charset charset]
	[--filelimit[=]#] [--si] [--timefmt[=]<f>] [<directory list>]
  ------- Listing options -------
  -a            All files are listed.
  -d            List directories only.
  -l            Follow symbolic links like directories.
  -f            Print the full path prefix for each file.
  -x            Stay on current filesystem only.
  -L level      Descend only level directories deep.
  -R            Rerun tree when max dir level reached.
  -P pattern    List only those files that match the pattern given.
  -I pattern    Do not list files that match the given pattern.
  --noreport    Turn off file/directory count at end of tree listing.
  --charset X   Use charset X for terminal/HTML and indentation line output.
  --filelimit # Do not descend dirs with more than # files in them.
  --timefmt <f> Print and format time according to the format <f>.
  -o filename   Output to file instead of stdout.
  --du          Print directory sizes.
  --prune       Prune empty directories from the output.
  -------- File options ---------
  -q            Print non-printable characters as '?'.
  -N            Print non-printable characters as is.
  -Q            Quote filenames with double quotes.
  -p            Print the protections for each file.
  -u            Displays file owner or UID number.
  -g            Displays file group owner or GID number.
  -s            Print the size in bytes of each file.
  -h            Print the size in a more human readable way.
  --si          Like -h, but use in SI units (powers of 1000).
  -D            Print the date of last modification or (-c) status change.
  -F            Appends '/', '=', '*', '@', '|' or '>' as per ls -F.
  --inodes      Print inode number of each file.
  --device      Print device ID number to which each file belongs.
  ------- Sorting options -------
  -v            Sort files alphanumerically by version.
  -r            Sort files in reverse alphanumeric order.
  -t            Sort files by last modification time.
  -c            Sort files by last status change time.
  -U            Leave files unsorted.
  --dirsfirst   List directories before files (-U disables).
  ------- Graphics options ------
  -i            Don't print indentation lines.
  -A            Print ANSI lines graphic indentation lines.
  -S            Print with ASCII graphics indentation lines.
  -n            Turn colorization off always (-C overrides).
  -C            Turn colorization on always.
  ------- XML/HTML options -------
  -X            Prints out an XML representation of the tree.
  -H baseHREF   Prints out HTML format with baseHREF as top directory.
  -T string     Replace the default HTML title and H1 header with string.
  --nolinks     Turn off hyperlinks in HTML output.
  ---- Miscellaneous options ----
  --version     Print version and exit.
  --help        Print usage and this help message and exit.
```













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

