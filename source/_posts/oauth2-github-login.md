---
title: 第三方授权登录(oauth2)--Github授权登录 
date: 2017-09-29 15:41:59
tags:
    - OAuth2
    - github
    - Java
categories: Java
---

## 第三方授权登录-github授权登录

> 现在时下的框架都是非常流行第三方的授权登录，比如QQ，微信，微博，或者github等等，都是基于OAuth2授权令牌。

### 前期的准备

在现在很多互联网项目当中很少没有第三方登录的，本来想用QQ或者微信来弄，但是发现认证有点麻烦，后来看到github上面的授权很简单，几乎只要有一个github账号就能开始开发了。其实不管是哪个平台授权登录，只要了解了一个原理，其它的都类似了。整个流程图如下：
![2017-09-29-16-25-04](/images/qiniu/2017-09-29-16-25-04.png)

#### github 账号

默认已经有了，如果没有的话作为开发者还是去注册一个吧，不多说了。

#### 注册一个应用

注册一个我们的应用，该应用就是我们需要用第三方授权登录的应用,github有个好处，可以用`127.0.0.1`直接挂到本地进行测试，不知道其他的第三方可不可以，还待验证。
![2017-09-29-15-55-40](/images/qiniu/2017-09-29-15-55-40.png)
<!--more-->
#### 获取到client\_id和client\_key

注册新应用之后我们主要是要拿到`client`和`client_key`:
![2017-09-29-16-10-22](/images/qiniu/2017-09-29-16-10-22.png)
拿到之后我们就可以开始开发了。

### SpringBoot实现授权登录

首先页面要有一个请求授权的操作，我们简化如下:

```html
<a href="http://github.com/login/oauth/authorize?client_id=0430a6c311c3dd1f4869"> github登录</a>
```

![2017-09-29-16-35-04](/images/qiniu/2017-09-29-16-35-04.png)

之后，需要在配置的回调地址就是我们`127.0.0.1`设置的url地址：

```java
    @GetMapping("login/github")
    public String loginGithub(String code, Model model) {
        logger.info(code);
        System.out.println("code: " + code);
        String url = "https://github.com/login/oauth/access_token";

        MultiValueMap<String, String> map = new LinkedMultiValueMap<>();
        map.add("client_id", ConfigConstant.CLIENT_ID);
        map.add("client_secret", ConfigConstant.CLIENT_SECRET);
        map.add("code", code);
        JSONObject jsonObject = RemoteRequestUtil.post(url, map, JSONObject.class);
        System.out.println("jsonObject:" + jsonObject);
        model.addAttribute("result", jsonObject);
        return "login/success";
    }
```
上面用了一个`RemoteRequestUtils`,其实你只要有一个可以调用post方法的就可以了,我用的是`RestTemplate`来实现远程调用的，请注意:`RestTemplate 只支持传参数MultiValueMap`。
```java
public class RemoteRequestUtil {

    private static RestTemplate restTemplate = new RestTemplate();

    public static <T> T post(String url, Object request, Class<T> clazz, Object... uriValues) {
        return restTemplate.postForObject(url, request, clazz, uriValues);
    }

    public static String get(String url) {
        System.out.println(url);
        return restTemplate.getForObject(url, String.class);
    }

    public static <T> T getT(String url, Class<T> clazz) {
        System.out.println(url);
        return restTemplate.getForObject(url, clazz);
    }

    public static <T> T postT(String url, String param, Class<T> clazz) {
        System.out.println(url + ":" + param);
        return restTemplate.postForObject(url, param, clazz);
    }

    public static <T> T postT(String url, Object param, Class<T> clazz) {
        System.out.println(url + ":" + param);
        return restTemplate.postForObject(url, param, clazz);
    }

    private RemoteRequestUtil() {
    }
}
```

当然你也可以是用POJO的方式，但是地城的HttpEntity也是用的MultiValueMap,不信可以自己看看源码，如果用POJO的方式如下:

```java
        GithubRequestLogin param = new GithubRequestLogin();
        param.setClient_id(ConfigConstant.CLIENT_ID);
        param.setClient_secret(ConfigConstant.CLIENT_SECRET);
        param.setCode(code);


        JSONObject jsonObj = RemoteRequestUtil.postT(url, param, JSONObject.class);
        System.out.println(jsonObj);
```

到这里，我们就可以获取到`access_token`了,现在就可以用token获取用户信息了。

获取用户信息的代码：  

```java
    @GetMapping("user/info")
    public String getGitHubUserInfo(String token, Model model) {
        String url = "https://api.github.com/user?access_token=" + token;
        String result = RemoteRequestUtil.get(url);
        model.addAttribute("result", result);
        return "login/success";
    }
```
之后就是这样的：
![2017-09-29-16-38-25](/images/qiniu/2017-09-29-16-38-25.png)
这样就可以说是将信息获取完了。其他第三方平台也是大同小异常，以后有机会再补上。
当然还有一个问题要留下来，如果token过期了该怎么破了？怎么知道是否过期了？可以思考下。

代码比较简单，需要的话可以看我写的，代码写的有点不太贵方，只做参考，记得替换成你的`client_id`和`client_secret`：
[github授权登录](https://gitlab.com/chenzhijun/github-login)

