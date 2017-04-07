title: "Https养成手记"
date: 2016-10-09 17:48:42
tags: ["Https", "Ubuntu", "Nginx"]
---

把大象放进冰箱分三步，那么实现https化你的网站也可以分三步：
1. `openssl req -newkey rsa:2048 -keyout server.key -out server.csr`在服务器上生成key和csr，可以直接扔在nginx目录，方便。此处需要**注意**的是，生成的private key需要移除passphrase，就是运行前面命令时让输入的`PEM`的值，否则nginx error。
执行`openssl rsa -in server.key -out unencripted-server.key`就OK了，我们在nginx配置中就使用未加密的key文件
2. 去[Startssl](https://www.startssl.com/Certificates/ApplySSLCert?level=1)，将刚才得到的server.csr文件内容填入表单中，Submit！
![Startssl配置](https://dn-xuqi.qbox.me/https.png)
然后下载生成的证书，因为我用的是nginx，就将NginxServer下面的的crt文件scp到服务器中
3. 配置nginx并将http80重定向到443即可
```
server {
    listen 443 ssl;
    ssl_certificate /path/to/crt;
    ssl_certificate_key /path/to/unencripted/key;
    ssl_session_tickets off;
    ssl_session_cache   shared:SSL:10m;
    server_name  yourdomain;

    location / {
        proxy_redirect off;
        proxy_pass  http://yourUpstream;
    }
}

server {
    listen 80;
    server_name yourdomain;
    return 301 https://yourdomain$request_uri;
}
```

Got it... nginx reload就可以了

Thanks to:  
[给你的网站穿上外衣 － HTTPS 免费部署指南](https://segmentfault.com/a/1190000007024673?utm_source=weekly&utm_medium=email&utm_campaign=email_weekly)  
[Nginx 102 error ssl](http://stackoverflow.com/questions/18101217/error-102-nginx-ssl)