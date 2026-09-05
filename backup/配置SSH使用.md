### 没有配置SSH的情况下，git克隆时可能会出现失败的情况
### 1. 检查是否有 SSH 密钥
`ls ~/.ssh/id_*`

#### 如果没有，生成一个
`ssh-keygen -t ed25519 -C "mparker@archMP"`
一路回车使用默认配置不设密码

### 2. 查看并复制公钥
`cat ~/.ssh/id_ed25519.pub`
然后打开https://github.com/settings/ssh/new
title填自己的主机名
密钥粘贴公钥
点Add SSH key添加密钥

### 3. 测试
`ssh -T git@github.com`
如果成功，会显示：
`Hi [你的用户名]! You've successfully authenticated...`
如果弹出以下信息：
```
The authenticity of host 'github.com (20.205.243.166)' can't be established.
ED25519 key fingerprint is: SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
直接输入`yes`回车即可
应该可以看到
`Hi [你的用户名]! You've successfully authenticated, but GitHub does not provide shell access.`