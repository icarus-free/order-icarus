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

Firestore 菜品和相册数据只保存图片下载 URL 或图片文档引用，避免单个菜单文档超过大小限制。如果 Storage 因 CORS、规则或超时暂不可用，页面会把压缩图片保存到 Firestore 的 `menu_room_images` 集合里，再在菜单文档里保存引用，换设备也能读取。

如果浏览器控制台出现 `blocked by CORS policy`，说明 Storage bucket 没有允许当前网页域名上传。可以用 Google Cloud SDK 给 bucket 添加 CORS；本页面会在 Storage 暂不可用时自动把压缩后的图片保存到 Firestore，保证上传功能不被阻断。

示例 `cors.json`：

```json
[
  {
    "origin": ["http://127.0.0.1:8765", "https://你的正式域名"],
    "method": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    "responseHeader": ["Content-Type", "Authorization", "x-goog-*"],
    "maxAgeSeconds": 3600
  }
]
```

设置命令：

```txt
gcloud storage buckets update gs://你的-bucket-名 --cors-file=cors.json
```

## 4. 安全规则（首版可用）
Firestore Rules 里先用：

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /menu_rooms/{docId} {
      allow read, write: if true;
    }
    match /menu_room_images/{imageId} {
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
3. 点“复制成员链接”发给朋友；这个链接只保留正常点菜功能。
4. 点“复制生日链接”给熙熙；这个链接会显示生日祝福、烟花动画和料理相册，并且可以编辑相册。
5. 朋友点开成员链接即可点菜并提交；管理员页面实时显示同步结果。

## 6. 发布公网（免费）
可用 Vercel / Netlify / Cloudflare Pages，上传当前目录即可。

## 7. 说明
- 用户端不显示云端配置。
- 管理员端不显示点菜界面，只显示增菜和统计。
- 普通成员链接不显示生日祝福和相册；生日祝福、烟花动画和相册只显示在专属生日链接中。
- 管理员端和生日专属链接都可以编辑料理相册；普通成员链接不能编辑相册。
