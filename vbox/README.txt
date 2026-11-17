# vbox-ios 精简配置包（iOS 15 巨魔方案）

本包由 build_vbox_config.py 从本工程已验证配置裁剪生成，供 vbox-ios 使用。

## 包含什么
- 3 个 drpy2 JS 站点：虎牙直播 / 斗鱼直播 / 儿童启蒙（兔小贝）
- 9 条直播订阅（lives，绝对 URL）
- live_checked.m3u：已验证直连直播源（可给 VLC/nPlayer 直接打开）

已剔除 45 个 JAR 站点：vbox-ios 是 JavaScriptCore/QuickJS 架构，没有 JVM，
跑不了依赖 fan.jar 的 csp_* 站点。这是 iOS 15 上的硬约束。

## 安装 vbox-ios
1. 从仓库 GitHub Actions 的最新成功运行下载 vbox-ipa 工件，解压得 vbox.ipa
   （需 GitHub 登录；也可在有 macOS 的环境自行编译）。
2. 用 TrollStore(巨魔)打开 vbox.ipa 安装。iOS 15 全系支持巨魔永久签名。
3. 打开 vbox → 设置 → 订阅 → 粘贴导入链接里的 URL → 添加订阅。

## 部署本配置包（关键）
vbox 按「订阅 URL 所在目录」解析相对路径，所以**必须把 bundle/ 整个目录
放到一个 HTTP(S) 静态服务上**，确保：
  <你的地址>/subscribe.json
  <你的地址>/lib/drpy2.min.js
三者能被浏览器直接访问。可用 GitHub Pages / Cloudflare Pages / 任意 nginx。

生成带真实地址的导入链接：
  python build_vbox_config.py --base-url https://你的域名/vbox

## 验证清单（按确定性排序）
- [ 必成 ] 直播分组能加载频道列表并播放（9 条绝对 URL，vbox 原生支持）
- [ 必成 ] 用 VLC/nPlayer 打开 live_checked.m3u 能播放（与 vbox 无关）
- [ 待验证 ] 订阅添加成功，显示 3 个站点 + 直播分组
- [ 待验证 ] 虎牙 / 斗鱼 / 兔小贝 能出分类、能搜索、能播放

## 已知限制（务必先读）
1. 3 个 JS 站依赖 drpy2 框架。本包提供的 lib/drpy2.min.js 已把原版 ES module
   转成全局脚本（剥 import/export + with(global) 绑定），本机做了语法检查，
   但 drpy2 的解析器(pdfh/pdfa/pd)在 init 阶段才初始化，实测在裸 JSC 环境会报
   "pdfh is not defined"。**这 3 个站能否跑通完全取决于 vbox 内置库是否补齐了
   drpy2 的运行时依赖，必须真机验证**——本包无法在 Windows 上闭环这一点。
2. 若 JS 站报错，按序尝试：
   a) 把 lib/drpy2.orig.min.js(原版 ES module) 覆盖 lib/drpy2.min.js；
   b) 确认 vbox 设置里已启用其自带的模板/cheerio/zhanyuan 库注入；
   c) 放弃 3 个 JS 站，专注直播分组（确定性可用）。
3. 45 个 JAR 站点在 iOS 15 无解；若要保留，需换 iOS 17+ 设备用映匣方案（见 docs/）。

一句话：本包**确定能用的是直播**；3 个 JS 站是"格式已就绪、待真机确认"。
