---
title: Configure the Shadowsocks client to across the GFW

categories: 

- Programer
- Net

tags: 

- GFW
- 翻墙
- shadowsocks

---

## 参考 ##

[shadowsocks.org](https://shadowsocks.org)

[shadowsocks client  download](https://shadowsocks.org/en/download/clients.html)

[shadowsoks Installation wiki](https://github.com/shadowsocks/shadowsocks-qt5/wiki/Installation)

[shadowsocks on github](https://github.com/shadowsocks)

[shadowsocks config](https://shadowsocks.org/en/config/advanced.html)

## commad line client ##

>  pip install shadowsocks

## configuration ##

[Install and use shadowsocks command line client on linux](https://www.linuxbabe.com/desktop-linux/how-to-install-and-use-shadowsocks-command-line-client)

[Using shadowsocks with command line tools](https://github.com/shadowsocks/shadowsocks/wiki/Using-Shadowsocks-with-Command-Line-Tools)

1. Create a Configuration File

> vim /etc/shadowsocks_client.json

2. Put the following text in the file. Replace server-ip with your actual IP and set a password.

```bash

{
"server":"server-ip",
"server_port":8000,
"password":"your-password",
"method":"aes-256-cfb"
"auth":false
}

```
3. To run in the background

> sudo sslocal -c /etc/shadowsocks.json -d start






