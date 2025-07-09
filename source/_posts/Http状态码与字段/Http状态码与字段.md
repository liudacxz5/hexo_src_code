---
title: Http状态码与字段
date: 2025-05-22 09:12:01
updated: 2025-05-22 09:12:01
tags:
categories:
keywords:
description:
---
### HTTP 常见状态码

HTTP 状态码分为五类，以三位数字表示，首数字定义分类：

#### 1xx（信息响应）
- **100 Continue**：服务器已收到请求头，客户端应继续发送请求体。
- **101 Switching Protocols**：服务器根据客户端请求切换协议（如升级到 WebSocket）。

#### 2xx（成功）
- **200 OK**：请求成功，响应中包含请求的数据。
- **201 Created**：资源创建成功（常用于 POST/PUT 请求）。
- **204 No Content**：服务器成功处理请求，但无返回内容（如删除操作）。
- **206 Partial Content**：响应部分内容（用于分块下载或断点续传）。

#### 3xx（重定向）
- **301 Moved Permanently**：资源永久重定向，客户端应更新书签。
- **302 Found**：资源临时重定向，后续请求仍用原 URL。
- **304 Not Modified**：资源未修改，客户端可使用缓存（需配合 `ETag` 或 `Last-Modified` 使用）。
- **307 Temporary Redirect**：临时重定向，要求请求方法不变（与 302 类似但更严格）。
- **308 Permanent Redirect**：永久重定向，要求请求方法不变（与 301 类似但更严格）。

#### 4xx（客户端错误）
- **400 Bad Request**：请求格式错误（如参数缺失）。
- **401 Unauthorized**：需身份验证（如未提供 Token）。
- **403 Forbidden**：服务器拒绝请求（权限不足）。
- **404 Not Found**：资源不存在。
- **405 Method Not Allowed**：请求方法不被允许（响应头包含 `Allow` 字段提示允许的方法）。
- **408 Request Timeout**：服务器等待请求超时。
- **409 Conflict**：资源冲突（如重复提交）。
- **429 Too Many Requests**：请求频率过高（限流场景）。

#### 5xx（服务器错误）
- **500 Internal Server Error**：服务器内部错误（通用错误）。
- **501 Not Implemented**：服务器不支持请求的功能。
- **502 Bad Gateway**：网关或代理服务器收到无效响应。
- **503 Service Unavailable**：服务暂时不可用（如维护或过载）。
- **504 Gateway Timeout**：网关服务器未及时获取响应。

---

### HTTP 常见字段

#### 通用字段（请求/响应均可出现）
- **Cache-Control**：缓存策略（如 `max-age=3600`、`no-cache`）。
- **Connection**：控制连接（如 `keep-alive` 保持长连接）。
- **Date**：消息生成的日期时间。

#### 请求头字段
- **Host**：目标服务器的域名和端口（HTTP/1.1 必须字段）。
- **User-Agent**：客户端信息（如浏览器类型）。
- **Accept**：客户端可处理的媒体类型（如 `application/json`）。
- **Accept-Encoding**：支持的压缩方式（如 `gzip`）。
- **Authorization**：认证凭证（如 `Bearer <token>`）。
- **Cookie**：客户端发送的 Cookie 数据。
- **Content-Type**：请求体的数据类型（如 `application/json`）。
- **Referer**：当前请求的来源页面 URL。
- **Origin**：跨域请求的源地址（用于 CORS）。

#### 响应头字段
- **Content-Type**：响应体的数据类型（如 `text/html`）。
- **Content-Encoding**：响应体的压缩方式（如 `gzip`）。
- **Set-Cookie**：服务器设置的 Cookie。
- **Location**：重定向目标 URL（用于 3xx 状态码）。
- **Access-Control-Allow-Origin**：允许跨域请求的源（如 `*` 表示任意）。
- **ETag**：资源版本标识符（用于缓存验证）。
- **Last-Modified**：资源最后修改时间（缓存相关）。

#### 实体头字段（描述数据内容）
- **Content-Length**：响应体的字节数。
- **Content-Disposition**：指示如何处理数据（如 `attachment; filename="file.txt"` 触发下载）。

---

### 示例场景

#### 成功请求
```
200 OK
Content-Type: application/json
Cache-Control: max-age=3600
{
  "data": "Hello, World!"
}
```

#### 重定向
```
301 Moved Permanently
Location: https://new-site.com/resource
```

#### 客户端错误
```
404 Not Found
Content-Type: text/html
<h1>Page Not Found</h1>
```

#### 缓存验证
客户端请求头包含 `If-None-Match: "abc123"`，若资源未修改：
```
304 Not Modified
ETag: "abc123"
```

#### 跨域请求
请求头：
```
Origin: https://client.com
```
响应头：
```
Access-Control-Allow-Origin: https://client.com
```
