# 问题：服务器上WhatsApp登录二维码显示不出

1、**基本情况**：多次按照文档执行命令，在配置时WhatsApp配置二维码登录二维码显示不出，在询问豆包等大模型后仍未解决

2、**服务器信息**：华为云服务器2vCPUs | 2GiB | x1.2u.2gUbuntu 22.04 server 64bit Linux  华东-上海一

3、**问题查询方式**：主要是豆包大模型，辅助腾讯混元

4、**操作平台**：windows的命令行远程登录服务器操作

5、**问题描述**

在执行`curl -fsSL https://molt.bot/install.sh | bash`后出现

```  bash
Link WhatsApp now (QR)?
│  Yes
(node:72663) Warning: Setting the NODE_TLS_REJECT_UNAUTHORIZED environment variable to '0' makes TLS connections and HTTPS requests insecure by disabling certificate verification.
(Use `node --trace-warnings ...` to show where the warning was created)
Waiting for WhatsApp connection...
WhatsApp Web connection ended before fully opening. status=408 Request Time-out WebSocket Error ()
WhatsApp login failed: Error: status=408 Request Time-out WebSocket Error ()
```

<img width="1223" height="700" alt="image" src="https://github.com/user-attachments/assets/5a3c3da9-ed79-4ad9-9c01-c2decdaba71f" />




6、**问题分析：**疑似国内服务器连接不上外网的网络问题导致

7、**尝试解决方法**（没有解决问题，这里写的是尝试中可能有效的方法，无效的未写）：

+ 方法一：反向代理：
  + 本主机：
    ssh -R 7890:127.0.0.1:14388 -p 22 root@ip
  + 服务器：
    export HTTP_PROXY=http://127.0.0.1:7890
    export HTTPS_PROXY=http://127.0.0.1:7890
  + <img width="2344" height="1380" alt="image" src="https://github.com/user-attachments/assets/851adc0a-0e3d-492e-b5ad-14e34e13c185" />


+ 方法二：clash的linux内核配置

  + 步骤：

    + 通过搜索镜像下载下clash的linux版本内核（官方仓库找不到了，大模型给出的地址都是404，从网上搜来的如链接https://drive.cn-v1-oss.eu.org/d/4b68087528ca4936b189/），选择[clash-freebsd-amd64-latest.gz](https://drive.cn-v1-oss.eu.org/d/4b68087528ca4936b189/files/?p=%2Fclash-freebsd-amd64-latest.gz)手动下载到本地，然后上传到服务器上解压配置

    + 导入购买的稳定vpn链接文档，测试网络连接，这里有些混乱

      + 由于这个版本clash直接用链接导入不了yaml文件，从自己电脑导出yaml文件然后又导入服务器中相应位置，然后激活`clash -d ~/.config/clash &`

      + 配置系统代理（临时生效）` export http_proxy=http://127.0.0.1:7890 ` `export https_proxy=http://127.0.0.1:7890 ` `export all_proxy=socks5://127.0.0.1:7891`(这里折腾了好多次，有http_proxy和socks5，最后网络还是有问题，大概这里端口可能有问题)

        

8、**操作耗时**：反复删除安装清理以及询问大模型来回折腾，三天左右未解决



