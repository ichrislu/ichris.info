---
title: "添加nginx stub_status"
date: "2019-11-19 10:11:03"
lastMod: "2019-11-19 10:11:03"
tags: ["nginx", "stub_status"]
---

获取Nginx运行状态数据，添加以下配置：

```conf
location /stub_status {
	stub_status on;

	# 密码验证
	auth_basic "stub status";
	auth_basic_user_file /etc/nginx/.htpasswd;

	# 限制IP访问
	allow	?.?.?.?;
	deny	all;

	access_log off;
}
```

密码验证的账号和密码生成：

```shell
echo "用户名:$(openssl passwd -crypt '<密码>')" >> .htpasswd
```

