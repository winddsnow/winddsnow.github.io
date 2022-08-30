---
title: Ribbin负载均衡源码原理与源码调试
categories:
  - 编程
tags:
  - Java
  - Spring
  - Ribbon
  - debug调试
date: 2022-08-30 11:26:09
---

# Ribbon负载均衡流程
1. 发起请求http://服务名xxx/服务接口名xxx/参数
2. 被LoadBalancerInterceptor负载均衡拦截器拦截
3. LoadBalancerInterceptor调用RibbonLoadBanlancerClient真实执行
4. RibbonLoadBanlancerClient获取url中的服务参数与服务接口名
5. RibbonLoadBanlancerClient调用DynamicServerListLoadBalancer，从eurekaserver中得到服务列表
6. DynamicServerListLoadBalancer调用IRule负载均衡得到某个服务真实服务地址，传递给RibbonLoadBanlancerClient
7. RibbonLoadBanlancerClient修改url为真实服务地址，发起请求http://localhost:18081xx服务端口/服务接口/参数
8. 对应客户端响应请求，完成整个流程。

## 使用断点调试
### LoadBalancerInterceptor
由流程可知，请求将会被LoadBalancerInterceptor拦截器拦截，我们在LoadBalancerInterceptor类下的intercept下打下断点，可以观察到ribbin请求到达拦截器，被拦截器拦截到，intercept调用execute方法去获取真实url及负载均衡
```java
@Override
	public ClientHttpResponse intercept(final HttpRequest request, final byte[] body,
			final ClientHttpRequestExecution execution) throws IOException {
		final URI originalUri = request.getURI();
		String serviceName = originalUri.getHost();
		Assert.state(serviceName != null,
				"Request URI does not contain a valid hostname: " + originalUri);
		//调用execute方法，并将服务地址等参数传递过去
		return this.loadBalancer.execute(serviceName,
				this.requestFactory.createRequest(request, body, execution));
	}
```

### execute
execute的类LoadBalancerClient是一个接口，我们在它的实现类RibbonLoadBalancerClient上的execute方法打上断点，可以观察到请求转发过来
```java
public <T> T execute(String serviceId, LoadBalancerRequest<T> request, Object hint)
			throws IOException {
		//getLoadBalance这个方法就是在eurekaserver中得到服务列表，并根据指定负载均衡算法得到一个真实可用的服务地址返回，默认算法是ZoneAvoidanceRule，区域轮询
		ILoadBalancer loadBalancer = getLoadBalancer(serviceId);
		Server server = getServer(loadBalancer, hint);
		if (server == null) {
			throw new IllegalStateException("No instances available for " + serviceId);
		}
		RibbonServer ribbonServer = new RibbonServer(serviceId, server,
				isSecure(server, serviceId),
				serverIntrospector(serviceId).getMetadata(server));

		return execute(serviceId, ribbonServer, request);
	}
```
### HttpURLConnection
最后在HttpURLConnection中打上断点，可以观察到，发起的http请求的地址已经是负载均衡过的可用真实地址
```java
protected HttpURLConnection (URL u) {
		//得到u: http://xxxx:180xx/user/1
        super(u);
    }
```
