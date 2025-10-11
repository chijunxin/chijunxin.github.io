# Windows平台docker修改容器的端口映射配置



<!--more-->

## 找到`containers`
* 键盘的WIN+R后输入：\\wsl$
* 进入docker-desktop
* 在右侧直接搜索 containers （需要先启动docker-desktop，不然找到的目录是错误的）
![](/images/windows平台docker修改容器的端口映射配置/1.png)

* 这里可能找到多个同名文件夹，右键在新窗口打开，如下图中文件夹名字都是由64个数字和字母组成。
![](/images/windows平台docker修改容器的端口映射配置/2.png)

## 找到对应的container的Id
### 方法一
* 通过docker.desktop，在对应容器中查看,图中只显示了Id的一部分，通常对比这部分已经能找到对应的文件夹。
![](/images/windows平台docker修改容器的端口映射配置/3.png)

### 方法二
* 键盘的WIN+R后输入：cmd
* docker inspect 容器名字
* 找到其中的Id  
![](/images/windows平台docker修改容器的端口映射配置/4.png)

## 修改两个配置
### 进入对应的文件夹
![](/images/windows平台docker修改容器的端口映射配置/5.png)

### config.v2.json
"ExposedPorts": {"8080/tcp": {},增加需要的端口}

### hostconfig.json
"PortBindings":{"8080/tcp":[{"HostIp":"","HostPort":"8081"}],增加需要的端口}

{{< admonition tip "Tips" >}}
建议格式化之后再修改
{{< /admonition >}}

## 重启docker
{{< admonition warning "注意" >}}
注意是重启docker，不是重启单个容器
{{< /admonition >}}
