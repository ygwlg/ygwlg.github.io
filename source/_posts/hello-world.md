---
title: Hello Hexo
tags:
    - Hexo
---


### 环境

+ aliyun 2vCPU/2GB/50GB/X64
+ Ubuntu 20.04.6 LTS
+ apt 2.0.10(amd64)

#### 1. 部署nodejs

```bash
wget https://nodejs.org/dist/v18.20.6/node-v18.20.6-linux-x64.tar.xz
tar -xvf ...

# add to ~/.bashrc
# export PATH=$PATH:...
source ~/.bashrc
```

部署结果

```bash
# node -v
v18.20.6
# npm -v
10.8.2
```

#### 2. 部署hexo

```bash
npm install hexo-cli
```

部署结果

```bash
# hexo -v
hexo-cli: 4.3.2
os: linux 5.4.0-182-generic Ubuntu 20.04.6 LTS (Focal Fossa)
node: 18.20.6
acorn: 8.13.0
ada: 2.8.0
ares: 1.29.0
base64: 0.5.2
brotli: 1.1.0
cjs_module_lexer: 1.2.2
cldr: 44.1
icu: 74.2
llhttp: 6.1.1
modules: 108
napi: 9
nghttp2: 1.61.0
nghttp3: 0.7.0
ngtcp2: 1.3.0
openssl: 3.0.15+quic
simdutf: 5.6.0
tz: 2024a
undici: 5.28.5
unicode: 15.1
uv: 1.44.2
uvwasi: 0.0.19
v8: 10.2.154.26-node.37
zlib: 1.3.0.1-motley
```

### 主题

[hexo 主题集合](https://hexo.io/themes/)

本站使用[Hexo-Theme-Async](https://hexo-theme-async.imalun.com/)主题

### 运行

```bash
hexo server -p 80 -i 0.0.0.0
```

### 部署github page