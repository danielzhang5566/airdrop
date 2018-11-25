# Snapdrop 

[Snapdrop](https://snapdrop.net): local file sharing in your browser - inspired by Apple's Airdrop.

#### Snapdrop (Version 2) is built with the following awesome technologies:
* Vanilla HTML5 / ES6 / CSS3  
* Progressive Web App
* [WebRTC](http://webrtc.org/)
* [WebSockets](http://www.websocket.org/) fallback (iDevices don't support WebRTC)
* [NodeJS](https://nodejs.org/en/)
* [Material Design](https://material.google.com/)


### Frequently Asked Questions

### Instructions
* [Video Instructions](https://www.youtube.com/watch?v=4XN02GkcHUM) (Big thanks to [TheiTeckHq](https://www.youtube.com/channel/UC_DUzWMb8gZZnAbISQjmAfQ))
* [idownloadblog](http://www.idownloadblog.com/2015/12/29/snapdrop/)
* [thenextweb](http://thenextweb.com/insider/2015/12/27/snapdrop-is-a-handy-web-based-replacement-for-apples-fiddly-airdrop-file-transfer-tool/)
* [winboard](http://www.winboard.org/artikel-ratgeber/6253-dateien-vom-desktop-pc-mit-anderen-plattformen-teilen-mit-snapdrop.html)
* [免費資源網路社群](https://free.com.tw/snapdrop/)

##### What about the connection? Is it a P2P-connection directly from device to device or is there any third-party-server?
It uses a P2P connection if WebRTC is supported by the browser. (WebRTC needs a Signaling Server, but it is only used to establish a connection and is not involved in the file transfer).

If WebRTC isn’t supported (Safari, IE) it uses a Web Sockets fallback for the file transfer. The server connects the clients with each other.  

##### What about privacy? Will files be saved on third-party-servers?
None of your files are ever saved on any server. 
Snapdrop doesn't even use a database. If you are curious have a look [at the Server](https://github.com/RobinLinus/snapdrop/blob/master/server/).

##### Is SnapDrop a fork of ShareDrop?
No. ShareDrop is built with Ember. Snapdrop is built with vanilla ES6. 
I wanted to play around with Progressive Web Apps and then I got the idea of a local file sharing app. By doing research on this idea I found and analysed ShareDrop. I liked it and thought about how to improve it.
ShareDrop uses WebRTC only and isn't compatible with Safari browsers. Snapdrop uses a Websocket fallback and some hacks to make Snapdrop work due to the download restrictions on iDevices. 


### Snapdrop is awesome! How can I support it? 
* [File bugs, give feedback, submit suggestions](https://github.com/RobinLinus/snapdrop/issues)
* Share Snapdrop on your social media.
* [Buy me a cup of coffee](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=R9C5E42UYEQCN)
* Fix bugs and make a pull request. 
* Do security analysis and suggestions

## Local Development
```
    git clone git@github.com:RobinLinus/snapdrop.git
    cd snapdrop/server
    npm install
    node index.js

    # open a second shell:
    cd snapdrop/client
    python -m SimpleHTTPServer
```

Now point your browser to http://localhost:8000.
    
## Deployment Notes
The client expects the server at http(s)://your.domain/server.

When serving the node server behind a proxy the `X-Forwarded-For` header has to be set by the proxy. Otherwise all clients that are served by the proxy will be mutually visible.

By default the server listens on port 3000.

For an nginx configuration example see `nginx.conf.example`.

## Licences
* Thanks to [Mark DiAngelo]() for the [Blop Sound](http://soundbible.com/2067-Blop.html)


## 改造记录
### 不按同一局域网IP分房间，改成所有网络同一房间
原始server创建的房间数据结构：

    { 
        '45.64.10.121':
       { '9c1224c9-f272-4b70-ac64-8e92bb2c68e5':
            Peer {
                socket: [WebSocket],
                ip: '45.64.10.121',
                id: '9c1224c9-f272-4b70-ac64-8e92bb2c68e5',
                rtcSupported: true,
                name: [Object],
                timerId: [Timeout],
                lastBeat: 1542954810497 
            },
         '969e6457-13bc-4d6f-a9bc-325b56b06e03; Hm_lvt_4fc61405bfdfe04b08a053cbd3e5971a=1542626590; Hm_lpvt_4fc61405bfdfe04b08a053cbd3e5971a=1542954531':
            Peer {
                socket: [WebSocket],
                ip: '45.64.10.121',
                id: '969e6457-13bc-4d6f-a9bc-325b56b06e03; Hm_lvt_4fc61405bfdfe04b08a053cbd3e5971a=1542626590; Hm_lpvt_4fc61405bfdfe04b08a053cbd3e5971a=1542954531',
                rtcSupported: true,
                name: [Object],
                timerId: 0,
                lastBeat: 1542954810663 
            } 
        },
      '45.64.22.107':
       { '3f0fd73f-d222-4ee0-b1f3-4f7456def4a6; Hm_lvt_4fc61405bfdfe04b08a053cbd3e5971a=1542596007; Hm_lpvt_4fc61405bfdfe04b08a053cbd3e5971a=1542596079':
            Peer {
                socket: [WebSocket],
                ip: '45.64.22.107',
                id: '3f0fd73f-d222-4ee0-b1f3-4f7456def4a6; Hm_lvt_4fc61405bfdfe04b08a053cbd3e5971a=1542596007; Hm_lpvt_4fc61405bfdfe04b08a053cbd3e5971a=1542596079',
                rtcSupported: true,
                name: [Object],
                timerId: [Timeout],
                lastBeat: 1542954805219 
            } 
        } 
    }

以上数据结构表示：当前server共创建了两个房间，第一个房间也就是`45.64.10.121`网络下存在两个设备。

因此，改造方式：
    1. 只创建一个默认房间`127.0.0.1`
    2. 随后进来的每台设备，同一进默认房间，不再区分ip进房



## 启动server

使用forever模块持久运行js

    npm install forever -g
    forever start index.js

Ps.可以指定app.js中的日志信息和错误日志输出文件，-o 就是console.log输出的信息，-e 就是console.error输出的信息：

    forever start -o out.log -e err.log index.js 

forever常用参数：

    start               Start SCRIPT as a daemon
    stop                Stop the daemon SCRIPT by Id|Uid|Pid|Index|Script
    stopall             Stop all running forever scripts
    restart             Restart the daemon SCRIPT
    restartall          Restart all running forever scripts
    list                List all running forever scripts


## nginx 配置

    server {
    
        listen 443;
        ssl on;
    
        root         /xxx/airdrop/client;
        server_name  xxx.com;
        index index.html index.htm;
    
        ssl_certificate      cert/xxx.pem;
        ssl_certificate_key  cert/xxx.key;
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
    
        location / {
            proxy_http_version 1.1;
        }
    
        location /server {
            # 代理    
            proxy_pass http://localhost:3000/;
            proxy_http_version 1.1;
            # 升级为WebSocket协议
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header X-Forwarded-For $remote_addr;
        }
    
        access_log  /var/log/nginx/sites/airdrop.susamko.com;
        error_page 404 /404.html;
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    
    }
    
    server {
        listen	 80;
        server_name  xxx.com;
    
        rewrite ^(.*)$ https://${server_name}$1 permanent; 
    }
