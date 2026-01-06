# Flutter Download - 抖音解析下载工具

<p align="center">
  <img src="https://rin-img.liyunfei.eu.org/douyin-hono-images/u1_1767670123250_uwjgkw.jpg" width="120" alt="App Logo"/>
</p>

<p align="center">
  <a href="https://github.com/42419/flutter_download">
    <img src="https://img.shields.io/github/stars/42419/flutter_download?style=social" alt="GitHub stars"/>
  </a>
  <a href="https://github.com/42419/flutter_download/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License"/>
  </a>
  <a href="https://github.com/42419/flutter_download">
    <img src="https://img.shields.io/badge/platform-Web%20%7C%20Mobile-blue.svg" alt="Platform"/>
  </a>
</p>

一个现代化的 **Flutter Web** 应用，提供抖音视频解析和无水印下载功能。采用优雅的 Material Design 3 设计语言，集成了流畅的动画效果、响应式布局和完整的用户系统。

## ✨ 项目特色

### 🎨 视觉特效

- **液态流体动画** - 登录页面的双球漂浮动画，使用 OpenSimplexNoise 算法和陀螺仪传感器实现
- **玻璃拟态设计** - 主页底部导航栏采用 BackdropFilter 实现毛玻璃效果
- **滚动模糊导航** - 关于页面顶部导航栏随滚动动态增加模糊效果
- **复古风格 UI** - 统一的边框设计（3px 宽度），卡片阴影和 hover 效果

### 📱 响应式设计

- **桌面/移动适配** - 大屏使用侧边导航栏（支持拖拽调整宽度 72px-200px），小屏使用底部导航
- **智能横竖屏** - 视频展示根据宽高自动调整，竖屏视频使用模糊背景+居中显示
- **优雅降级** - 渐进式 Token 验证策略，提升用户体验

### 🔐 认证优化

- **智能状态管理** - Token 有效期 7 天，通过 SharedPreferences 本地存储
- **乐观策略** - 有 Token 就先进入主页，避免不必要的等待
- **自动重定向** - 未登录用户自动跳转到登录页
- **自动刷新** - Token 快过期时智能验证

### 🚀 部署优化

- **自动哈希处理** - 构建后自动为 JS/CSS 文件添加内容哈希，解决缓存问题
- **SPA 支持** - 自动生成 404.html 支持客户端路由
- **Cloudflare 优化** - 支持 Cloudflare Pages 和 Workers 部署
- **版本追踪** - 构建时自动注入 Git Commit Hash 用于调试

## 🛠️ 技术栈

### 前端 (Flutter Web v1.0.3+1)

| 包名                          | 版本     | 用途                             |
| ----------------------------- | -------- | -------------------------------- |
| `sensors_plus`                | ^7.0.0   | 陀螺仪传感器支持（液态球动画）   |
| `open_simplex_noise`          | ^2.3.1   | Simplex 噪声生成（流体效果）     |
| `dio`                         | ^5.9.0   | HTTP 客户端（API 调用）          |
| `shared_preferences`          | ^2.5.4   | 本地数据存储（Token、用户信息）  |
| `url_launcher`                | ^6.3.2   | 外部链接处理（GitHub、邮件反馈） |
| `device_info_plus`            | ^12.3.0  | 跨平台设备信息获取               |
| `image_picker`                | ^1.2.1   | 头像选择                         |
| `image_cropper`               | ^11.0.0  | 头像裁剪                         |
| `adaptive_theme`              | ^3.7.2   | 主题管理（浅色/深色/跟随系统）   |
| `flutter_markdown`            | ^0.7.7+1 | Markdown 渲染                    |
| `gal`                         | ^2.3.2   | 相册保存（下载视频）             |
| `permission_handler`          | ^12.0.1  | 权限处理                         |
| `flutter_local_notifications` | ^19.5.0  | 本地通知（下载进度）             |
| `crypto`                      | ^3.0.7   | 密码 SHA-256 加密                |

### 后端 (Cloudflare Workers + Hono)

| 技术                 | 版本/说明     | 用途                   |
| -------------------- | ------------- | ---------------------- |
| `Hono`               | ^4.0.0        | 轻量级 Web 框架        |
| `Cloudflare Workers` | -             | 无服务器边缘计算       |
| `Cloudflare D1`      | SQLite 数据库 | 用户数据、解析历史存储 |
| `Cloudflare KV`      | 键值存储      | Session Token 缓存     |
| `Cloudflare R2`      | 对象存储      | 头像文件存储           |
| `TypeScript`         | -             | 类型安全的 API 开发    |
| `wrangler`           | ^3.28.0       | Cloudflare CLI 工具    |

## 🎯 核心功能详解

### 1. 视频解析与下载

**前端流程**:

```
用户输入链接 → Dio 请求后端 → 解析视频数据 → 展示详情 → 下载到相册
```

**后端流程** (BFF 模式):

```
Hono 接收请求 → 重定向解析 → 提取 aweme_id → 第三方 API → 保存历史 → 返回数据
```

**特性**:

- ✅ 无水印视频下载
- ✅ 多清晰度选择
- ✅ 中文文件名支持
- ✅ 流式下载，节省内存
- ✅ 自动保存到相册（Gal 包）

### 2. 用户认证系统

**前端 (Flutter):**

```dart
// Token 管理 - 7 天有效期
final prefs = await SharedPreferences.getInstance();
await prefs.setString('token', token);
await prefs.setString('user_info', jsonEncode(userInfo));
```

**后端 (Hono):**

```typescript
// 生成 Token 并存储到 KV
const token = crypto.randomUUID();
await c.env.AUTH_KV.put(
  `session:${token}`,
  JSON.stringify({ id: row.id, email: row.email }),
  { expirationTtl: 60 * 60 * 24 * 7 } // 7天有效期
);
```

**特点:**

- 乐观策略：有 Token 直接进入主页
- 智能验证：Token 快过期时才验证有效性
- 自动重定向：未登录用户自动跳转
- **安全**: 密码使用 SHA-256 加密存储

### 3. 公告管理系统

**功能**:

- **状态机**: 草稿 → 待发布 → 活跃 → 已过期 → 已归档
- **乐观锁**: 使用版本号防止并发冲突
- **回收站**: 软删除，支持恢复
- **权限控制**: 仅管理员可操作

**UI 特色**:

- 自定义头部设计（ANNO SYSTEM / RECYCLE BIN / SYSTEM NOTICE）
- 复古边框风格
- 优先级和状态徽章

### 4. 历史记录功能

- 自动保存解析记录
- 分页查询（支持无限滚动）
- 按用户隔离（私有历史表）
- 支持删除单条/清空全部

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 🙏 致谢

- [Flutter Team](https://flutter.dev/) - 优秀的跨平台框架
- [Hono](https://hono.dev/) - 轻量级 Web 框架
- [Cloudflare](https://www.cloudflare.com/) - 优秀的部署平台

---

**注意**: 本项目仅供学习和研究使用，请遵守相关平台的服务条款和版权规定。

## 📞 联系方式

- **GitHub**: [@42419](https://github.com/42419)
- **Issues**: [GitHub Issues](https://github.com/42419/fkdouyin_remake_app/issues)

---

<p align="center">
  Made with ❤️ using Flutter & Cloudflare
</p>
