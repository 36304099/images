# cloudflare + github + 自己域名 搭建免费永久图床

引用搭建教程 https://www.geeklinux.cn/jsjc/166.html

demo: https://pic.beijin.email/picGo/皮卡.png


### 创建 GitHub仓库 ： 
在GitHUB上创建一个仓库，专门用于存放图片的，怎么创建就不用多说了吧，不能使用私有仓库！！！

获取 GitHub Token
需要获取GitHub Token， tokens ：https://github.com/settings/tokens

创建一个新的token(classic方式可以设置Expiration永不过期)

勾选repo，将Expiration过期时间调整为永不过期

然后记住你的token，接下来会用到

### Picgo配置
下载按照picgo，下载地址：https://molunerfinn.com/PicGo/

安装完成后，点击图床设置，GitHub

仓库名：这个就填写你的GitHub仓库名字/仓库。例如我的仓库名称是geeklinux,我创建了一个叫做picgo的仓库，所以我就填为 geeklinux/picgo

分支名称：这里我们都是单分支，填写main即可。

Token：这个就是你上一步在GitHub获取到的Token。

存储路径，这个就是配置上传的图片放到你的仓库中的哪一个子目录里，为空默认放你仓库根目录


### cloudflare设置

cloudflare上创建账号，新建worker

worker最终代码，并且绑定自己域名(不是必须)好用些：

~~~ javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  // 获取请求的 URL
  const url = new URL(request.url)
  
  // 提取完整路径
  const filePath = url.pathname.slice(1) // 去掉开头的斜杠

  // 设置实际代理地址
  const proxyUrl = `https://raw.githubusercontent.com/36304099/images/main/${filePath}`

  // 发起代理请求
  const response = await fetch(proxyUrl)

  // 返回响应
  return new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers: response.headers
  })
}
~~~





## 下面引用的原文内容没用到(可以不用cdn.jsdelivr.net)

**自定义域名：我们可以采用 cdn.jsdelivr.net 来加速访问你的仓库

配置方法：https://cdn.jsdelivr.net/gh/github用户名/仓库名/

但是处于某些原因，国内访问cdn.jsdelivr.net时不时的抽风或者无法访问，非常影响使用体验，所以这里我提供两种解决方法，请继续往下看。

配置代理加速
方法1：反向代理
我们知道，cdn.jsdelivr.net 在国内的访问速度并不理想，我们可以使用香港等延迟比较低比较稳定的服务器通过反向代理来加速访问 cdn.jsdelivr.net

例如我这里使用的一台日本的服务器，使用nginx反向代理cdn.jsdelivr.net 配置如下；

server {  
    listen 80;  
    server_name img.geeklinux.cn;  
  
    # Redirect all HTTP requests to HTTPS  
    location / {  
        return 301 https://$host$request_uri;  
    }  
}  
  
server {  
    listen 443 ssl;  
    server_name img.geeklinux.cn;
      
    ssl_certificate     F:/cert/geeklinux.cn/ssl.pem;  
    ssl_certificate_key F:/cert/geeklinux.cn/ssl.key;  
 
    location / {  
        proxy_pass https://cdn.jsdelivr.net:443;
        proxy_ssl_server_name on;
        proxy_set_header Host cdn.jsdelivr.net;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
    }  
}

访问测试

