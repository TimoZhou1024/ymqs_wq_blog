---
title: 群晖SSL证书自动续签+反向代理
date: \today
tags: 
- Synology
- acme
- Nginx
- 反向代理
categories: 
- tech
---

## SSL证书问题

```bash
git clone https://gitee.com/neilpang/acme.sh.git
cd acme.sh

./acme.sh --install -m your_email@example.com --force
export Ali_Key="YOUR_ALI_KEY"
export Ali_Secret="YOUR_ALI_SECRET"
./acme.sh --issue --dns dns_ali -d example.com -d *.example.com --server letsencrypt

export SYNO_Username='YOUR_SYNOLOGY_USERNAME'
export SYNO_Password='YOUR_SYNOLOGY_PASSWORD'
/root/.acme.sh/acme.sh --deploy --home /root/.acme.sh -d example.com --deploy-hook synology_dsm
cat /root/.acme.sh/example.com_ecc/example.com.conf

# 证书检查更新
/root/.acme.sh/acme.sh --cron --home /root/.acme.sh

# 在部署证书前提权一下
sudo su root

# 部署证书
/root/.acme.sh/acme.sh --deploy --home /root/.acme.sh --deploy-hook synology_dsm  -d example.com
```

## 参考链接

[阿里云 DNS API](https://github.com/acmesh-official/acme.sh/wiki/dnsapi#dns_ali)
[acme.sh](https://github.com/Eagle3386/acme.sh)