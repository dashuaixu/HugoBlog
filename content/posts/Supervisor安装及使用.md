+++
title = 'Supervisor安装及使用'
date = 2024-02-28T22:26:32+08:00
draft = false
+++

#### 安装

centos：` yum install supervisor`

pip：`pip install supervisor`

---

#### 配置

配置文件路径：`/etc/supervisord.conf `

自定义配置文件放置路径：`/etc/supervisord.d/`

将自定义的配置文件(以`.ini`结尾的文件)放入自定义配置路径下，启动时会读取

---

#### 使用

启动：`supervisord -c /etc/supervisord.conf`

控制台：`supervisorctl -c /etc/supervisord.conf`

```shell
[root@VM_0_48_centos etc]# supervisord -c /etc/supervisord.conf # 先启动才能使用
[root@VM_0_48_centos etc]# supervisorctl -c /etc/supervisord.conf
supervisor> status
supervisor> 
```

控制台命令：

```shell
[root@spider2 ~]# supervisorctl -c /etc/supervisord.conf
Jyeoo_spider                     RUNNING   pid 5086, uptime 0:48:15
supervisor> 
supervisor> status
Jyeoo_spider                     RUNNING   pid 5086, uptime 0:48:21
supervisor> 


> status    # 查看程序状态
> stop usercenter   # 关闭 usercenter 程序
> start usercenter  # 启动 usercenter 程序
> restart usercenter    # 重启 usercenter 程序
> reread    ＃ 读取有更新（增加）的配置文件，不会启动新添加的程序
> update    ＃ 重启配置文件修改过的程序
> quit    # 退出
```

---

#### 配置文件样例

```shell
[program:Jyeoo_spider] 
command=scrapy crawl jyeoo -s JOBDIR=job_process/001 
directory=/root/jyeoo_spider/baidu_tiku/baidu_tiku/spiders  
environment=ASPNETCORE__ENVIRONMENT=Production
user=root 
stopsignal=INT 
autostart=true 
autorestart=true 
startsecs=3
stderr_logfile=/root/jyeoo_spider.err.log 
stdout_logfile=/root/jyeoo_spider.out.log 
```



```shell
[program:usercenter]
directory = /home/leon/projects/usercenter ; 程序的启动目录
command = gunicorn -c gunicorn.py wsgi:app  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = leon          ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /data/logs/usercenter_stdout.log
 
; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```

---

#### 常见错误

```shell
Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.
For help, use /usr/bin/supervisord -h
```

解决：`sudo unlink /run/supervisor/supervisor.sock `

```shell
[root@spider2 baidu_tiku]# supervisorctl -c /etc/supervisord.conf
Jyeoo_spider                     FATAL     Exited too quickly (process log may have details)
supervisor> start Jyeoo_spider
Jyeoo_spider: ERROR (abnormal termination)
```

解决：`ps -ef |grep super` kill掉supervisor相关进程 然后重启