# Firebase 多人实时同步部署

## 1. 创建 Firebase 项目
1. 打开 https://console.firebase.google.com
2. 新建项目。
3. 添加 Web App，记下配置里的：
- `apiKey`
- `authDomain`
- `projectId`
- `appId`

## 2. 开启 Firestore
1. 进入 Build -> Firestore Database。
2. 点击 Create database。
3. 先选测试模式（后续可收紧规则）。

## 3. 安全规则（首版可用）
Firestore Rules 里先用：

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /menu_rooms/{docId} {
      allow read, write: if true;
    }
  }
}
```

## 4. 使用
1. 管理员打开 `index.html?admin=1&key=chef-admin`。
2. 在“云端配置（管理员）”填 4 个 Firebase 参数并点“连接实时同步”。
3. 点“复制成员链接”发给朋友。
4. 朋友点开即可点菜并提交；管理员页面实时显示同步结果。

## 5. 发布公网（免费）
可用 Vercel / Netlify / Cloudflare Pages，上传当前目录即可。

## 6. 说明
- 用户端不显示云端配置。
- 管理员端不显示点菜界面，只显示增菜和统计。
