### redhat 上 supervisor 的安装和使用

##### 1. 安装

可以通过pip 安装,
这种方式安装对python的版本要求比较严格, 我在写文档的时候, supervisor==3.3.5, Supervisor is intended to work on Python 3 version 3.4 or later and on Python 2 version 2.7.

```
pip install supervisor
```

如果是内网机没有网的话可以去[Github仓库](https://github.com/Supervisor/supervisor), clone 一份源码来安装.

```
python setup.py install 
```

安装过程中可能会报缺少`meld3` python 安装包, 安装之后就可以正常supervisor了

#### 2. 使用

如果系统上有多个版本的python, 直接运行 `supervisord` 会显示 `command not found`, 这是因为supervisor 安装在某个版本的python 目录下了, 需要手动找到, 创建软链接.

```
find / -name supervisord
//2.7 显示是
/usr/local/python2.7/bin/supervisord
//3.7 显示是
/root/.pyenv/versions/venv370/bin/supervisord

//创建链接
ln -s /usr/local/python2.7/bin/supervisord  /usr/local/bin/supervisord
//或者:
ln -s /root/.pyenv/versions/venv370/bin/supervisord  /usr/bin/supervisord

//同理:
ln -s /usr/local/python2.7/bin/supervisorctl  /usr/local/bin/supervisorctl
//或者:
ln -s /root/.pyenv/versions/venv370/bin/supervisorctl  /usr/bin/supervisorctl

ln -s /usr/local/python2.7/bin/echo_supervisord_conf  /usr/local/bin/echo_supervisord_conf
//或者:
ln -s /root/.pyenv/versions/venv370/bin/echo_supervisord_conf  /usr/bin/echo_supervisord_conf
```

现在可以使用了.

* 配置

    创建配置文件, 根据官网, 我将目录创建在/etc目录下
    
    ```
    echo_supervisord_conf > /etc/supervisord.conf
    ```
    打开文件, 在文件末尾追加:
    
    ```
    vi /etc/supervisord.conf
    
    //我这里监控一个我的python3.7 的uwsgi进程, 这个进程用来启动我的django 项目
    
    [program:dbyw-python3]
    directory=/dbtool/python_software/dbyw-python3
    command=uwsgi --ini uwsgi.ini
    autostart=true
    stdout_logfile=/etc/supervisor.d/log/dbyw3_stdout.log
    stderr_logfile=/etc/supervisor.d/log/dbyw3_stderr.log
        
    //监控python celery
    [program:dbyw-python3-celery]
    directory=/dbtool/python_software/dbyw-python3
    command=celery -A celery_tasks worker --loglevel=info --logfile celery_beat.log
    autostart=true
    stdout_logfile=/etc/supervisor.d/log/dbyw3_celery_stdout.log
    stderr_logfile=/etc/supervisor.d/log/dbyw3_celery_stderr.log
    
    
    //监控zookeeper
    [program:zookeeper]
    command=/dbtool/python_software/kafka/bin/zookeeper-server-start.sh config/zookeeper.properties
    autostart=true
    stdout_logfile=/etc/supervisor.d/log/zookeeper_stdout.log
    stderr_logfile=/etc/supervisor.d/log/zookeeper_stderr.log
    
    
    //监控kafka
    [program:kafka]
    command=/dbtool/python_software/kafka/bin/kafka-server-start.sh config/server.properties
    autostart=true
    stdout_logfile=/etc/supervisor.d/log/kafka_stdout.log
    stderr_logfile=/etc/supervisor.d/log/kafka_stderr.log
    
    ```
    
    创建日志目录
    
    ```
    mkdir -p /etc/supervisor.d/log
    ```

* 启动

    运行一下命令可以测试是否配置成功
    
    ```
    supervisord -c /etc/supervisord.conf
    ```
    之后可以查看要监控的进程是否已经启动
    
    ```
    supervisorctl
    ```
    
##### 3. supervisord 命令行操作

* 启动supervisord进程

    ```
    supervisord -c /etc/supervisord.conf
    ```
* 查看进程状态

    ```
    supervisorctl
    ```

* 更多supervisorctl命令

    ```
    supervisorctl status
    supervisorctl stop all
    supervisorctl stop dbyw-python3
    supervisorctl start dbyw-python3
    supervisorctl restart dbyw-python3
    supervisorctl reload
    supervisorctl update
    ```
    
##### 4. 设置supervisor 开启自启

```
vi /etc/rc.local

//添加
/usr/local/bin/supervisord -c /etc/supervisord.conf
```



