# URL 路由[English](README.md)

<p align="center">
    <a href="http://perfect.org/get-involved.html" target="_blank">
        <img src="http://perfect.org/assets/github/perfect_github_2_0_0.jpg" alt="Get Involed with Perfect!" width="854" />
    </a>
</p>

<p align="center">
    <a href="https://github.com/PerfectlySoft/Perfect" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_1_Star.jpg" alt="Star Perfect On Github" />
    </a>  
    <a href="http://stackoverflow.com/questions/tagged/perfect" target="_blank">
        <img src="http://www.perfect.org/github/perfect_gh_button_2_SO.jpg" alt="Stack Overflow" />
    </a>  
    <a href="https://twitter.com/perfectlysoft" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_3_twit.jpg" alt="Follow Perfect on Twitter" />
    </a>  
    <a href="http://perfect.ly" target="_blank">
        <img src="http://www.perfect.org/github/Perfect_GH_button_4_slack.jpg" alt="Join the Perfect Slack" />
    </a>
</p>

<p align="center">
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Swift-3.0-orange.svg?style=flat" alt="Swift 3.0">
    </a>
    <a href="https://developer.apple.com/swift/" target="_blank">
        <img src="https://img.shields.io/badge/Platforms-OS%20X%20%7C%20Linux%20-lightgray.svg?style=flat" alt="Platforms OS X | Linux">
    </a>
    <a href="http://perfect.org/licensing.html" target="_blank">
        <img src="https://img.shields.io/badge/License-Apache-lightgrey.svg?style=flat" alt="License Apache">
    </a>
    <a href="http://twitter.com/PerfectlySoft" target="_blank">
        <img src="https://img.shields.io/badge/Twitter-@PerfectlySoft-blue.svg?style=flat" alt="PerfectlySoft Twitter">
    </a>
    <a href="http://perfect.ly" target="_blank">
        <img src="http://perfect.ly/badge.svg" alt="Slack Status">
    </a>
</p>

此示例说明如何设置URL路由，以及将请求直接指向自定义处理程序.

通过Swift Package Manager使用示例, 请执行 ```swift build```并接着执行 ``` .build/debug/URLRouting```.

通过Xcode使用示例, 请运行**URL Routing**. 这将会启动Perfect HTTP服务器. 

在您的浏览器中访问[http://localhost:8181/](http://localhost:8181/),试验已经添加的路由.


## 问题报告

目前我们已经把所有错误报告合并转移到了JIRA上，因此github原有的错误汇报功能不能用于本项目。

您的任何宝贵建意见或建议，或者发现我们的程序有问题，欢迎您在这里告诉我们。[http://jira.perfect.org:8080/servicedesk/customer/portal/1](http://jira.perfect.org:8080/servicedesk/customer/portal/1)。

目前问题清单请参考以下链接： [http://jira.perfect.org:8080/projects/ISS/issues](http://jira.perfect.org:8080/projects/ISS/issues)


## 启用URL路由

下面的代码取自示例项目，并演示如何创建和向服务器添加路由.

```swift
var routes = Routes()

routes.add(method: .get, uris: ["/", "index.html"], handler: indexHandler)
routes.add(method: .get, uri: "/foo/*/baz", handler: echoHandler)
routes.add(method: .get, uri: "/foo/bar/baz", handler: echoHandler)
routes.add(method: .get, uri: "/user/{id}/baz", handler: echo2Handler)
routes.add(method: .get, uri: "/user/{id}", handler: echo2Handler)
routes.add(method: .post, uri: "/user/{id}/baz", handler: echo3Handler)

// 通过这个命令测试curl:
// curl --data "{\"id\":123}" http://0.0.0.0:8181/raw --header "Content-Type:application/json"
routes.add(method: .post, uri: "/raw", handler: rawPOSTHandler)

// 结尾通配符能够匹配从通配符开始的所有以之前内容的URI
routes.add(method: .get, uri: "**", handler: echo4Handler)

//程序接口URI的路由

// 为程序接口API版本v1创建路由表
var api = Routes()
api.add(method: .get, uri: "/call1", handler: { _, response in
	response.setBody(string: "API CALL 1")
	response.completed()
})
api.add(method: .get, uri: "/call2", handler: { _, response in
	response.setBody(string: "API CALL 2")
	response.completed()
})

// API版本v1
var api1Routes = Routes(baseUri: "/v1")
// API版本v2
var api2Routes = Routes(baseUri: "/v2")

// 为API版本v1增加主调函数
api1Routes.add(routes: api)
// 为API版本v2增加主调函数
api2Routes.add(routes: api)
// 更新API版本v2主调函数
api2Routes.add(method: .get, uri: "/call2", handler: { _, response in
	response.setBody(string: "API v2 CALL 2")
	response.completed()
})

// 将两个版本的内容都注册到服务器主路由表上
routes.add(routes: api1Routes)
routes.add(routes: api2Routes)

// Check the console to see the logical structure of what was installed.
print("\(routes.navigator.description)")

// 创建服务器对象
let server = HTTPServer()

// 监听8181端口
server.serverPort = 8181

// 注册路由
server.addRoutes(routes)

do {
    // 启动服务器
    try server.start()
} catch PerfectError.networkError(let err, let msg) {
    print("Network error thrown: \(err) \(msg)")
}
```
## 处理请求

`EchoHandler`例子如下.

```swift
func echoHandler(request: HTTPRequest, _ response: HTTPResponse) {
	response.appendBody(string: "Echo handler: You accessed path \(request.path) with variables \(request.urlVariables)")
	response.completed()
}
```

## 如果希望与Apache配合使用
下面的Apache配置脚本能够将从Apache服务器接收到的客户请求以管道方式重新定向到Perfect 服务器，哪怕请求的地址指向的是原本在Apache服务器上并不存在的文件。

```apacheconf
	RewriteEngine on
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule (.*) - [L,NS,H=perfect-handler]
```


## 更多内容
关于Perfect更多内容，请参考[perfect.org](http://perfect.org)官网。
