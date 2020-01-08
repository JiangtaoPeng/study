# 微服务
对复杂的业务系统统一做逻辑拆分，保持逻辑上的独立

# Kong
[教程](https://www.jianshu.com/p/a68e45bcadb6)
[教程2](https://segmentfault.com/a/1190000019857235)
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
eyJoaXN0b3J5IjpbMTAxOTQzMTAzMSw3MTk5MDk1NjUsNDY5NT
I1OTkwLC0xNTY1NjA0NzEsOTE1OTM2MjE0LC0xNjQyMzU2MzE2
LDE3ODg4NTQyNjIsLTE1NTc5ODAzOTUsLTE0NDgzNDExNTQsLT
kzNjA1NTc0Myw3MzA5OTgxMTZdfQ==
-->