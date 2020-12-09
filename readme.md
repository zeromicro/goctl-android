# goctl-android

![go-zero](https://img.shields.io/badge/Github-go--zero-brightgreen?link=https://github.com/tal-tech/go-zero&logo=github)
![license](https://img.shields.io/badge/License-Apache-blue?link=https://github.com/zeromicro/goctl-android/blob/master/LICENSE&logo=apache)
![Go](https://github.com/zeromicro/goctl-android/workflows/Go/badge.svg)

goctl-android是一款基于goctl的插件，用于生成java（android）端http client请求代码。
本插件特性：
* 仅支持post json
* 支持get query参数
* 支持path路由变量
* 仅支持响应体为json

> 警告：本插件是对goctl plugin开发流程的指引，切勿用于生产环境。

# 插件使用
* 编译goctl-android插件
    ```shell script
    $ GO111MODULE=on GOPROXY=https://goproxy.cn/,direct go get -u github.com/zeromicro/goctl-android
    ```
* 将`$GOPATH/bin`中的`goctl-android`添加到环境变量
* 创建api文件
    ```go
    info(
    	title: "type title here"
    	desc: "type desc here"
    	author: "type author here"
    	email: "type email here"
    	version: "type version here"
    )
    
    
    type (
    	RegisterReq {
    		Username string `json:"username"`
    		Password string `json:"password"`
    		Mobile string `json:"mobile"`
    	}
    	
    	LoginReq {
    		Username string `json:"username"`
    		Password string `json:"password"`
    	}
    	
    	UserInfoReq {
    		Id string `path:"id"`
    	}
    	
    	UserInfoReply {
    		Name string `json:"name"`
    		Age int `json:"age"`
    		Birthday string `json:"birthday"`
    		Description string `json:"description"`
    		Tag []string `json:"tag"`
    	}
    	
    	UserSearchReq {
    		KeyWord string `form:"keyWord"`
    	}
    )
    
    service user-api {
    	@doc(
    		summary: "注册"
    	)
    	@handler register
    	post /api/user/register (RegisterReq)
    	
    	@doc(
    		summary: "登录"
    	)
    	@handler login
    	post /api/user/login (LoginReq)
    	
    	@doc(
    		summary: "获取用户信息"
    	)
    	@handler getUserInfo
    	get /api/user/:id (UserInfoReq) returns (UserInfoReply)
    	
    	@doc(
    		summary: "用户搜索"
    	)
    	@handler searchUser
    	get /api/user/search (UserSearchReq) returns (UserInfoReply)
    }
    ```
* 生成android代码
    
    ```shell script
    $ goctl api plugin -plugin goctl-android="android -package com.tal" -api user.api -dir .
    ```
    >说明： 其中`goctl-android`为可执行的二进制文件，`"android -package com.tal"`为goctl-plugin自定义的参数，这里需要用引号`""`引起来。

我们来看一下生成java代码后的目录结构
```text
├── bean
│   ├── LoginReq.java
│   ├── RegisterReq.java
│   ├── UserInfoReply.java
│   ├── UserInfoReq.java
│   └── UserSearchReq.java
├── service
│   ├── IService.java
│   └── Service.java
└── user.api
```

> [点击这里](https://github.com/zeromicro/ClientCall) 查看Java示例源码

maven依赖
```xml
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>retrofit</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.0</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.75</version>
</dependency>
<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>converter-gson</artifactId>
    <version>2.9.0</version>
</dependency>
```

> 本插件是基于retrofit来实现http请求，因此会用到一些java依赖，gradle包管理形式自行处理。

* 编写测试
```Java
public static void main(String[] args) {
LoginReq loginReq = new LoginReq();
loginReq.setUsername("zeromicro");
loginReq.setPassword("111111");
Service service = Service.getInstance();
service.login(loginReq, new Callback<Void>() {
    public void onResponse(Call<Void> call, Response<Void> response) {
        System.out.println("login success");
    }

    public void onFailure(Call<Void> call, Throwable throwable) {

    }
});

RegisterReq registerReq=new RegisterReq();
registerReq.setUsername("zeromicro");
registerReq.setPassword("1111");
registerReq.setMobile("12311111111");
service.register(registerReq, new Callback<Void>() {
    public void onResponse(Call<Void> call, Response<Void> response) {
        System.out.println("register success");
    }

    public void onFailure(Call<Void> call, Throwable throwable) {

    }
});

UserInfoReq infoReq = new UserInfoReq();
infoReq.setId("8888");
service.getUserInfo(infoReq, new Callback<UserInfoReply>() {
    public void onResponse(Call<UserInfoReply> call, Response<UserInfoReply> response) {
        System.out.println("userInfo:"+ JSON.toJSONString(response.body()));
    }

    public void onFailure(Call<UserInfoReply> call, Throwable throwable) {

    }
});

UserSearchReq searchReq=new UserSearchReq();
searchReq.setKeyWord("song");
service.searchUser(searchReq, new Callback<UserInfoReply>() {
    public void onResponse(Call<UserInfoReply> call, Response<UserInfoReply> response) {
        System.out.println("search:"+JSON.toJSONString(response.body()));
    }

    public void onFailure(Call<UserInfoReply> call, Throwable throwable) {

    }
});
```

* 请求结果
    * client log
    ```text
    register success
    login success
    search:{"age":20,"birthday":"1991-01-01","description":"coding now","name":"zeromicro","tag":["Golang","Android"]}
    userInfo:{"age":20,"birthday":"1991-01-01","description":"coding now","name":"zeromicro","tag":["Golang","Android"]}
    ```
    
    * server log
    ```text
    SearchUser: {KeyWord:song}
    GetUserInfo: {Id:8888}
    Login: {Username:zeromicro Password:111111}
    Register: {Username:zeromicro Password:1111 Mobile:12311111111}
    ```

# 插件开发流程

* 自定义参数
    ```go
    commands = []*cli.Command{
        {
            Name:   "android",
            Usage:  "generates http client for android",
            Action: action.Android,
            Flags: []cli.Flag{
                &cli.StringFlag{
                    Name:  "package",
                    Usage: "the package of android",
                },
            },
        },
    }
    ```

* 获取goctl传递过来的json信息
    * 利用goctl中提供的方法解析
        ```go
        plugin, err := plugin.NewPlugin()
        if err != nil {
            return err
        }
        ```
  
    * 或者自定义结构体去反序列化
  
        ```go
        var plugin generate.Plugin
        plugin.ParentPackage = pkg
        err = json.Unmarshal(std, &plugin)
        if err != nil {
            return err
        }
        ```

* 实现插件逻辑
    ```go
    generate.Do(plugin)
    ```

>说明：上述摘要代码来自goctl-android,完整信息可浏览源码。
