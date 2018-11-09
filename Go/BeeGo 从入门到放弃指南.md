# BeeGo 从入门到放弃指南

注释：此指南基于配置好GOPATH 安装好godep vendor

一、入门创建beego工程

首先安装好

```
go get github.com/beego/bee
```

然后控制台中输入

```
bee new project_name
```

根据修改conf/app.conf可以修改相应的信息

使用bee run启动工程

使用bee run可以热部署功能 ，但无法在IDE中调试。ide中启动main.go 无法热部署，用哪个启动自己取舍。

```
bee run
```

这个时候可以修改一个Controller
新建一个BaseController.go 放置于controller 目录下
新建一个结构体，感觉是跟类差不多的东西吧。

```
type BaseController struct {
	beego.Controller		// 这种组合方式 可以直接使用beego.Controller的所有方法
	controllerName string //控制器名称
	actionName     string //当前action名
}
```

新建一个HomeController.go
跟上面差不多，结构体，组合BaseController

```
type HomeController struct {
	BaseController
}

func (this *HomeController) Index(){
	this.Ctx.WriteString("this a index page.")
}
```

修改router.go

```
package routers

import (
	"admin_sys_exer/controllers"
	"github.com/astaxie/beego"
)

func init() {
    beego.Router("/", &controllers.HomeController{},"GET:Index")
}
```

浏览器输入 127.0.0.1:port/   即可看到输入的内容

入门实例结束。

接下来会在此基础上写一个后台管理系统

二、beego中的Controller