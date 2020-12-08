# 错误码

方便面面试API采用标准HTTP错误码来表示接口的错误:


错误码 | 含义
---------- | -------
200 | OK -- 请求成功
400 | Bad Request -- 接口请求参数错误.
401 | Unauthorized -- 认证信息非法.
403 | Forbidden -- 所访问的资源拒绝访问.
404 | Not Found -- 访问的资源不存在.
405 | Method Not Allowed -- 错误的请求Method.
500 | Internal Server Error -- 服务器内部错误.
503 | Service Unavailable -- 服务器暂时不可用.
