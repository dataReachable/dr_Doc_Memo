# Axios

## axios 获取 message header 时大小写问题

### HTTP/1.1 规范

在 [RFC2616 的 4.2 节](https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html#sec4.2) 中表示 header message field 不区分大小写。
> Field names are case-insensitive. 

### axios 中的处理

在 axios 中并不是使用大写或小写都能获取相同的值，而是必须要使用小写字母 (`response.headers["lower-case"]`) 的形式。[axios issues #413](https://github.com/axios/axios/issues/413) 提到过此情况。