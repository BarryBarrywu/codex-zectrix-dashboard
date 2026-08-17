

<p align="center">
  <img src="./assets/readme/hero.svg" width="100%" alt="Codex Dashboard for ZECTRIX：在 NOTE4 电子纸上显示 Codex 额度与任务动态">
</p>

把 Codex 的额度、重置时间和最近任务动态放到一块安静常亮的 ZECTRIX NOTE4 上。macOS companion 在本地读取状态、渲染 400×300 黑白画面，只在内容变化时推送。

## 实机效果

<p align="center">
  <img src="./assets/readme/device-photo.jpg" width="520" alt="ZECTRIX NOTE4 实机显示 Codex 剩余额度、重置时间和最近任务动态">
</p>

ZECTRIX NOTE4 上的实际运行效果。照片中的画面由 companion 读取当前 Codex 状态后生成并推送。

### 原始看板输出

<p align="center">
  <img src="./assets/readme/dashboard-preview.png" width="400" alt="NOTE4 示例画面，显示 Codex 剩余额度、重置时间和三条任务动态">
</p>

这张 400×300 图片由仓库内的固定 fixture 直接生成，没有使用设计稿代替真实输出。

- Codex 配额剩余百分比、使用量、窗口时长和重置时间
- 最多三条近期任务动态：`执行中`、`本轮完成`、`失败`或`已中断`
- 数据源暂时不可用时保留上次状态，并明确标记为可能过期
- 隐私模式隐藏任务标题，但保留状态和数量

## 工作方式

1. companion 通过官方独立 `codex app-server` 只读请求 `account/rateLimits/read`。
2. Codex hooks 提供执行事件；只读任务元数据用于匹配顶层任务标题。
3. 状态在 Mac 本地归一化，并渲染为适合 NOTE4 的 400×300 PNG。
4. 只有画面发生变化且通过推送节流后，才上传到选定的 ZECTRIX 页面。

NOTE4 的刷新频率表示设备检查云端新画面的频率，并不代表 companion 会按相同频率上传。Dashboard 仅在额度、任务、日期等可见信息变化时上传新画面，因此画面中的“上次同步”是最近一次成功上传时间，不会随设备的每分钟刷新自动更新。

程序只处理配额百分比、窗口与重置数据、任务标题，以及受限的任务活动状态。它不会保存或显示提示词、回复、推理、工具参数、计划文本或项目路径。

## 安装

需要 macOS、Codex 和一台 ZECTRIX NOTE4。正式插件已经内置 companion 二进制文件，不需要另外安装 Python、Node.js 或 Rust。

### 准备 ZECTRIX API Key

1. 打开并登录 [ZECTRIX 极趣云平台](https://cloud.zectrix.com)。
2. 进入「开放 API」，点击「创建 API Key」。
3. 妥善保存生成的 Key；它等同账号密码，请勿粘贴到聊天中或提交到代码仓库。

完整接口说明参见 [ZECTRIX 官方 API 文档](https://wiki.zectrix.com/zh/software/api-docs)。setup 会在本机读取 API Key，自动列出兼容的 NOTE4 设备，并让你选择要使用的持久页面，无需手动填写设备 MAC 地址或编辑配置文件。

```sh
codex plugin marketplace add BarryBarrywu/codex-zectrix-dashboard
codex plugin add codex-zectrix-dashboard@codex-zectrix-dashboard
```

安装后，在 Codex 中运行：

```text
$setup-zectrix-dashboard
```

setup 会在本机无回显终端中收集 ZECTRIX API Key，将它保存到 macOS Keychain，并在首次推送前展示预览和上传边界。只有你确认后，画面才会发送到设备。

### 更新

再次运行 `$setup-zectrix-dashboard`，按照 guarded update 流程操作。更新过程中需要 reload 或 restart Codex；不要直接删除旧插件缓存，也不要先运行 `codex plugin marketplace upgrade`。

## 不连接设备也能试用

从仓库生成固定示例：

```sh
cargo run --locked --release -- preview \
  --input fixtures/sample-dashboard.json \
  --output preview.png
```

读取当前 Codex 配额并生成实时预览：

```sh
cargo run --locked --release -- live-preview --output live-preview.png
```

`live-preview` 通过本地 `codex app-server` 读取配额，不会连接 Codex Desktop 或 ZECTRIX 设备。此处从源码运行需要 Rust；安装正式插件则不需要。

## 隐私边界

默认画面包含任务标题，渲染后的 PNG 会上传到 ZECTRIX Cloud。启用隐私模式后，标题会替换为“隐私任务”，但配额、任务状态和计数仍会出现在画面中。

API Key 由 macOS Keychain 保存。诊断输出只报告数据源状态，不输出账户、设备 ID、提示词、回复或其他原始载荷。

## 当前限制

- 仅支持 macOS 和 ZECTRIX NOTE4
- 不同步 Codex Desktop 的未读蓝点
- 暂不提供权威的“待你”“检查”状态或计划进度
- 不能修改、控制或结束 Codex 任务
- 完成 fixture 与 fake ZECTRIX 测试，不等于完成 NOTE4 实机验证

## 验证

```sh
cargo test --locked --all-features
./scripts/build-release.sh
./scripts/test-clean-install.sh
```

自动化测试、分发包验证和 NOTE4 实机验证应分别报告。设备选择、持久 `pageId`、首次推送、后续更新和实体屏幕可读性仍需单独进行实机检查。

## 关注我们

如果这个小工具对你有帮助，欢迎在 B 站关注我们：

- [极趣实验室（硬件官方）](https://space.bilibili.com/13131424)：了解 ZECTRIX 墨水屏、桌搭硬件和更多开源玩法。
- [最近使用（项目作者）](https://space.bilibili.com/217963572)：分享苹果生态、AI 工具与效率实践。

## License

[MIT](./LICENSE)
