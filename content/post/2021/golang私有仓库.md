---
title: "goland私有仓库"
date: "2021-03-29 19:19:04"
lastMod: "2021-03-29 19:19:04"
tags: ["goland", "私有仓库"]
---

https://golang.org/cmd/go/#hdr-Modules__module_versions__and_more

https://golang.org/ref/mod#private-modules

https://golang.org/ref/mod#vcs-find

https://golang.org/doc/tutorial/create-module

https://sagikazarmark.hu/blog/vanity-import-paths-in-go/

https://medium.com/@dayakar88/a-guide-to-solve-no-go-import-meta-tags-for-private-repositories-with-go-modules-6b9237f9c9f

为了gitlab-ci在构建时拉取私有仓库依赖，构建工作使用的docker容器的dockerfile需要加上：
RUN git config --global url."git@192.168.1.10:".insteadOf "https://192.168.1.10/"
COPY gitlab-ci /root/.ssh/id_rsa
RUN chmod 600 /root/.ssh/id_rsa
RUN echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config

其中的id_rsa为部署密钥，需要在私有仓库中添加部署密钥
上述参考
https://cloud.tencent.com/developer/article/1602151
https://stackoverflow.com/questions/27500861/whats-the-proper-way-to-go-get-a-private-repository

使用本地仓库问题，这本不属于本文之列，为防忘记，记录下来

开发阶段，不建议go get @tag，而是go get @branch

版本有过调整，导致本地缓存一直依赖之前旧的更高正式版，需要使用go clean -modcache

https://zhuanlan.zhihu.com/p/103534192