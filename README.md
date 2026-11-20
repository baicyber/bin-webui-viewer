# Bin WebUI Viewer

上传固件 `.bin` / `.img`，直接在浏览器里显示固件内嵌的 `192.168.4.1` WebUI 页面。

纯前端、纯本地处理：文件不会上传到任何服务器。

![license](https://img.shields.io/badge/license-MIT-blue.svg)

## 它能做什么

- 扫描固件里的 gzip 压缩块（magic `1f 8b 08`）
- 用浏览器 `DecompressionStream` 流式解压，逐个候选寻找真正的 HTML 页面
- 把真实页面放进 iframe 渲染，而不是用模板伪装
- 注入通用 API shim，让离线页面也能交互：
  - 拦截 `fetch` / `XMLHttpRequest` 的相对地址和局域网请求
  - 模拟系统状态、Wi-Fi、OTA、管理状态、首次配置等接口
  - 关闭免责声明和首次配置弹窗
  - 兼容多种导航结构（`[data-page-target]`、`[data-app-page]`、`.navbtn` 等）
- 信号模拟：
  - Albert FSD 页面：CAN0 / CAN1 / CAN2 已连接、温度、运行时间、系统健康、网关等
  - 旧版 P4-CAN 页面：CAN1 / CAN2 / CAN3 `RUNNING + LIVE`、AP、S3XY、R79 注入状态等
- 中文 / English / Français 三语界面，语言偏好保存在 `localStorage`
- 可把提取出的页面在新窗口打开

## 使用方法

直接用新版 Chrome 或 Edge 打开 `index.html`，然后把固件 bin 拖到上传区即可。

也可以本地起一个静态服务：

```bash
python -m http.server 8000
```

然后访问 <http://localhost:8000/>。

> 需要浏览器支持 `DecompressionStream('gzip')`。旧版浏览器可能无法解压内嵌页面。

## 已验证的固件

- `P4-CAN-Universal-v3.35.2.zh-CN.FULLFLASH.REV3.X.merged.bin`（32MB 整片镜像）
- `P1.9.9.3-V1.3-c5740a36-EXPERIMENTAL-ota.bin`（Albert FSD OTA 包）

不同固件的页面结构和接口字段可能不同。本项目不是通用固件仿真器，而是尽量覆盖已见过的 WebUI 结构；遇到新页面可以补充 shim 字段。

## 工作原理

1. `File.arrayBuffer()` 读取整个 bin。
2. 扫描所有 gzip 头，逐块用 `DecompressionStream('gzip')` 解压。
   - 尾部可能有无关字节，因此使用 reader 分块读取，不依赖 `Response.arrayBuffer()`。
3. 找到以 `<!DOCTYPE html` 或 `<html` 开头且足够大的 HTML。
4. 在 `<head>` 里注入 `WEB_API_SHIM`。
5. 通过 `iframe.srcdoc` 渲染页面。

## 隐私与安全

- 不发起任何外部网络请求。
- 不上传、不保存你的固件文件。
- 提取出的页面来自你自己的 bin，项目仓库不包含任何固件或厂商页面资源。

## 许可证

MIT License，见 [LICENSE](LICENSE)。

本项目代码使用 MIT；从固件中提取出来的网页、图片、商标等资源仍归各自权利人所有，本项目不主张其所有权。

## 免责声明

这是一个离线查看/演示工具，与 Tesla、P4-CAN、Albert FSD 或其他固件作者无关。不要在真实车辆上依赖这里的模拟状态。
