# Firebase 多人实时同步部署

## 1. 创建 Firebase 项目
1. 打开 https://console.firebase.google.com
2. 新建项目。
3. 添加 Web App，记下配置里的：
- `apiKey`
- `authDomain`
- `projectId`
- `storageBucket`
- `appId`

## 2. 开启 Firestore
1. 进入 Build -> Firestore Database。
2. 点击 Create database。
3. 先选测试模式（后续可收紧规则）。

## 3. 开启 Storage
1. 进入 Build -> Storage。
2. 点击 Get started / 开始使用，创建默认 bucket。
3. bucket 名一般类似 `你的项目ID.firebasestorage.app`，填到管理员端的 `Firebase storageBucket`。

菜品图片会压缩到约 100KB，并上传到：

```txt
menu_rooms/main/dishes/{菜品ID}.jpg
menu_rooms/main/album/{相册记录ID}.jpg
```

Firestore 菜品和相册数据只保存图片下载 URL，避免单个菜单文档超过大小限制。按 100KB/张估算，15MB 大约可放 150 张图片，实际容量以 Firebase Storage 配额为准。

## 4. 安全规则（首版可用）
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

Storage Rules 里先用：

```txt
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /menu_rooms/main/dishes/{fileName} {
      allow read, write: if true;
    }
    match /menu_rooms/main/album/{fileName} {
      allow read, write: if true;
    }
  }
}
```

## 5. 使用
1. 管理员打开 `index.html?admin=1&key=chef-admin` 或通过页面里的“管理员入口”输入口令进入。
2. 在“云端配置（管理员）”填 Firebase 参数并点“连接实时同步”。
3. 点“复制成员链接”发给朋友。
4. 朋友点开即可点菜并提交；管理员页面实时显示同步结果。

## 6. 发布公网（免费）
可用 Vercel / Netlify / Cloudflare Pages，上传当前目录即可。

## 7. 说明
- 用户端不显示云端配置。
- 管理员端不显示点菜界面，只显示增菜和统计。
- 管理员端可切换“一般模式 / 庆祝模式”；庆祝模式下用户端显示生日祝福和只读料理相册。
