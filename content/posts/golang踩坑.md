---
date: "2018-08-14 20:10:55"
title: golang踩坑
---

### 一

> x509 error when using HTTPS inside a Docker container

因为docker中没有CA证书。

普通的镜像解决办法

```
FROM ubuntu:14.04.1

RUN apt-get update
RUN apt-get install -y ca-certificates

CMD curl https://www.google.com
```

如果是alpine的参考这个：

```
FROM docker.finogeeks.club/base/alpine
MAINTAINER "zhuzhenfeng@finogeeks.club"

RUN set -ex \
    && apk add --no-cache ca-certificates

COPY src/wallet/wallet /opt/wallet

ENTRYPOINT /opt/wallet
```

### 二

> panic: runtime error: invalid memory address or nil pointer dereference
> [signal 0xb code=0x1 addr=0x38 pc=0x26df]

*"An error is returned if caused by client policy (such as CheckRedirect), or if there was an HTTP protocol error. A non-2xx response doesn't cause an error.*

*When err is nil, resp always contains a non-nil resp.Body."*

是http请求的时候，defer res.Body.Close()引起的，应该在err检查之后。

The `defer` only defers the function call. The field and method are accessed immediately.