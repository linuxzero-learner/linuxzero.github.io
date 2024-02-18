## 安装 VirtualBMC

```bash
#安装 python
dnf install -y python3

#安装virtualbmc
pip3 install virtualbmc

#中间会有报错，安装 libvirt-devel 和 python3-devel
dnf install -y libvirt-devel python3-devel
```

## 配置 VirtualBMC

### 启动 vbmcd 服务

```bash
#前台显示，这个可以在写服务文件的时候用
vbmcd --foreground

#后台执行
vbmcd
```

### 检查 vbmcd 服务

```bash
[root@foreman ~]# ss -ntlp | grep vbmcd
LISTEN 0      100             127.0.0.1:50891      0.0.0.0:*    users:(("vbmcd",pid=31291,fd=15),("vbmcd",pid=29619,fd=15))
```

### 添加虚拟机

`qemu+ssh://root@foreman/system` 表示远程的 libvirtd 服务，这个可以用 `virt-manager` 先远程连接一下，然后就能在 `virt-manager` 上看到远程 libvirtd 的 URI，我这的这个是 `qemu+ssh://root@foreman/system`。
`username` 表示 ipmitool 连接时使用的用户名
`password` 表示 ipmitool 连接时使用的密码
`192.168.144.254` 表示 ipmitool 连接的地址，这个得是 virtualbmc 服务器上实际存在的地址
`6666` 表示 ipmitool 连接的端口，这个得是 virtualbmc 服务器上未使用的端口

```bash
#将虚拟机添加到 vbmcd 中
vbmc add --libvirt-uri qemu+ssh://root@foreman/system --username root --password redhat --port 6666 --address 192.168.144.254 rhel8.3
```

### 检查虚拟机 vbmc 配置

```bash
[root@foreman ~]# vbmc list
+-------------+--------+-----------------+------+
| Domain name | Status | Address         | Port |
+-------------+--------+-----------------+------+
| rhel8.3     | down   | 192.168.144.254 | 6666 |
+-------------+--------+-----------------+------+

[root@foreman ~]# ss -nulp | grep 6666
UNCONN 0      0      192.168.144.254:6666      0.0.0.0:*    users:(("vbmcd",pid=31291,fd=18))

[root@foreman ~]# ps -ef | grep vbmc
root       41578    2064  5 20:00 pts/0    00:00:00 /usr/bin/python3.6 /usr/local/bin/vbmcd --foreground
root       41580   41578  0 20:00 pts/0    00:00:00 /usr/bin/python3.6 /usr/local/bin/vbmcd --foreground
root       41585   29636  0 20:01 pts/1    00:00:00 grep --color=auto vbmc
```

### 详细显示虚拟机 vbmc 配置

```bash
[root@foreman ~]# vbmc show rhel8.3
+-----------------------+--------------------------------+
| Property              | Value                          |
+-----------------------+--------------------------------+
| active                | False                          |
| address               | 192.168.144.254                |
| domain_name           | rhel8.3                        |
| libvirt_sasl_password | ***                            |
| libvirt_sasl_username | None                           |
| libvirt_uri           | qemu+ssh://root@foreman/system |
| password              | ***                            |
| port                  | 6666                           |
| status                | down                           |
| username              | root                           |
+-----------------------+--------------------------------+
```

### 启动和关闭 vbmc

这个启动和关闭指的是启动和关闭虚拟机的 vbmc 功能（是否允许通过 ipmitool 控制虚拟机），并不是虚拟机的启动和关闭。

下边的 `down` 就表示关闭 vbmc 功能，`running` 表示开启 vbmc 功能

```bash
[root@foreman ~]# vbmc list
+-------------+--------+-----------------+------+
| Domain name | Status | Address         | Port |
+-------------+--------+-----------------+------+
| rhel8.3     | down   | 192.168.144.254 | 6666 |
+-------------+--------+-----------------+------+
[root@foreman ~]# vbmc start rhel8.3
[root@foreman ~]# vbmc list
+-------------+---------+-----------------+------+
| Domain name | Status  | Address         | Port |
+-------------+---------+-----------------+------+
| rhel8.3     | running | 192.168.144.254 | 6666 |
+-------------+---------+-----------------+------+
[root@foreman ~]# vbmc show rhel8.3
+-----------------------+--------------------------------+
| Property              | Value                          |
+-----------------------+--------------------------------+
| active                | True                           |
| address               | 192.168.144.254                |
| domain_name           | rhel8.3                        |
| libvirt_sasl_password | ***                            |
| libvirt_sasl_username | None                           |
| libvirt_uri           | qemu+ssh://root@foreman/system |
| password              | ***                            |
| port                  | 6666                           |
| status                | running                        |
| username              | root                           |
+-----------------------+--------------------------------+
```

### 使用 ipmitool 控制虚拟机开关机

下边是用 ipmitool 来开关虚拟机的命令，我现在能想到的就是用这个给虚拟机做电源管理，下边还测试了 关闭 vbmc 功能后再使用 ipmitool 的效果。

```bash
[root@foreman ~]# ipmitool -I lanplus -H 192.168.144.254 -U root -P redhat -p 6666 power status
Chassis Power is off
[root@foreman ~]# virsh list
 Id   Name   State
--------------------

[root@foreman ~]# virsh list --all
 Id   Name      State
--------------------------
 -    rhel8.3   shut off

[root@foreman ~]# ipmitool -I lanplus -H 192.168.144.254 -U root -P redhat -p 6666 power up
Chassis Power Control: Up/On
[root@foreman ~]# virsh list --all
 Id   Name      State
-------------------------
 5    rhel8.3   running

[root@foreman ~]# ipmitool -I lanplus -H 192.168.144.254 -U root -P redhat -p 6666 power down
Chassis Power Control: Down/Off
[root@foreman ~]# virsh list --all
 Id   Name      State
--------------------------
 -    rhel8.3   shut off

[root@foreman ~]# vbmc list
+-------------+---------+-----------------+------+
| Domain name | Status  | Address         | Port |
+-------------+---------+-----------------+------+
| rhel8.3     | running | 192.168.144.254 | 6666 |
+-------------+---------+-----------------+------+
[root@foreman ~]# vbmc stop rhel8.3
[root@foreman ~]# vbmc list
+-------------+--------+-----------------+------+
| Domain name | Status | Address         | Port |
+-------------+--------+-----------------+------+
| rhel8.3     | down   | 192.168.144.254 | 6666 |
+-------------+--------+-----------------+------+
[root@foreman ~]# ipmitool -I lanplus -H 192.168.144.254 -U root -P redhat -p 6666 power status
Error: Unable to establish IPMI v2 / RMCP+ session
[root@foreman ~]# vbmc start rhel8.3
[root@foreman ~]# vbmc list
+-------------+---------+-----------------+------+
| Domain name | Status  | Address         | Port |
+-------------+---------+-----------------+------+
| rhel8.3     | running | 192.168.144.254 | 6666 |
+-------------+---------+-----------------+------+
[root@foreman ~]# ipmitool -I lanplus -H 192.168.144.254 -U root -P redhat -p 6666 power status
Chassis Power is off
```

可以看到 ipmitool 已经无法建立会话了，但是开启 vbmc 功能后就可以了。

## troubleshooting

### pip 安装 virtualbmc 报错

报错内容如下

```bash
    Complete output from command python setup.py egg_info:
    Package libvirt was not found in the pkg-config search path.
    Perhaps you should add the directory containing `libvirt.pc'
    to the PKG_CONFIG_PATH environment variable
    Package 'libvirt', required by 'virtual:world', not found
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-fn0i50gu/libvirt-python/setup.py", line 270, in <module>
        _c_modules, _py_modules = get_module_lists()
      File "/tmp/pip-build-fn0i50gu/libvirt-python/setup.py", line 87, in get_module_lists
        ldflags = get_pkgconfig_data(["--libs-only-L"], "libvirt", False).split()
      File "/tmp/pip-build-fn0i50gu/libvirt-python/setup.py", line 54, in get_pkgconfig_data
        output = subprocess.check_output(cmd, universal_newlines=True)
      File "/usr/lib64/python3.6/subprocess.py", line 356, in check_output
        **kwargs).stdout
      File "/usr/lib64/python3.6/subprocess.py", line 438, in run
        output=stdout, stderr=stderr)
    subprocess.CalledProcessError: Command '['pkg-config', '--libs-only-L', 'libvirt']' returned non-zero exit status 1.

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-fn0i50gu/libvirt-python/
```

安装 libvirt-devel

```bash
dnf install -y libvirt-devel
```

### Upgrade to the latest pip and try again

pip 安装 virtualbmc 报错

```bash
    Complete output from command python setup.py egg_info:

            =============================DEBUG ASSISTANCE==========================
            If you are seeing an error here please try the following to
            successfully install cryptography:

            Upgrade to the latest pip and try again. This will fix errors for most
            users. See: https://pip.pypa.io/en/stable/installing/#upgrading-pip
            =============================DEBUG ASSISTANCE==========================

    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-ffaqbh3b/cryptography/setup.py", line 18, in <module>
        from setuptools_rust import RustExtension
    ModuleNotFoundError: No module named 'setuptools_rust'

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-build-ffaqbh3b/cryptography/
```

更新 pip

```bash
python3 -m pip install --upgrade pip
```

### ERROR: Failed building wheel for libvirt-python

报错

```bash
  ERROR: Command errored out with exit status 1:
   command: /usr/bin/python3.6 /usr/local/lib/python3.6/site-packages/pip/_vendor/pep517/in_process/_in_process.py build_wheel /tmp/tmpx0dz4c5z
       cwd: /tmp/pip-install-ikob8qxl/libvirt-python_aecd553a6cff40daadfb30c96d70f786
  Complete output (17 lines):
  running bdist_wheel
  running build
  running build_py
  creating build/lib.linux-x86_64-3.6
  copying build/libvirt.py -> build/lib.linux-x86_64-3.6
  copying build/libvirt_qemu.py -> build/lib.linux-x86_64-3.6
  copying build/libvirt_lxc.py -> build/lib.linux-x86_64-3.6
  copying build/libvirtaio.py -> build/lib.linux-x86_64-3.6
  running build_ext
  Generated 280 wrapper functions
  Generated 1 wrapper functions
  Generated 1 wrapper functions
  building 'libvirtmod' extension
  creating build/temp.linux-x86_64-3.6
  creating build/temp.linux-x86_64-3.6/build
  gcc -pthread -Wno-unused-result -Wsign-compare -DDYNAMIC_ANNOTATIONS_ENABLED=1 -DNDEBUG -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -D_GNU_SOURCE -fPIC -fwrapv -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -D_GNU_SOURCE -fPIC -fwrapv -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -D_GNU_SOURCE -fPIC -fwrapv -fPIC -I. -I/usr/include/python3.6m -c libvirt-override.c -o build/temp.linux-x86_64-3.6/libvirt-override.o -Wp,-DPy_LIMITED_API=0x03060000
  error: command 'gcc' failed with exit status 1
  ----------------------------------------
  ERROR: Failed building wheel for libvirt-python
Failed to build libvirt-python
ERROR: Could not build wheels for libvirt-python, which is required to install pyproject.toml-based projects
```

安装 pyproject

```bash
dnf install -y python3-devel
```

