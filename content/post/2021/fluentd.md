---
title: "fluentd"
date: "2021-04-06 13:48:03"
lastMod: "2021-04-06 13:48:03"
tags: ["fluentd", "k8s", "日志"]
---

综合需要考虑的问题

- [ ] 最好只处理指定namespace的日志
- [ ] 非标准日志的处理（SQL和Gin的日志），考虑忽略非标准日志
- [ ] 加入k8s metadata
- [ ] 按名称空间加日志（%{[k8s]}-%{+yyyy.MM.dd}）
- [ ] 删除不需要的字段
- [ ] 直连ES
- [ ] 消息主体替换根节点的message，level和time提到根节点

---

安装前系统配置

https://docs.fluentd.org/installation/before-install

安装指南

https://docs.fluentd.org/installation/install-by-rpm

官方的rpm方式安装非常简单，但是经常一天的尝试，失败了。最开始是找不到原因，es并没有收到任何日志，一直在查source或match的配置问题，后来发现是因为source配置的k8s容器日志路径/var/log/containers/*.log，td-agent并没有读取权限，属性用户和组都是root，但权限是777，手工读取日志文件，确实报没有权限，折腾了一天

选择了卸载rpm，改用docker方式运行，发现也不行，最终改成k8s方式运行可以！
期间遇到kibana一直提示创建索引问题，又折腾了大半天，结果按官方解答，重启kibana就解决了

中间考虑过logrus -> es，或logrus -> fluent的方案，但因会造成开发和生产环境的人为差异而放弃

关于filebeat和fluentd对选型

1. 就开发语言选择上，理论应该更倾向于filebeat(使用go，而fluent是ruby)
2. Github starts上，fluentd比filebeat多了不到1000，约10%（2021-4-7）
3. fluentd是CNCF的项目
4. fluentd在k8s官方有示例yaml（这个信息也许预示了k8s更倾向fluentd吧

最终选择了fluentd，虽然filebeat也不错：commit更多、tag更多、issue更多（这个姑且算理由吧）、开发成员更多，但是第3和第4条理由太强了～～～

---

折腾了几天filebeat，放弃了，两个都不折腾了！

filebeat放弃理由：对于json格式解析一直报错，都是按官方说法配置的！

---

经过反复折腾，重新整理思路，终于用filebeat搞定！不需要multiline，也不需要decode_json_fields，可以直接向ES发送数据，不需要Logstash，只是根据k8s namespace自动创建索引暂时未能实现，不过也不重要了！

k8s官方配置：https://github.com/elastic/beats/blob/master/deploy/kubernetes/filebeat-kubernetes.yaml

使用ecs-logrus：https://www.elastic.co/guide/en/ecs-logging/go-logrus/current/setup.html

官方文档：
https://www.elastic.co/guide/en/beats/filebeat/7.12/decode-json-fields.html
https://www.elastic.co/guide/en/beats/filebeat/7.12/multiline-examples.html
https://www.elastic.co/guide/en/beats/filebeat/7.12/filebeat-input-container.html