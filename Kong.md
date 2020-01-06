# 微服务

# Kong

[https://docs.konghq.com](https://docs.konghq.com/)
[kong proxy docs/](https://git.102no.com/2019/06/12/kong-proxy-docs/)
https://git.102no.com/2019/06/12/kong-proxy-docs/

### 如何限制访问Admin API和Kong Proxy
[Secure Admin API](https://docs.konghq.com/1.3.x/secure-admin-api/)
- 利用配置 admin_listen和proxy_listen限制请求来源
[Network Layer Access Restrictions](https://docs.konghq.com/1.3.x/secure-admin-api/#network-layer-access-restrictions)
- 限制admin_listen到localhost，配置route转发到admin api，然后为该路径添加plugin限制访问
- 使用kong enterprise

### kong.db.services
:insert({host=test_host})
:select({id=svc.id}) (primary key)
:select_by_name("name_str")
:update({id=svc.id}, {change_table}) (primary key)
:update_by_name("svc_name", {change_table})
:delete()
:delete_by_name
:truncate()


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE1NjU2MDQ3MSw5MTU5MzYyMTQsLTE2ND
IzNTYzMTYsMTc4ODg1NDI2MiwtMTU1Nzk4MDM5NSwtMTQ0ODM0
MTE1NCwtOTM2MDU1NzQzLDczMDk5ODExNl19
-->