# 用户模块
# com.tianque.userauth.client.api.UserDubboService

## 获取用户信息 getSimpleUserById

| API列表            | 类型     | 描述  |
| ------------- |:-------------:| -----:|
| getSimpleUserById      | 公开 | 用户ID获取用户信息 |
| findUserByUserName      | 公开      |   用户名获取用户信息 |
| getOrganizationById | 公开      |    组织机构ID获取组织机构 |
| getPropertyDomainById | 公开      |    数据字典ID获取数据字典项 |

| | |
| - | - |
| url | [https://~/v1/book](https://~/v1/books) | 
| method | GET | 

#### 参数

| name | type | mandatory | 描述 | 取值范围 |
| - | - | - | - | - |
| page | number | N | 当前面码 | - |
| id | string | Y | 书的编号 | - |

### 返回值

```javascript
{
  "success": true,
  "errCode": "xxx-xxx",
  "message": "",
  "data": null
}
```