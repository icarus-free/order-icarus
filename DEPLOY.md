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

## 3. 图片保存方式
本项目不使用 Firebase Storage，不需要升级项目，也不需要手动创建图片文件夹。

菜品和相册图片会先压缩到约 100KB，然后保存到 Firestore 的 `menu_room_images` 集合；菜单主文档只保存图片引用。第一次保存图片时，Firestore 会自动创建这个集合。

## 4. 安全规则
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
