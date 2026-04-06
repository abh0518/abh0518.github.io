---
layout: post
title: "Servlet filter code for Spring-Boot Gzip request"
date: 2017-09-27 13:30:55 +0900
categories: [프로그래밍]
---

web-server를 별도로 사용하지 않고 부트로면 서비스 할때 Gzip request 지원을 하기 위한 서블릿 필터..

```
package net.abh0518.spring.boot.test.filter;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.zip.GZIPInputStream;

@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class GzipRequestFilter implements Filter {

private static final Log logger = LogFactory.getLog(GzipRequestFilter.class);

@Override
public void init(FilterConfig filterConfig) throws ServletException {
logger.info("Init GzipRequestFilter");
}

@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
HttpServletRequest httpReq = (HttpServletRequest) request;
String encoding = httpReq.getHeader("Content-Encoding");
if(encoding != null){
if(encoding.toLowerCase().contains("gzip")){
request = new GZIPServletRequestWrapper(httpReq);
}
}

chain.doFilter(request, response);
}

@Override
public void destroy() {

}

private class GZIPServletRequestWrapper extends HttpServletRequestWrapper{

public GZIPServletRequestWrapper(HttpServletRequest request) {
super(request);
}

@Override
public ServletInputStream getInputStream() throws IOException {
return new GZIPServletInputStream(super.getInputStream());
}

@Override
public BufferedReader getReader() throws IOException {
return new BufferedReader(new InputStreamReader(new GZIPServletInputStream(super.getInputStream())));
}
}

private class GZIPServletInputStream extends ServletInputStream{
private InputStream input;

public GZIPServletInputStream(InputStream input) throws IOException {
this.input = new GZIPInputStream(input);
}

@Override
public int read() throws IOException {
return input.read();
}

@Override
public boolean isFinished() {
boolean finished = false;
try {
if(input.available() == 0){
finished = true;
}
} catch (IOException e) {
throw new RuntimeException(e);
}
return finished;
}

@Override
public boolean isReady() {
boolean ready = false;
try {
if(input.available() > 0){
ready = true;
}
} catch (IOException e) {
throw new RuntimeException(e);
}
return ready;
}

@Override
public void setReadListener(ReadListener listener) {}
}
}
```
