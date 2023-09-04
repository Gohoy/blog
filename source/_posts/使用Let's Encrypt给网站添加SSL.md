---
title: 使用Let's Encrypt给网站添加SSL
category: 教程
tag: 
 - 教程
 - ssl
 - https

---

# 使用Let's Encrypt给网站添加SSL

## 1.安装cerbot

```bash
sudo apt install certbot python3-certbot-nginx
```

## 2.为nginx添加SSL

```bash
sudo certbot --nginx
```

## 3.续订SSL(默认三个月)

```bash
certbot renew
```

