# vbox-ios 精简配置包（iOS 15 巨魔方案）

本包由 build_vbox_config.py + build_vbox_bundle.py 从本工程已验证配置裁剪生成，供 vbox-ios 使用。

## 包含什么
- 3 个 drpy2 JS 站点：虎牙直播 / 斗鱼直播 / 儿童启蒙（兔小贝）
- 9 条直播订阅（lives，绝对 URL）
- live_checked.m3u：已验证直连直播源（可给 VLC/nPlayer 直接打开）
- lib/drpy2.bundle.js：**单文件自包含全局版框架**（vbox 实际加载的就是它）

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
  <你的地址>/lib/drpy2.bundle.js
能被直接访问。可用 GitHub Pages / Cloudflare Pages / 任意 nginx。

本包已部署到：https://poenr.github.io/vbox/subscribe.json
（重新部署：python deploy_vbox_config.py）

## 验证清单（按确定性排序）
- [ 必成 ] 直播分组能加载频道列表并播放（9 条绝对 URL，vbox 原生支持）
- [ 必成 ] 用 VLC/nPlayer 打开 live_checked.m3u 能播放（与 vbox 无关）
- [ 待验证 ] 订阅添加成功，显示 3 个站点 + 直播分组
- [ 待验证 ] 虎牙 / 斗鱼 / 兔小贝 能出分类、能搜索、能播放

## 已知限制（务必先读）
1. 45 个 JAR 站点在 iOS 15 无解；若要保留，需换 iOS 17+ 设备用映匣方案（见 docs/）。
2. drpy2 框架已打包成单文件自包含全局脚本（lib/drpy2.bundle.js），
   内含 htmlParser(提供 pdfh/pdfa/pd) + cheerio + crypto-js + node-rsa + pako
   + 模板 + gbk + json5 + jinja，并在本机 node 沙箱里验证通过：
   框架加载无错、spider API 齐全、解析器实测正常、三个规则均可正常加载。
3. 剩下的不确定项只有**运行期**：三个站点的真实接口是否仍可用（站点可能改版），
   以及 vbox 的 http() 桥接在 JSC 与 QuickJS 两引擎下的行为差异。
   若 JS 站报错，先用 VLC/nPlayer 打开 live_checked.m3u 确认直播链路正常。
