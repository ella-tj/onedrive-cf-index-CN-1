[![Hosted on Cloudflare Workers](https://img.shields.io/badge/Hosted%20on-CF%20Workers-f38020?logo=cloudflare&logoColor=f38020&labelColor=282d33)](https://drive.tcxz.cc/)

<h1>onedrive-cf-index(for 21vianet Users)</h1>

> Hint: This demostration is mainland-orited, so the default language：`中文`

Fork 自 [onedrive-cf-index，请 ⭐star 原项目 ](https://github.com/spencerwooo/onedrive-cf-index)

**源库已支持世纪互联，所以把此文档留作一个简陋的中文指示，希望能帮助到有需要的朋友**

## 演示地址

[☘️ Beet's OneDrive Index](https://drive.tcxz.cc/).

---

## 部署指南

### 准备：

#### 获得 OneDrive API 令牌

> 需自行保存的 key:
>
> - redirect_url
> - client_id
> - client_secret
> - refresh_token

1. `Azure.cn` [应用注册](https://portal.azure.cn/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) :

   - `任何组织目录(任何 Azure AD 目录 - 多租户)中的帐户`
   - `重定向 url` 键入并保存 `http://localhost/od-cf`

2. 保存 `client_id` ，添加并保存 `client_secret`

3. API 权限（Microsoft Graph）添加 `offline_access, Files.Read, Files.Read.All` 三个权限。

4. **使用 POSTMAN 获得 `refresh_token`**
   > 源库提供的 `https://heymind.github.io/tools/microsoft-graph-api-auth-cn` （世纪互联）尝试失败, 所以人工获取：

- 浏览器访问如下地址，需自行替换 `[client_id]`

  ```text
  https://login.chinacloudapi.cn/common/oauth2/v2.0/authorize?client_id=[client_id]&response_type=code&redirect_uri=http://localhost/od-cf&response_mode=query&scope=offline_access%20Files.Read%20Files.ReadWrite.All
  ```

  登陆后复制地址栏 `code=` 后字符串作为 `authorize_code`, 即为下面 json 里的 `code`

* 发送 POST 获得 `refresh_token`
  ，发送 post 请求，可以使用 postman 或 curl
  ，按照如下 `key：value` 在 POSTMAN 里填入(请自行填入`client_id` 、 `code` 、`client_secret`)

  ```bash
  {
    "client_id": "",
    "code": "",
    "redirect_uri": "http://localhost/od-cf",
    "grant_type": "authorization_code",
    "client_secret": ""
  }
  ```

  会成功返回 `refresh_token`, 参考下图，保存备用。

  ![](https://i.imgur.com/yhSl4gc.png)


    如出现问题，请参考 [azure doc](https://docs.azure.cn/zh-cn/active-directory/develop/v2-oauth2-auth-code-flow)

### 构建应用

clone （fork） 本项目，安装依赖：

```sh
# 安装Wrangler实现项目打包，附加从命令行部署到 cf worker（后项可选）
npm i -g @cloudflare/wrangler

# 安装依赖
npm i
```

使用 wrangler 前的准备:

1. 进入 https://dash.cloudflare.com/profile/api-tokens » 创建 cloudflare api 令牌（使用 Edit Cloudflare Workers 模板） » 登陆（示例如下）

   ```sh
   # 登陆 cloudflare
   wrangler config

   # 验证登录状态
   wrangler whoami
   ```

2. 创建 worker ，获取 Account ID
   在 cloudflare 创建新 worker，并在 worker 的 overview 页面右侧获取到 `Account ID`

   > 如果需要使用 cloudflare 里的域名绑定 worker，需要额外获取 `zone ID`, [参考文档](https://developers.cloudflare.com/workers/quickstart#account-id-and-zone-id)

3. 创建 KV 
```sh
# Create KV bucket
wrangler kv:namespace create "BUCKET"
```
复制返回对象于第四步中 `kv_namespaces` 数组中

4. 修改两个配置文件的几个选项

   - `wrangler.toml`

   ```toml

     # 刚刚创建的 worker 名
     name = "beet"

     # 获取的 Account ID
     account_id = ""

     # 如果使用了域名，需要添加 zone_id
     # zone_id = ""
     
     # KV
     kv_namespaces = [
            { binding = "BUCKET", id = "1cac5d222f33415dbd351720b331a5b5", preview_id = "ad5f8a197a784fef8cd2282a8997c1b0" }
     ]
   ```

   - `src/config/default.js`


    ```javascript
      client_id: '',

      // 网盘索引目录
      base = "",

      firebase_url = '',
    ```

### 使用 wrangler 上传密钥：

```sh
# 上传 refresh_token, client_secret, firebase_token 到 cloudflare 并加密

wrangler secret put REFRESH_TOKEN
# ... enter your refresh_token

wrangler secret put CLIENT_SECRET
# ... enter your client_secret
```

全部上传成功后，可以预览和发布：

```sh
wrangler preview

wrangler publish
```

> 如出现网络问题无法上传，可手动复制 `worker/worker.js` 到 cf worker

### 自定义

`themes/spencer.css` -> CSS

`src/render/htmlWrapper.js` -> HEAD FOOTER

`themes/prism-github.css` -> prism theme

`src/folderView.js` -> home intro

`src/config/default.js` -> toggle pagination feature
