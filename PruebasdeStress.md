### Realizar pruebas de estres sobre servidores CentOS / RHEL

# Instalar los repos necesarios

CentOS
```
yum install epel-release
```
RHEL
```
yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

# Intalar las herramienta stress
```
yum install stress -y
```

# Validar el numero de procesadores 
```
[root@node3 ~]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
**CPU(s):                2**
On-line CPU(s) list:   0,1
Thread(s) per core:    1
Core(s) per socket:    2
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 79
Model name:            Intel(R) Xeon(R) CPU E5-2686 v4 @ 2.30GHz
Stepping:              1
CPU MHz:               2300.190
BogoMIPS:              4600.14
Hypervisor vendor:     Xen
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              46080K
NUMA node0 CPU(s):     0,1
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology eagerfpu pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm fsgsbase bmi1 avx2 smep bmi2 erms invpcid xsaveopt
```

##Estresar la maquina a nivel de CPU
el numero 2 es por el numero de vcpus de la maquina

```
[root@node3 ~]# stress -c 2
stress: info: [31097] dispatching hogs: 2 cpu, 0 io, 0 vm, 0 hdd
```

# Monitoreo

##Estresar la maquina a nivel de Memoria
Primero necesitamos definir cuánto llenaremos la memoria. Si tiene 4Gb, puede llenar 1024 M * 4 asi:
```
[root@node3 ~]# stress --vm 4 --vm-bytes 1024M
stress: info: [693] dispatching hogs: 0 cpu, 0 io, 4 vm, 0 hdd
```


# Monitoreo
Mienstra se realiza las pruebas se pueden utilizar comandos como:
```
- vmstat
- top
- tload * paquete procps-ng
- uptime
```
