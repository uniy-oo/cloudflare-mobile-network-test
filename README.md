# Cloudflare 手机网络测试

一个无后端、无数据库、无外部依赖的 Cloudflare Workers 静态资源测试页。

## 目录

```text
cloudflare-mobile-network-test/
├─ public/
│  ├─ index.html
│  └─ test-video.mp4
├─ .gitignore
├─ package.json
├─ README.md
└─ wrangler.jsonc
```

## 本地预览

```powershell
npm install
npm run dev
```

## 重新生成测试视频

项目中的视频由原项目 `References/力量训练.mp4` 截取 60 秒生成，输出为 H.264、720×1280，音频为 AAC，目标约 10MB。`+faststart` 会把 MP4 的 moov 元数据移到文件开头，便于边下载边播放。

```powershell
ffmpeg -ss 0 -i "源视频.mp4" -t 60 -vf "scale=720:1280:force_original_aspect_ratio=decrease,pad=720:1280:(ow-iw)/2:(oh-ih)/2" -c:v libx264 -preset medium -profile:v high -level 3.1 -b:v 1250k -minrate 1250k -maxrate 1250k -bufsize 2500k -x264-params "nal-hrd=cbr:force-cfr=1" -pix_fmt yuv420p -c:a aac -b:a 128k -movflags +faststart -y public/test-video.mp4
```

## 建立私有 GitHub 仓库

项目已初始化为本地 Git 仓库。先在 GitHub 新建一个 **Private** 空仓库，不要自动添加 README，然后执行：

```powershell
git remote add origin https://github.com/你的账号/cloudflare-mobile-network-test.git
git branch -M main
git push -u origin main
```

也可在已登录 GitHub CLI 时执行：

```powershell
gh repo create cloudflare-mobile-network-test --private --source . --remote origin --push
```

## Cloudflare 控制台自动部署

1. 登录 Cloudflare 控制台，进入 **Workers & Pages**。
2. 点击 **Create application**。
3. 选择 **GitHub**；首次使用时授权 Cloudflare GitHub App，只授予这个私有仓库即可。
4. 选择 `cloudflare-mobile-network-test` 仓库。
5. 项目名称填写 `cloudflare-mobile-network-test`，必须与 `wrangler.jsonc` 中的 `name` 一致。
6. Production branch 选择 `main`，Root directory 填 `/`。
7. Build command 留空；Deploy command 填 `npx wrangler deploy`。
8. 点击 **Save and Deploy**。以后 push 到 `main` 会自动构建并部署。

部署成功后的默认地址格式：

```text
https://cloudflare-mobile-network-test.<你的-workers.dev-子域>.workers.dev
```

如先用 Wrangler 手动部署，可执行 `npm install`、`npx wrangler login`、`npm run deploy`。

## 手机测试清单

- 分别用移动、联通、电信蜂窝网络打开，关闭 Wi-Fi，并记录首次打开是否成功及所需时间。
- 检查北京时间是否每秒更新，时间和手机显示是否一致。
- 点击“点我测试”，检查是否立即有短音效；在微信内置浏览器也测试一次。
- 观察视频封面/时长信息出现速度，这是 `preload="metadata"` 的表现。
- 点击播放，记录首帧等待时间、是否频繁缓冲、声音是否正常、竖屏是否完整显示。
- 拖动视频进度，观察能否快速继续播放，以验证视频分段请求体验。
- 退出页面再打开一次，比较首次访问与缓存后的速度。
- 在早晚不同时段各测一次，并记录运营商、网络类型（4G/5G）、地点、成功与否及等待秒数。

`workers.dev` 在中国大陆不同运营商和时段的访问质量可能不同，因此测试结果应按网络、地点和时间分别记录。
