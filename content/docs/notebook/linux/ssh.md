## ssh 相关密钥

sshd 服务在 `/etc/ssh` 目录下保存了一些密钥，制作系统模板的时候需要把这些删除掉，这些文件会在重启后重新生成。

`~/.ssh/` 写的文件也要都删除

```bash
ls /etc/ssh/
moduli      ssh_config.d  ssh_host_ecdsa_key      ssh_host_ed25519_key      ssh_host_rsa_key
ssh_config  sshd_config   ssh_host_ecdsa_key.pub  ssh_host_ed25519_key.pub  ssh_host_rsa_key.pub

ls ~/.ssh/
authorized_keys  id_rsa           id_rsa.pub       known_hosts
```

## SSH 主机密钥

SSH 通过公钥加密的方式保持通信安全。当某一 SSH 客户端连接到 SSH 服务器时在客户端登录之前，服务器会向其发送公钥副本。此密钥可以帮助设置通信通道的安全加密并可验证客户端系统的身份。

当用户使用 ssh 命令连接 SSH 服务器时该命令会检查其本地已知主机文件中是否有该服务器的公钥副本。该密钥可能己在 `/etc/ssh/ssh_known_hosts` 文件中进行了预配置，或者用户的主目录中可能具有包含该密钥的 `~/.ssh/known_hosts` 文件。

### SSH 已知主机密钥管理

有关已知远程系统及其密钥的信息存储在以下任一位置:

- 系统范围的 `/etc/ssh/ssh_known_hosts` 文件。
- 每个用户的主目录中的 `~/.ssh/known_hosts` 文件。

`/etc/ssh/ssh_known_hosts` 文件是系统范围的文件用于存储系统已知的主机的公钥。您必须手动或通过某种自动化方法(如使用 `Ansible` 或使用 `ssh-keyscan` 实用程序的脚本)来创建和管理此文件。

如果由于硬盘驱动器故障而导致公钥丢失或由于某些正当理由而导致公钥被更换，并因此更改了服务器的公钥必须修改 `/etc/ssh/ssh_known_hosts` 文件，将先前的公钥条目替换为新公钥条目。 

如果您连接到远程系统，并且该系统的公钥不在 `/etc/ssh/ssh_known_hosts` 文件中 SSH 客户端将在 ~/ssh/known_hosts 文件中搜索公钥。 

每个已知主机密钥条自包含一行，其中有三个字段：

- 第一个字段是共享该公钥的主机名和 IP 地址的列表
- 第二个字段是公钥的加密算法
- 最后一个字段是公钥本身

```bash
[root@localhost .ssh]# cat ~/.ssh/known_hosts
127.0.0.1 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBILFlIaJYN5vC594ohSC+Ptox9mH7zS3UGfHPiXxbAyl2qsT5TP8uKLXNHuKUM4WuLk/J/AoISjCMZt/do2oXs4=
```

当本地主机远程目标主机时，当前用户的 `~/.ssh/known_hosts` 文件会保存目标主机的 `/etc/ssh/ssh_host_ecdsa_key.pub` 内容，再次连接时，本地主机就会用 `~/.ssh/known_hosts` 的内容来进行检查，如果不匹配，就会认为不安全，断开连接

```bash
[root@localhost .ssh]# ssh 127.0.0.1
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:dmrX88zz3HZTGc5N0ODy1mqOaMNyq4zdgxKFr7+9Yu0.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /root/.ssh/known_hosts:1
ECDSA host key for 127.0.0.1 has changed and you have requested strict checking.
Host key verification failed.
```

Linux 系统默认设置为严格的主机密钥检查，即远程主机的密钥和本地存储的密钥副本不匹配的时候拒绝连接，如果想修改为允许存在不同的密钥副本，则需要修改 `StrictHostKeyChecking=` 参数，可以使用 `ssh -o StrictHostKeyChecking=no 127.0.0.1` 临时配置，如果想永久配置的需要修改 `/etc/ssh/ssh_config` 或 `~/.ssh/config` 文件，如果是  `/etc/ssh/ssh_config` 文件就去掉 `Host` 和 `StrictHostKeyChecking=` 的注释，将 `StrictHostKeyChecking=` 改为 no，`~/.ssh/config` 修改内容如下

```bash
Host *
  StrictHostKeyChecking=no
```

## 配置基于 SSH 密钥的身份验证

### 创建密钥

使用 `ssh-keygen` 命令创建一个密钥对，默认为 `~/.ssh/id_rsa` 和 `~/.ssh/id_rsa.pub` 文件中，使用 `-f` 参数可以指定文件名称和位置。

注意不要和已有的文件同名，会覆盖内容。

```bash
[root@localhost test]# ssh-keygen -f test
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in test.
Your public key has been saved in test.pub.
The key fingerprint is:
SHA256:7hiXavU6c91rZkh2A9bciY/aUc8zKlRnVERSQaPWP4w root@localhost.localdomain
The key's randomart image is:
+---[RSA 3072]----+
|              oBB|
|              oo.|
|             =oo.|
|            +o==o|
|        S  ..E*+o|
|       ... .ooo++|
|      ..+..+o+o.o|
|      .*o oooo=  |
|     .o o=  .+.. |
+----[SHA256]-----+
[root@localhost test]# ll
total 8
-rw-------. 1 root root 2610 Jan 30 15:29 test
-rw-r--r--. 1 root root  580 Jan 30 15:29 test.pub
```

### 共享公钥

使用 `ssh-copy-id` 命令可以将本地的公钥共享给其他主机，默认共享 `~/.ssh/id_rsa.pub` 公钥，可以使用 `-i` 参数共享其他公钥。

```bash
[root@localhost test]# ssh-copy-id -i test.pub 127.0.0.1
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "test.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '127.0.0.1'"
and check to make sure that only the key(s) you wanted were added.

```

### 使用密钥管理器进行非交互式身份验证

如果 ssh 密钥设置了密码，在每次身份连接的时候都会要求输入密码，使用 `ssh-agent` 可以将多个 ssh 密钥添加到密钥管理器，添加后再次连接的时候就不需要输入密钥密码了，每打开一个命令行都需要手动启动一个 `ssh-agent` 程序。

```bash
#启动一个 ssh-agent 进程
eval $(ssh-agent)

#添加密钥
[root@localhost test]# ssh-add /root/test/id_test
Enter passphrase for /root/test/id_test:
Identity added: /root/test/id_test (root@localhost.localdomain)
```

## 故障排查

当 SSH 连接出现故障时，如无法连接到远程主机，可以使用 `-v`、`-vv`、`-vvv` 等参数 debug 信息进行排查。

## 客户端配置

用户可以创建 `~/.ssh/config` 文件来配置 SSH 连接。在配置文件中，可以为特定主机指定连接参数，如连接用户、密钥和端口等等，一下是一个配置文件的例子。

```bash
Host servera
  StrictHostKeyChecking=no
  Hostname servera.example.com
  Port 2222
  User student
  IdentityFile /root/test/id_test
```

## 配置 root 相关登陆验证

### 禁止 root 登录

修改 `/etc/ssh/sshd_config` 的 `PermitRootLogin` 参数为 `no`

### 配置只允许密钥登陆 root

修改 `/etc/ssh/sshd_config` 的 `PermitRootLogin` 参数为 `without-password` or `prohibit-password`

### 配置禁止密码登录系统

修改 `/etc/ssh/sshd_config` 的 `PasswordAuthentication` 参数为 `no`