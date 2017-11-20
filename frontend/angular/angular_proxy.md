# Angular 调试时获取后端数据

项目使用的 Angular + Spring，项目使用 Angular-cli 创建，在后期优化过程中，将所有前端文件放在 dist 文件中，这就导致了一个问题，即使 Intellij 配置了热部署等，前端 html 文件和 ts 文件的变动也不会显示。只能 ng build 后再通过 maven 打包发布，过程相当复杂。

想到的解决方案一个是伪造数据，但无疑花费也是较多的。由于前后端分离，开发前端时，后端一般不会变化。所以想的是能否一直启动后端服务（端口8080），前端服务通过 `ng serve` 启动，端口4200 来先完成前端。但接着又有个问题，如何在这两个端口间通信呢？

在查询过程中，意外发现可以通过 proxy 的方式来解决这个问题，解决方案非常简单，如下：

- 在项目目录下创建文件 proxy.conf.json，内容如下：
```
{
  "/rest": {
    "target": "http://localhost:8080",
    "sercure": false
  }
}
```
说明：
  - `/rest` 为代理规则，即代理以 `/rest` 开头的请求。该项根据实际情况设置。
  - target 为目标服务地址。如一个 get 请求的地址为 `http://localhost:4200/rest/v1/user/1`，则将被代理为 `http://localhost:8080/rest/v1/user/1`。
  - secure 为是否开启 ssl 验证，设置为 false。
- 以 `ng serve --proxy-config proxy.conf.json` 的形式启动 ng 服务，即可。

##　参考
- [Angular2 + node 接口调试解决方案](http://www.jianshu.com/p/74bc41c3d95e)
