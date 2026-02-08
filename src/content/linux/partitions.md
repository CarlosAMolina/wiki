## Create partition

### fdisk

The following commands are a summary of the steps done when I installed a new operating system.

```bash
fdisk -l
fdisk /dev/sda
o
n
p
1
<none>
+3.6G
n
e
2
<none>
<none>
n
<none>
+0.5G
n
<none>
<none>
t
6
82
a
1
w
```

