---
title: CORS Filter
description: Configure Cross-Origin Resource Sharing (CORS) for Ant Media Server to control which domains can access streams.
keywords: [CORS Filter, Cross-Origin Resource Sharing, Stream Security, Ant Media Server Documentation, Ant Media Server Tutorials]
sidebar_position: 7
---

# CORS Filter

By default, the **CORS (Cross-Origin Resource Sharing)** filter is enabled and accepts requests from all origins (`*`).

## Application-Level CORS Configuration

To customize the CORS filter at the application level, access:

```
/usr/local/antmedia/webapps/{AppName}/WEB-INF/web.xml
```

### Default Configuration (Allow All Origins)

```xml
<filter>
  <filter-name>CorsFilter</filter-name>
  <filter-class>io.antmedia.filter.CorsHeaderFilter</filter-class>
  <init-param>
    <param-name>cors.allowed.origins</param-name>
    <param-value>*</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.methods</param-name>
    <param-value>GET,POST,HEAD,OPTIONS,PUT,DELETE</param-value>
  </init-param>
  <!-- cors.allowed.origins -> * and credentials are not supported at the same time.
  If you set cors.allowed.origins to specific domains and support credentials, open the below lines
  <init-param>
    <param-name>cors.support.credentials</param-name>
    <param-value>true</param-value>
  </init-param>
  -->
  <init-param>
    <param-name>cors.allowed.headers</param-name>
    <param-value>Accept, Origin, X-Requested-With, Access-Control-Request-Headers, Content-Type, Access-Control-Request-Method, Authorization</param-value>
  </init-param>
  <async-supported>true</async-supported>
</filter>
<filter-mapping>
  <filter-name>CorsFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Restrict to Specific Domains

If you are experiencing CORS errors when playing streams or integrating the player into another domain, allow the specific domain and uncomment the credentials section:

```xml
<filter>
  <filter-name>CorsFilter</filter-name>
  <filter-class>io.antmedia.filter.CorsHeaderFilter</filter-class>
  <init-param>
    <param-name>cors.allowed.origins</param-name>
    <param-value>https://domain:port</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.methods</param-name>
    <param-value>GET,POST,HEAD,OPTIONS,PUT,DELETE</param-value>
  </init-param>
  <init-param>
    <param-name>cors.support.credentials</param-name>
    <param-value>true</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.headers</param-name>
    <param-value>Accept, Origin, X-Requested-With, Access-Control-Request-Headers, Content-Type, Access-Control-Request-Method, Authorization</param-value>
  </init-param>
  <async-supported>true</async-supported>
</filter>
<filter-mapping>
  <filter-name>CorsFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

## Root-Level CORS Configuration

To customize the CORS filter in the root folder, edit:

```
/usr/local/antmedia/webapps/root/WEB-INF/web.xml
```

```xml
<filter>
  <filter-name>CorsFilter</filter-name>
  <filter-class>io.antmedia.filter.CorsHeaderFilter</filter-class>
  <init-param>
    <param-name>cors.allowed.origins</param-name>
    <param-value>*</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.methods</param-name>
    <param-value>GET,POST,HEAD,OPTIONS,PUT,DELETE</param-value>
  </init-param>
  <init-param>
    <param-name>cors.allowed.headers</param-name>
    <param-value>Accept, Origin, X-Requested-With, Access-Control-Request-Headers, Content-Type, Access-Control-Request-Method, Authorization, ProxyAuthorization</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>CorsFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

:::info
To learn more about CORS filter, check out [Tomcat CORS Filter](https://tomcat.apache.org/tomcat-8.0-doc/api/index.html?org/apache/catalina/filters/CorsFilter.html).
:::
