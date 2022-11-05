GitLab

 ####　１．配置镜像源

```shell
echo -e '
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
' >> /etc/yum.repos.d/gitlab-ce.repo
```

####　２．yum 安装gitlab

```shell
yum install gitlab-ce     # 自动安装最新版

yum install gitlab-ce-11.7.5    # 安装指定版本

# 可在镜像源上查看版本：比如 https://mirrors.tuna.tsinghua.edu.cn/
```

#### 3. 配置 gitlab

##### 3.1 配置文件目录

```shell
/etc/gitlab/gitlab.rb；
```

##### 3.2 配置访问路径及接口

external_url 'http://gitlab.example.com'

默认为上面路径，改为 http://192.168.15.148:8888

可以修改为域名访问，需配置nginx，在gitlab.rb中配置nginx监听端口；

nginx['listen_port'] = nil，将此行放开，nil修改为要监听的端口，再配置nginx即可通过域名访问；

##### 3.3 配置项目存储路径

```shell
git_data_dirs({
   "default" => {
     "path" => "/mnt/nfs-01/git-data"
    }
 })
 # 将path中的路径修改为想要使用的路径，此处修改为，/data/env/gitlab
```

#### 4. 启动gitlab

##### 4.1 重置配置文件，使配置生效

gitlab-ctl reconfigure

这步需要较长时间。

卸载再安装gitlab后会出现这种问题，卡在ruby_block[supervise_redis_sleep] action run；

这时需要强制退出安装，运行：

```shell
sudo systemctl restart gitlab-runsvdir
```

再次进行安装：

```shell
sudo gitlab-ctl reconfigure
```

解决方法参考： https://gitlab.com/gitlab-org/omnibus-gitlab/issues/160

出现 gitlab Reconfigured! 表示配置完成；

##### 4.2 操作命令

```sh
gitlab-ctl restart # 重启

#其他操作命令：
sudo gitlab-ctl status # 查看服务的状态
sudo gitlab-ctl start   # 启动
sudo gitlab-ctl stop   #关闭
sudo gitlab-ctl restart  #重启
```

#### 5. 访问

##### 5.1 配置端口

```shel
gitlab.rb #配置文件中配置的端口是8888； 对外放开此端口并重启防火墙；
firewall-cmd --zone=public --add-port=8888/tcp--permanent
firewalld is not running #需要开启防火墙
```

##### 5.2 访问路径

请求路径为gitlab.rb中配置的路径；

第一次进入需要为root用户配置密码，此处设置为315che@poly，长度要求大于8位；

#### 6. 控制面板

##### 6.1 创建项目群组

用于不同项目组操作不同项目；

公司项目，权限必须设置为私有。在外部服务器通过域名访问，如果设置public很容易泄露代码；

##### 6.2 添加测试开发人员

管理员可将自行注册或由管理员创建的开发人员账号加入项目组中；

在Group页面中，点击右侧Members进行人员添加；

在搜索栏即可看到刚才添加的测试账号；可修改账号的权限；

##### 6.3 创建空项目

此处不能使用README，不然会在上传项目的时候出现不能上传的问题；

在创建的新项目右侧，clone选择http连接进行项目上传。没有做ssh秘钥操作。

IDEA中项目上传到gitlab参考 https://blog.csdn.net/chenpuzhen/article/details/80766250

#### 7. GitLab操作

##### 7.1 停止gitlab

```shell
sudo gitlab-ctl stop
```

##### 7.2 卸载gitlab

```shell
sudo rpm -e gitlab-ce
```

##### 7.3 查看gitlab进程

```shell
ps -ef|grep gitlab
# 杀掉第一个守护进程(runsvdir -P /opt/gitlab/service log)
kill -9 4473
```

##### 7.4 删除gitlab文件

```shell
 find / -name *gitlab*|xargs rm -rf      # 删除所有包含gitlab的文件及目录
 find / -name gitlab |xargs rm -rf 
```

删除gitlab-ctl uninstall时自动在root下备份的配置文件（ls /root/gitlab* 看看有没有，有也删除）

#### 8. Gitlab的备份与迁移

参考： https://blog.csdn.net/ouyang_peng/article/details/77070977

