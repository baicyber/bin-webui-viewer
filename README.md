# Bin WebUI Viewer

上传固件 `.bin` / `.img`，直接在浏览器里显示固件内嵌的 `192.168.4.1` WebUI 页面。纯前端、纯本地处理，文件不会上传到任何服务器。

**语言 / Language / Langue:** [中文](#中文) · [English](#english) · [Français](#français)

![license](https://img.shields.io/badge/license-MIT-blue.svg)

---

<a id="中文"></a>

## 中文

上传固件 `.bin` / `.img`，直接在浏览器里显示固件内嵌的 `192.168.4.1` WebUI 页面。

纯前端、纯本地处理：文件不会上传到任何服务器。

### 它能做什么

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

### 使用方法

直接用新版 Chrome 或 Edge 打开 `index.html`，然后把固件 bin 拖到上传区即可。

也可以本地起一个静态服务：

```bash
python -m http.server 8000
```

然后访问 <http://localhost:8000/>。

> 需要浏览器支持 `DecompressionStream('gzip')`。旧版浏览器可能无法解压内嵌页面。

### 已验证的固件

- `P4-CAN-Universal-v3.35.2.zh-CN.FULLFLASH.REV3.X.merged.bin`（32MB 整片镜像）
- `P1.9.9.3-V1.3-c5740a36-EXPERIMENTAL-ota.bin`（Albert FSD OTA 包）

不同固件的页面结构和接口字段可能不同。本项目不是通用固件仿真器，而是尽量覆盖已见过的 WebUI 结构；遇到新页面可以补充 shim 字段。

### 工作原理

1. `File.arrayBuffer()` 读取整个 bin。
2. 扫描所有 gzip 头，逐块用 `DecompressionStream('gzip')` 解压。
   - 尾部可能有无关字节，因此使用 reader 分块读取，不依赖 `Response.arrayBuffer()`。
3. 找到以 `<!DOCTYPE html` 或 `<html` 开头且足够大的 HTML。
4. 在 `<head>` 里注入 `WEB_API_SHIM`。
5. 通过 `iframe.srcdoc` 渲染页面。

### 隐私与安全

- 不发起任何外部网络请求。
- 不上传、不保存你的固件文件。
- 提取出的页面来自你自己的 bin，项目仓库不包含任何固件或厂商页面资源。

### 许可证与免责声明

MIT License，见 [LICENSE](LICENSE)。项目代码使用 MIT；从固件中提取出来的网页、图片、商标等资源仍归各自权利人所有，本项目不主张其所有权。

这是一个离线查看/演示工具，与 Tesla、P4-CAN、Albert FSD 或其他固件作者无关。不要在真实车辆上依赖这里的模拟状态。

---

<a id="english"></a>

## English

Upload a firmware `.bin` / `.img` and view the embedded `192.168.4.1` WebUI directly in your browser. Fully front-end and processed locally — your file is never uploaded to any server.

### Features

- Scans the firmware for gzip-compressed blocks (magic `1f 8b 08`)
- Streams decompression with the browser's `DecompressionStream`, testing each candidate until the real HTML page is found
- Renders the actual page in an iframe instead of faking it with a template
- Injects a generic API shim so the offline page stays interactive:
  - Intercepts `fetch` / `XMLHttpRequest` calls to relative and LAN addresses
  - Mocks system status, Wi-Fi, OTA, management state, first-time setup, and more
  - Dismisses legal notices and first-run dialogs
  - Supports multiple navigation patterns (`[data-page-target]`, `[data-app-page]`, `.navbtn`, etc.)
- Simulated signals:
  - Albert FSD page: CAN0 / CAN1 / CAN2 connected, temperature, uptime, system health, gateway, etc.
  - Legacy P4-CAN page: CAN1 / CAN2 / CAN3 `RUNNING + LIVE`, AP, S3XY, R79 injection state, etc.
- Trilingual UI in 中文 / English / Français, with the language choice saved in `localStorage`
- Opens the extracted page in a new window

### Usage

Open `index.html` in a recent Chrome or Edge, then drag your firmware bin onto the upload area.

You can also serve it locally:

```bash
python -m http.server 8000
```

Then visit <http://localhost:8000/>.

> Requires a browser with `DecompressionStream('gzip')` support. Older browsers may not be able to decompress the embedded page.

### Verified firmware

- `P4-CAN-Universal-v3.35.2.zh-CN.FULLFLASH.REV3.X.merged.bin` (32 MB full-flash image)
- `P1.9.9.3-V1.3-c5740a36-EXPERIMENTAL-ota.bin` (Albert FSD OTA package)

Page structures and API fields vary between firmware versions. This is not a universal firmware emulator — it aims to cover the WebUI layouts seen so far; new shim fields can be added for new pages.

### How it works

1. Read the whole bin with `File.arrayBuffer()`.
2. Scan for gzip headers and decompress each block with `DecompressionStream('gzip')`.
   - Trailing junk bytes may be present, so it reads in chunks via a reader instead of relying on `Response.arrayBuffer()`.
3. Pick the HTML starting with `<!DOCTYPE html` or `<html` that is large enough.
4. Inject `WEB_API_SHIM` into `<head>`.
5. Render the page through `iframe.srcdoc`.

### Privacy & security

- No external network requests are made.
- Your firmware file is neither uploaded nor stored.
- The extracted page comes from your own bin; this repository contains no firmware or vendor page assets.

### License & disclaimer

MIT License, see [LICENSE](LICENSE). The project code is MIT-licensed; web pages, images, and trademarks extracted from firmware remain the property of their respective owners, and this project claims no ownership of them.

This is an offline viewing/demo tool, unaffiliated with Tesla, P4-CAN, Albert FSD, or any other firmware author. Do not rely on the simulated states here for a real vehicle.

---

<a id="français"></a>

## Français

Téléversez un firmware `.bin` / `.img` et affichez directement dans votre navigateur l'interface WebUI `192.168.4.1` intégrée au firmware. Tout se passe côté navigateur et localement — votre fichier n'est jamais envoyé à un serveur.

### Fonctionnalités

- Analyse le firmware à la recherche de blocs compressés en gzip (signature `1f 8b 08`)
- Décompresse en flux avec le `DecompressionStream` du navigateur, testant chaque candidat jusqu'à trouver la vraie page HTML
- Affiche la page réelle dans une iframe au lieu d'imiter une interface avec un modèle
- Injecte un shim d'API générique pour que la page hors ligne reste interactive :
  - Intercepte les appels `fetch` / `XMLHttpRequest` vers les adresses relatives et du réseau local
  - Simule l'état du système, le Wi-Fi, l'OTA, l'état d'administration, la configuration initiale, etc.
  - Ferme les avis légaux et les fenêtres de premier lancement
  - Prend en charge plusieurs types de navigation (`[data-page-target]`, `[data-app-page]`, `.navbtn`, etc.)
- Signaux simulés :
  - Page Albert FSD : CAN0 / CAN1 / CAN2 connectés, température, temps de fonctionnement, santé du système, passerelle, etc.
  - Ancienne page P4-CAN : CAN1 / CAN2 / CAN3 `RUNNING + LIVE`, AP, S3XY, état de l'injection R79, etc.
- Interface trilingue 中文 / English / Français, avec le choix de langue enregistré dans `localStorage`
- Ouvre la page extraite dans une nouvelle fenêtre

### Utilisation

Ouvrez `index.html` dans une version récente de Chrome ou Edge, puis déposez votre fichier bin dans la zone de téléversement.

Vous pouvez aussi le servir localement :

```bash
python -m http.server 8000
```

Puis visitez <http://localhost:8000/>.

> Nécessite un navigateur prenant en charge `DecompressionStream('gzip')`. Les anciens navigateurs risquent de ne pas pouvoir décompresser la page intégrée.

### Firmwares vérifiés

- `P4-CAN-Universal-v3.35.2.zh-CN.FULLFLASH.REV3.X.merged.bin` (image complète de 32 Mo)
- `P1.9.9.3-V1.3-c5740a36-EXPERIMENTAL-ota.bin` (paquet OTA Albert FSD)

La structure des pages et les champs d'API varient selon les versions de firmware. Ce projet n'est pas un émulateur de firmware universel — il vise à couvrir les interfaces WebUI rencontrées jusqu'ici ; de nouveaux champs de shim peuvent être ajoutés pour de nouvelles pages.

### Fonctionnement

1. Lit le bin entier avec `File.arrayBuffer()`.
2. Recherche les en-têtes gzip et décompresse chaque bloc avec `DecompressionStream('gzip')`.
   - Des octets parasites peuvent suivre les données ; la lecture se fait donc par morceaux via un reader, sans dépendre de `Response.arrayBuffer()`.
3. Sélectionne le HTML commençant par `<!DOCTYPE html` ou `<html` et suffisamment volumineux.
4. Injecte `WEB_API_SHIM` dans `<head>`.
5. Affiche la page via `iframe.srcdoc`.

### Confidentialité et sécurité

- Aucune requête réseau externe n'est effectuée.
- Votre fichier firmware n'est ni téléversé ni conservé.
- La page extraite provient de votre propre bin ; ce dépôt ne contient aucun firmware ni ressource de page propriétaire.

### Licence et avertissement

Licence MIT, voir [LICENSE](LICENSE). Le code du projet est sous licence MIT ; les pages web, images et marques extraites du firmware restent la propriété de leurs détenteurs respectifs, et ce projet n'en revendique aucune propriété.

Cet outil est destiné à la visualisation et à la démonstration hors ligne ; il n'est affilié ni à Tesla, ni à P4-CAN, ni à Albert FSD, ni à tout autre auteur de firmware. Ne vous fiez pas aux états simulés ici pour un véhicule réel.
