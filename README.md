# v2ray-network
本项目旨在突破越来越严苛的互联网环境，采用CDN+TLS+Nginx+v2ray进行伪装并突破防火墙。

# 技能点列表
- [bandwagonhost[centos7]](https://bandwagonhost.com)【境外vps】
- [cloudflare](cloudflare.com)【CDN】
- [godaddy](https://www.godaddy.com/)【域名厂商】
- [letsencrypt](https://letsencrypt.org/)【HTTPS】
- [Nginx](https://www.nginx.com/)【反向代理】
- [v2ray](v2ray.com)【代理工具】

# 1.准备工作
## 1.注册[cloudflare](cloudflare.com)
## 2.注册[godaddy](https://www.godaddy.com/)并购买域名
- 域名可选择xyz结尾的国际域名，可采用多字符乱码的方式组合域名，首年大概8RMB左右，第二年可以直接买一个新的。

## 3.修改godaddy域名的DNS解析
### 1.登录cloudflare，添加域名
<img src="https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/fodder/cloudflare.png" width=400>

### 2.选择套餐
- 如果仅仅只享受科学上网功能，选择free即可
- 如果需要更好的网络环境、更快的速度，可选择相应的套餐
<img src="https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/fodder/cloudflare_plan.png" width=400>

### 3.根据提示修改godaddy的dns解析
- cloudflare提示界面
<img src="https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/fodder/cloudflare_dns.png" width=400>

- 修改dns
<img src="https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/fodder/godayddy_dns.png" width=400>

## 4.增加cloudflare域名解析
- 添加域名解析(记录)，可以选择二级域名，这样就可以一个月解析到不同的服务器，name填写你要解析的二级域名的host部分，比如ls.example.com 只填写ls即可
<img src="https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/fodder/cloudflare_record_dns.png" width=400>

## 5.修改godaddy SSL/TLS
- 如果vps选择使用https，需要把类型修改为Full
<img src="https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/fodder/cloudflare_tls.png" width=400>

# 2.vps配置Nginx、https
## 1.安装Nginx
```
yum install nginx
```
## 2.nginx配置文件

- 1.下载配置文件并替换默认文件
```
cd /etc/nginx&&rm -rf /etc/nginx/nginx.conf&&wget https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/config/nginx.conf
# 如果缺少wget 则执行，然后重复上面的命令
yum install wget
```
- 将下载好的文件中关于ls.xxx.xyz的内容都替换成你的二级域名

## 3.生成https

- 1.安装acme.sh
```
curl https://get.acme.sh | sh
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
100   671  100   671    0     0    680      0 --:--:-- --:--:-- --:--:--   679
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
100  112k  100  112k    0     0   690k      0 --:--:-- --:--:-- --:--:--  693k
[Fri 30 Dec 01:03:32 GMT 2016] Installing from online archive.
[Fri 30 Dec 01:03:32 GMT 2016] Downloading https://github.com/Neilpang/acme.sh/archive/master.tar.gz
[Fri 30 Dec 01:03:33 GMT 2016] Extracting master.tar.gz
[Fri 30 Dec 01:03:33 GMT 2016] Installing to /home/user/.acme.sh
[Fri 30 Dec 01:03:33 GMT 2016] Installed to /home/user/.acme.sh/acme.sh
[Fri 30 Dec 01:03:33 GMT 2016] Installing alias to '/home/user/.profile'
[Fri 30 Dec 01:03:33 GMT 2016] OK, Close and reopen your terminal to start using acme.sh
[Fri 30 Dec 01:03:33 GMT 2016] Installing cron job
no crontab for user
no crontab for user
[Fri 30 Dec 01:03:33 GMT 2016] Good, bash is found, so change the shebang to use bash as preferred.
[Fri 30 Dec 01:03:33 GMT 2016] OK
[Fri 30 Dec 01:03:33 GMT 2016] Install success!
```

- 2.生成https证书
```
sudo ~/.acme.sh/acme.sh --issue -d ls.xxx.xyz --standalone -k ec-256

# 如果提示Please install socat tools first.则执行，安装完成后继续重复执行上面的命令
yum install socat
```

- 3.安装证书
```
~/.acme.sh/acme.sh --installcert -d ls.xxx.xyz --fullchainpath /etc/nginx/ls.xxx.xyz.crt --keypath /etc/nginx/ls.xxx.xyz.key --ecc
```

- 4.修改/etc/nginx/nginx.conf
```
# 将下面这部分前面的#去掉，并将ssl_certificate、ssl_certificate_key修改成自己的路径

# ssl on;
# ssl_certificate /etc/nginx/ls.xx.xyz.crt;
# ssl_certificate_key /etc/nginx/ls.xx.xyz.key;
# ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
# ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
# ssl_prefer_server_ciphers on;
```
# 3.配置v2ray
## 1.安装v2ray

```
bash <(curl -L -s https://install.direct/go.sh)
```

## 2.v2ray配置文件

- 下载config_ws_tls.json
```
cd&&wget https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/config/config_ws_tls.json
```

- 配置文件的id可以自己生成一个新的，替换即可
```
/usr/bin/v2ray/v2ctl  uuid
```

## 3.启动v2ray
```
/usr/bin/v2ray/v2ray -config ./config_ws_tls.json&
```

# 4.客户端
## 1.MacOS
- 下载V2RayU[点我下载](https://github.com/yanue/V2rayU/releases/download/1.4.1/V2rayU.dmg)
- 下载后打开，服务器设置，修改address即可
<img src="https://raw.githubusercontent.com/jerry-ap/v2ray-network/master/fodder/v2rayU_服务器配置.png" width=400>

- pac设置，添加下面的链接
```
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```
- 选择使用Pac模式，即可

# 5.其余设置
