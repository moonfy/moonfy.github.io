[`返回首页`](../README.md)
#### 创建任务
通过 createTask方法 创建识别任务

请求节点：
国际节点
 https://api.1captcha.vip


请求地址： https://api.1captcha.vip/createTask

请求格式：POST application/json

RecaptchaV3 Enterprise 平均识别时间在130S左右
#### 对象结构
 

> ⚠️ v3 关键：求解用的 IP 与 UserAgent 必须与客户端提交表单时**一致**，否则分数会大幅下降（可能低至 0.1）。强烈建议在 task 中携带与业务侧相同的 proxyAddress/proxyPort 和 userAgent，并使用 sticky session。

| 属性 | 类型 | 必须 | 说明 |
|:--------------------------------------------:|:--------------------------------------------:|:--------------------------------------------:|:--------------------------------------------:|
| type              | string        | 是 | RecaptchaV3EnterpriseTaskProxyless   |
| websiteURL        | string        | 是 | ReCaptchaV3 企业版网页地址，一般固定值。   |
| websiteKey        | string        | 是 | ReCaptchaV3 企业版网站密钥，固定值。   |
| minScore          | string        | 是 | 期望最低分，如 "0.9"。   |
| pageAction        | string        | 是 | 页面 action，须与站点一致，如 "login"。   |
| recaptchaDataSValue | string      | 否 | 企业版常见的 data-s 值，站点可能动态生成/隐藏。   |
| userAgent         | string        | 否 | 浏览器 UA，v3 强烈建议携带，需与业务侧一致。   |
| cookies           | string        | 否 | 站点 cookie。   |
| proxyType         | string        | 否 | 代理类型：http / https / socks4 / socks5，默认 http。   |
| proxyAddress      | string        | 否 | 代理主机。   |
| proxyPort         | int           | 否 | 代理端口。   |
| proxyLogin        | string        | 否 | 代理用户名。   |
| proxyPassword     | string        | 否 | 代理密码。   |

> 说明：平台内部会以上游文档规定的 `RecaptchaV3EnterpriseTask` 类型提交，对外仍使用 `...Proxyless` 名称；`data-s` 直接以 `recaptchaDataSValue` 传给上游。企业版还需在上游 Options 中开启"通过 API 接收 proxy / userAgent"，否则传入的 proxy/UA 会被忽略。

#### 请求示例


```
{
    "clientKey": "cc9c18d3e263515c2c072b36a7125eecc078618f",
    "task": {
        "websiteURL": "https://example.com/submit",
        "websiteKey": "6LdKlZEpAAAAAAOQjzC2v_d36tWxCl6dWsozdSy9",
        "type": "RecaptchaV3EnterpriseTaskProxyless",
        "minScore": "0.9",
        "pageAction": "submit",
        "recaptchaDataSValue": "data-s值" // 可选，站点需要 data-s 时携带
    }
}
```

#### 响应示例

```
{
    "errorId": 0,
    "errorCode": "",
    "errorDescription": "",
    "taskId": "61138bb6-19fb-11ec-a9c8-0242ac110006" // 请记录此ID
}
```

#### 获取结果
使用 getTaskResult 方法获取识别结果

请求节点：
国际节点
 https://api.1captcha.vip

请求地址： https://api.1captcha.vip/getTaskResult

请求格式：POST application/json

根据系统负载，您将在  120s 的时间间隔内得到结果，300秒自动超时


```
{
    "clientKey":"cc9c18d3e263515c2c072b36a7125eecc078618f3",
    "taskId": "61138bb6-19fb-11ec-a9c8-0242ac110006"
}
```
#### 响应结果

| 属性 | 类型 |  说明 |
|:--------------------------------------------:|:--------------------------------------------:|:--------------------------------------------:|
| errorId              | Integer        | 错误提示： 0 - 没有错误，1 - 有错误   |
| errorCode            | string         | 错误代码   |
| errorDescription     | string         | 错误详细描述   |
| status               | string         | **processing** - 正在识别中，请3秒后重试。    **ready** - 识别完成，在solution参数中找到结果   |
| solution             | Object         | 识别结果，不同类型的任务结果会有所区别。   |
| gRecaptchaResponse   | string         | 识别结果：response值。一次性使用，有效期120s，建议在60s内使用。   |


#### 响应示例

```
{
    "errorId": 0,
    "errorCode": null,
    "errorDescription": null,
    "solution": {
        "gRecaptchaResponse": "03AGdBq25SxXT-pmSeBXjzScW-EiocHwwpwqtk1QXlJnGnU......"
    },
    "status": "ready"
}
```

#### 响应说明
- 识别成功：当errorId等于0 并且status等于 ready，结果在solution里面。

- 正在识别中：当errorId等于0 并且status等于 processing，请3秒后重试。

- 出错了：当errorId 大于0，请根据errorDescription了解出错误信息

- v3 拿到 gRecaptchaResponse 后，请用**与求解相同的代理和 UserAgent**回填表单提交，才能保证分数有效。
