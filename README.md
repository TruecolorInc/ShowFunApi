# ShowFunApi
base_ulr = http://papertiger.top/

http://118.25.63.125:3000/

  

1. 获取此项目的提交权限
2. 修改db.json 更新接口文档, 注意json-server第一级为接口的path，所以必须是对象。
3. 稍等片刻(自动刷新)



# 接口文档
base_ulr = http://papertiger.top/
- 首页顶部tab: /homeTabs
- 热门tab: /hotTabs
- 热门列表:/hotLists
- 追番接口: /favoriteLists
- 详情页信息接口：/detailInfo
- 详情页分集接口：/detailEpisode


# 原理

## json-server 

```javascript
const jsonServer = require('json-server')
const server = jsonServer.create()
const router = jsonServer.router('./ShowFunApi/db.json')
const middlewares = jsonServer.defaults()

server.use(middlewares)
server.use(router)
server.listen(3000, ()=> {
        console.log('JSON Server is running')
})
```

## nginx 反向代理

```shell
  server {
                listen  80;
                server_name www.papertiger.top;
                location / {
                        proxy_pass http://127.0.0.1:3000/;
                }

                location /auto_build {
                        proxy_pass http://127.0.0.1:6666/auto_build;
                }
}
```

## GithubWebHooks 监听push事件

screctKye 生成方法https://developer.github.com/webhooks/securing/

在repository的setting里配置WebHooks

```
var http = require('http');
var spawn = require('child_process').spawn;
var createHandler = require('github-webhook-handler');

// 下面填写的myscrect跟github webhooks配置一样，下一步会说；path是我们访问的路径
var handler = createHandler({ path: '/auto_build', secret: 'screctKey' });

http.createServer(function (req, res) {
  handler(req, res, function (err) {
    res.statusCode = 404;
    res.end('no such location');
  })
}).listen(6666);

handler.on('error', function (err) {
  console.error('Error:', err.message)
});

// 监听到push事件的时候执行我们的自动化脚本
handler.on('push', function (event) {
  console.log('Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref);

  runCommand('sh', ['./auto_build.sh'], function( txt ){
    console.log(txt);
  });
});

function runCommand( cmd, args, callback ){
    var child = spawn( cmd, args );
    var resp = '';
    child.stdout.on('data', function( buffer ){ resp += buffer.toString(); });
    child.stdout.on('end', function(){ callback( resp ) });
}
```

### 执行脚本自动更新
```
#! /bin/bash

SITE_PATH='../ShowFunApi/'

cd $SITE_PATH
git reset --hard origin/master
git clean -f
git pull
git checkout master
pm2 reload server
```


