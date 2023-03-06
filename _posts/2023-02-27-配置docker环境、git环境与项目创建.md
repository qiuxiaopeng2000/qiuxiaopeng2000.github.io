---
layout: post
title: "配置docker环境与git环境"
date: 2023-02-27 
description: "将项目部署到云端服务器上，记录下配置docker环境与git环境需要用到的命令"
tags: [Docker, Git]
---   


以下操作均在Linux虚拟机中进行，作者使用的服务器是腾讯云的轻量应用服务器。


## 前置操作

### 免密远程登录服务器

我们首先要租一台云服务器。
1. 在本地Linux虚拟机中使用命令`ssh-keygen`生成ssh公钥和私钥，让服务器能识别你本地虚拟机的身份，使用`cd ~/.ssh`可以进入ssh文件夹查看生成的密钥。
2. 使用ssh远程登录你的服务器。使用`ssh name@ip`进行远程登录，其中name是你的服务器的名字即hostname，ip是你服务器的公有ip。一般租用的服务器都有初始的hostname，阿里云的默认是root，腾讯云的默认是ubuntu，也可自行修改hostname文件修改hostname`vim /etc/hostname`。比如作者将服务器的hostname设置成qxp，服务器的ip是1.15.93.63，则使用命令`ssh qxp@1.15.93.63`远程登录服务器。
3. root的权限过高，为了避免错误使用服务器而造成不可挽回后果，应当新建一个用户赋予部分权限，避免菜鸟因权限过高而造成损失。使用`usradd test`新建命名为test的用户，然后使用`passwd test`为用户设置密码。一般创建了用户后会在usr目录下为该用户建立一个文件夹，进入usr目录`cd /usr`，为test用户分配权限`chmod 777 -R test`。
   > 用户权限用三位bit位表示，第一位表示读，第二位表示行，第三位表示可执行，若可读则为100，转化为十进制就是4，7就表示该文件的权限为可读、可写、可执行。上述命令使用了三个7，第一个7是文件所有者的权限，第二个7是文件所属组的权限（同组成员的访问权限），第三个7表示其他用户的权限（游客的权限，一般游客只允许读，设置为4即可，这里为了方便都设置为7）。若需要更高的权限比如root权限，修改/etc/passwd即可，把用户名的ID和ID组修改成0（不建议）如需root权限自行百度
   > 给用户赋予root最高权限：直接修改/etc/passwd文件，将第三列的数字设置为0（可以参考root的值）即可
4. 配置config。使用`vim .ssh/config`命令新建config文件，写入配置如下：
   ```bash
    Host qxp # Host后写上你要登陆的服务器的hostname
        Hostname 1.15.93.63 # 写上要登陆的服务器的ip
        User test # 写上你要登录的用户身份，即上文创建的用户test
   ```
5. 使用命令`ssh-copy-id qxp`命令并通过密码验证即可成功设置免密登录，登录的用户身份及权限是User中填写的用户名，ssh-copy-id后填入的是你要免密登录的服务器的hostname。
6. 直接通过ssh登录到服务器中的容器：通过容器的端口进行映射
   ```bash
    Host qxp # Host后写上你要登陆的服务器的hostname
        Hostname 1.15.93.63 # 写上要登陆的服务器的ip
        User test # 写上你要登录的用户身份，即上文创建的用户test
        Port 20000
   ```

### 上传文件至服务器
`scp 文件名 qxp:`

scp后第一个变量为需要上传到服务器的本地文件，第二个变量为要上传到的服务器的hostname，由于已经配置了免密登录，可以直接传输文件，命令末尾不要忘记加`:`

## 配置docker环境

1. 启动dockers服务：`systemctl start docker`。启动docker命令需要root权限，最好是先使用root用户登录服务器并启动docker。
2. 为项目创建docker镜像：`docker load -i filename`，-i的含义是指定导入的文件，filename即要创建docker镜像的文件名。
3. 生成容器及端口映射：将dockers容器中的端口映射到服务器中，即连通docker容器与服务器 `docker run -p 20000:22 -p 8000:8000 --name django_server -itd django_lesson` 前面的端口是服务器的端口，后面的是容器的端口。22号端口是ssh应用端口，用于ssh登录容器，8000号端口用于测试用途，无特殊含义；name是为此次映射形成的容器的名称；-itd是一些配置参数，可自行百度；最后填上映射的docker镜像image。注意：容器一旦生成无法再修改容器映射。
> 记得在云服务器上开放相应的端口。
> 本人在使用腾讯云轻量应用服务器期间遇到过在控制台中打开端口仍然无法访问的情况，可以使用Linux命令手动打开端口。
> 首先执行`systemctl status firewall`查看防火墙是否打开，若防火墙关闭了可以使用`systemctl start firewall`打开防火墙，然后使用命令`firewall-cmd --zone=public --add-port=20000/tcp --permanent`打开端口，再使用`firewall-cmd --reload`加载设置，最终实现手动端口放行。可以使用命令`firewall-cmd --zone=public --list-ports`查看已经放行的端口，检查端口是否放行成功。
4. 进入容器命令：`docker attach image_name(container_name)`。关闭容器：Ctrl + D，挂起容器：先按Ctrl + P, 再按 Ctrl + Q
5. 使用ssh通过开放的端口直接登录容器，通过修改config配置，具体步骤参考[免密登录服务器](#免密远程登录服务器)第6小节。

## 使用git维护项目
任意找一个代码托管平台如[GitHub](https://github.com/)
1. 使用`git init`生成.git文件并初始化
2. 使用`git config --global user.name "用户名"`和`git config --global user.email "邮箱地址"`配置身份信息，只有配置了身份信息才能正常使用git
3. 使用`git remote add origin git@github.com:yourName/repositoryname.git`连接到远程仓库（初次连接时使用，下一次不需要重新建立连接）。
> 以上三步可以使用另一种方法代替。先在代码托管平台上建立一个代码仓库，再把该仓库克隆`git clone git@github.com:yourName/repositoryname.git`到本地，之后在本地项目上修改代码内容，使用下述方法进行提交、更新等管理。
4. 使用`git add .`把全部文件添加到暂存区，注意add后有一个空格。
5. 使用`git commit`将暂存区内容添加到仓库中，此时内容只记录在了.git文件中，还未正式提交。
6. 使用`git push`把要提交的内容正式提交到远程仓库
7. 使用`git pull`把远程仓库的内容更新到本地


