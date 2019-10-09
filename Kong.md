# Kong

[https://docs.konghq.com](https://docs.konghq.com/)


### 如何限制访问Admin API和Kong Proxy
[Secure Admin API](https://docs.konghq.com/1.3.x/secure-admin-api/)
- 利用配置 admin_listen和proxy_listen限制请求来源
[Network Layer Access Restrictions](https://docs.konghq.com/1.3.x/secure-admin-api/#network-layer-access-restrictions)
- 限制admin_listen到localhost，配置route转发到admin api，然后为该路径添加plugin限制role

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0NDgzNDExNTQsLTkzNjA1NTc0Myw3Mz
A5OTgxMTZdfQ==
-->