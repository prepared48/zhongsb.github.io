---
layout: post
title:  "【翻译】Reactor 第七篇 Spring WebFlux 怎么进行异常处理"
date:   2021-06-25 09:11:54
categories: Java 响应式编程
tags: Java WebFlux
mathjax: true
---

* content
{:toc}

在本教程中，我们将通过一个实际示例了解Spring WebFlux项目中处理错误的各种策略。

我们还将指出使用一种策略比另一种策略更有利的地方，并在最后提供完整源代码的链接。





## 1 概览

在本教程中，我们将通过一个实际示例了解Spring WebFlux项目中处理错误的各种策略。

我们还将指出使用一种策略比另一种策略更有利的地方，并在最后提供完整源代码的链接。

## 2 开始示例代码

maven 设置和之前介绍 Spring WebFlux 的文章一样，

对于我们的示例，我们将使用一个 RESTful 端点，它将用户名作为查询参数并返回“Hello username”作为结果。首先，让我们创建一个路由函数，这个路由函数将 “/hello” 请求路由到处理程序中名为 handleRequest 的方法，代码如下：

```java
@Bean
public RouterFunction<ServerResponse> routeRequest(Handler handler) {
    return RouterFunctions.route(RequestPredicates.GET("/hello")
      .and(RequestPredicates.accept(MediaType.TEXT_PLAIN)), 
        handler::handleRequest);
    }
```

然后，我们定义个 handleRequest() 方法，这个方法调用 sayHello() 方法，并找到一个在 ServerResponse 中包含或返回其（sayHello方法的返回）结果的方法。

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return 
      //...
        sayHello(request)
      //...
}
```

最后，实现 sayHello 方法，实现很简单，直接拼接 hello 和参数 username 即可。

```java
private Mono<String> sayHello(ServerRequest request) {
    try {
        // 应该是 username
        return Mono.just("Hello, " + request.queryParam("username").get());
    } catch (Exception e) {
        return Mono.error(e);
    }
}
```

因此，只要我们的请求中带了 username 参数，我们的请求就能正常返回。举个例子：我们请求“/hello?username=Tonni”，类似请求，我们总是能正常返回。

然而，如果我们的请求不带 username 参数，我们的请求就会抛出异常了。下面，我们来看看 Spring WebFlux 在哪里以及怎么重组代码来处理我们的异常。


## 3 方法级别处理异常

Mono 和 Flux API 中内置了两个关键运算符来处理方法级别的错误。我们简要探讨一下它们及其用法。

### 3.1 onErrorReturn 处理异常

当我们碰到异常的时候，我们可以用 onErrorReturn 来直接返回静态结果：

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return sayHello(request)
      .onErrorReturn("Hello Stranger")
      .flatMap(s -> ServerResponse.ok()
        .contentType(MediaType.TEXT_PLAIN)
        .bodyValue(s));
}
```

这里，每当有问题的连接函数抛出异常的时候，我们直接返回一个静态结果：“Hello Stranger”。

### 3.2 onErrorResume 处理异常

有三种使用 onErrorResume 处理异常的方式：

- 计算动态回调值
- 通过回调函数执行其他分支
- 捕获、包装并重新抛出错误，例如，作为自定义业务异常

让我们看看怎么计算值：

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return sayHello(request)
      .flatMap(s -> ServerResponse.ok()
        .contentType(MediaType.TEXT_PLAIN)
        .bodyValue(s))
      .onErrorResume(e -> Mono.just("Error " + e.getMessage())
        .flatMap(s -> ServerResponse.ok()
          .contentType(MediaType.TEXT_PLAIN)
          .bodyValue(s)));
}
```

这里，每当 sayHello 抛出异常的时候，我们返回一个 “Error + 异常信息（e.getMessage()）”。

接下来，我们看看当异常发生调用回调函数：

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return sayHello(request)
      .flatMap(s -> ServerResponse.ok()
        .contentType(MediaType.TEXT_PLAIN)
        .bodyValue(s))
      .onErrorResume(e -> sayHelloFallback()
        .flatMap(s -> ServerResponse.ok()
        .contentType(MediaType.TEXT_PLAIN)
        .bodyValue(s)));
}
```

这里，每当 sayHello 抛出异常的时候，我们执行一个其他函数 sayHelloFallback 。

最后，使用 onErrorResume 来捕获、包装并重新抛出错误，举例如：NameRequiredException

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return ServerResponse.ok()
      .body(sayHello(request)
      .onErrorResume(e -> Mono.error(new NameRequiredException(
        HttpStatus.BAD_REQUEST, 
        "username is required", e))), String.class);
}
```

这里，当 sayHello 抛出异常的时候，我们抛出一个定制异常 NameRequiredException，message是 “username is required”。

## 全局处理异常

目前为止，我们提供的所有示例都在方法级别上处理了错误处理。但是我们可以选择在全局层面处理异常。为此，我们只需要两步：

- 自定义一个全局错误响应属性
- 实现全局错误处理 handler

这样我们程序抛出的异常将会自动转换成 HTTP 状态和 JSON 错误体。我们只需要继承 DefaultErrorAttributes 类然后重写 getErrorAttributes 方法就可以自定义这些。

```java
public class GlobalErrorAttributes extends DefaultErrorAttributes{
    
    @Override
    public Map<String, Object> getErrorAttributes(ServerRequest request, 
      ErrorAttributeOptions options) {
        Map<String, Object> map = super.getErrorAttributes(
          request, options);
        map.put("status", HttpStatus.BAD_REQUEST);
        map.put("message", "username is required");
        return map;
    }

}
```

这里，当异常抛出是，我们想要返回 BAD_REQUEST 状态码和“username is required”错误信息作为错误属性的一部分。

然后，我们来实现全局错误处理 handler。

为此，Spring 提供了一个方便的 AbstractErrorWebExceptionHandler 类，供我们在处理全局错误时进行扩展和实现：

```java
@Component
@Order(-2)
public class GlobalErrorWebExceptionHandler extends 
    AbstractErrorWebExceptionHandler {

    // constructors

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(
      ErrorAttributes errorAttributes) {

        return RouterFunctions.route(
          RequestPredicates.all(), this::renderErrorResponse);
    }

    private Mono<ServerResponse> renderErrorResponse(
       ServerRequest request) {

       Map<String, Object> errorPropertiesMap = getErrorAttributes(request, 
         ErrorAttributeOptions.defaults());

       return ServerResponse.status(HttpStatus.BAD_REQUEST)
         .contentType(MediaType.APPLICATION_JSON)
         .body(BodyInserters.fromValue(errorPropertiesMap));
    }
}

```

在这个例子中，我们设置 handler 的 order 为 -2。这是为了给它一个比默认 handler，也就是 DefaultErrorWebExceptionHandler 一个更高的优先级，它设置的 order 为 -1。

errorAttributes 对象将是我们在 Web 异常处理程序的构造函数中传递的对象的精确副本。理想情况下，这应该是我们自定义的错误属性类。

然后我们明确生命了，我们希望将所有的异常处理路由到 renderErrorResponse() 中。

最后，我们获取了错误属性并插入到服务端响应体中。

然后这会生成一个 JSON 响应，其中包含了错误的详细信息，HTTP 状态、机器端的异常信息等。对于浏览器端，它有一个 “white-label”错误处理程序，可以以 HTML 形式呈现相同的数据，当然这个页面可以定制。

## 总结

在本文中，我们研究了在 Spring WebFlux 项目中处理异常的集中策略，并指出使用一个策略优于其他策略的地方。

源码在 github 上




原文：https://www.baeldung.com/spring-webflux-errors

