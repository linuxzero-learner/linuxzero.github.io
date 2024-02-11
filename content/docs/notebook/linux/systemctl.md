## systemctl常用选项

```bash
#列出所有服务
systemctl list-units

#列出指定类型服务单元状态
systemctl list-units --type help
systemctl list-units --type service
systemctl list-units --state not-found --all
UNIT：服务单元名称。
LOAD：systemd 守护进程是否正确解析了单元的配置并将该单元加载至内存中。
ACTIVE：单元的高级别激活状态。此项表明单元是否已成功启动。
SUB：单元的低级别激活状态。此信息指示有关该单元的更多详细信息。信息视单元类型、状态以及单元的执行方式而异。
DESCRIPTION：单元的简短描述。
默认情况下， systemctl list-units --type=service 命令仅列出激活状态为 active 的服务单元。 
systemctl list-units --all 选项可列出出所有服务单元不论激活状态如何。
使用 --state= 选项可按照 LOAD ACTIVE SUB 字段中的值进行筛选。

systemctl 命令 list-units 选项可显示 systemd 服务尝试解析并加载到内存中的单元。此选项不显示己安装但未启用的服务。
您可以使用 systemctl 命令 list-unit-files 选项来查看所有己安装的单元文件的状态。

#列出所有已安装的单元文件状态
systemctl list-unit-files
systemctl list-unit-files --type target

#查看服务状态
systemctl status sshd

#查看服务是否开始自启动
systemctl is-enabled sshd

#查看服务是否启动
systemctl is-active sshd

#查看服务是否在启动过程中失败
systemctl is-failed sshd

如果单元运行正常，该命令将返回 active ，如果启动过程中发生了错误，则返回 failed。如果单元己被停止它将返回 unknown、inactive。
要列出所有失败的单元，请运行 systemctl --failed --type=service 命令。
```

## 服务单元信息

|   字段   |                    描述                    |
| :------: | :----------------------------------------: |
|  Loaded  |         服务单元是否已加载到内存中         |
|  Active  | 服务单元是否正在运行；若为是，它运行了多久 |
|   Docs   |      哪里可以找到有关该服务的更多信息      |
| Main PID |         服务主进程ID，包括命令名称         |
|  Status  |               服务的更多信息               |
| Process  |             相关进程的更多信息             |
|  CGroup  |            相关控制组的更多信息            |

## systemctl 输出中的服务状态

|     关键字      |                     描述                     |
| :-------------: | :------------------------------------------: |
|     loaded      |             单元配置文件已被处理             |
| active(running) |         服务正在运行，并有持续的进程         |
| active(exited)  |           服务成功完成了一次性配置           |
| active(waiting) |         服务正在运行中，但在等待时间         |
|    inactive     |                 服务未在运行                 |
|     enabled     |             服务在系统引导时启动             |
|    disabled     |         服务未设置为在系统引导时启动         |
|     static      | 服务无法启用，但可以由某一启用的单元自动启动 |

## 服务管理实用命令

|          命令          |                            任务                             |
| :--------------------: | :---------------------------------------------------------: |
| systemctl status UNIT  |                   查看单元状态的详细信息                    |
|  systemctl stop UNIT   |                在运行中的系统上停止一项服务                 |
|  systemctl start UNIT  |                在运行中的系统上启动一项服务                 |
| systemctl restart UNIT |              在运行中的系统上重新启动一项服务               |
| systemctl reload UNIT  |                重新加载运行中服务的配置文件                 |
|  systemctl mask UNIT   |        禁用服务，使其无法手动启动或在系统引导时启动         |
| systemctl unmask UNIT  |                    使屏蔽的服务变为可用                     |
| systemctl enable UNIT  | 将服务配置为在系统引导时启动。使用 --now 选项也会停止该服务 |
| systemctl disable UNIT |   禁止服务在系统引导时启动。使用 --now 选项也会停止该服务   |

