> 这个目录是给「公开仓库」用的。里面只有一条 Actions 流水线，不含任何 SDK 源码或模型资源。

# ApkDemo iOS 构建流水线（公开仓库）

## 为什么拆成两个仓库

Cubism SDK 是商业授权，模型资源是版权素材，都不能公开。但 macOS runner 在**私有仓库**里按 10 倍倍率计费（2000 分钟/月免费额度 ≈ 只能跑 10 次），而**公开仓库**的 Actions 不占额度。

所以：

- **私有仓库** `apkdemo-ios-private`：存 `Core / Framework / Samples / ApkDemoAssets`（约 89MB）
- **公开仓库**（本目录）：只有这一份 workflow，运行时去私有仓库拉源码，编完把 ipa 回传到私有仓库的 Release

公开仓库的 Actions 页面不保留任何产物，ipa 只在私有仓库可见。

## 一次性配置（5 分钟）

### 1. 建私有仓库

在 GitHub 新建一个 **Private** 仓库，比如 `apkdemo-ios-private`，先不要勾 README / .gitignore / license。

把 `C:\live2d_build` 推上去：

```powershell
cd C:\live2d_build
git remote add origin https://github.com/<你的账号>/apkdemo-ios-private.git
git branch -M main
git push -u origin main
```

第一次 push 会弹浏览器让你登录，正常登录即可。

### 2. 建公开仓库

再新建一个 **Public** 仓库，比如 `apkdemo-ios-build`。

把这个目录里的内容推上去：

```powershell
cd "C:\Users\Administrator\Desktop\实用工具\live2D\apk转ipa逆向工程2\和爱撒娇的酶酶_Live2D工程\iOS构建流水线"
git init
git add -A
git commit -m "构建流水线"
git remote add origin https://github.com/<你的账号>/apkdemo-ios-build.git
git branch -M main
git push -u origin main
```

### 3. 造一个 PAT

GitHub → 右上角头像 → **Settings** → 最下面 **Developer settings** → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**

- Token name：随便，比如 `apkdemo-build`
- Expiration：90 天或自定义
- Repository access：**Only select repositories** → 选 `apkdemo-ios-private`
- Permissions → **Repository permissions** → **Contents** → 改成 **Read and write**

生成后复制那串 `github_pat_...`（只显示一次）。

### 4. 配 Secret

打开**公开仓库** → Settings → Secrets and variables → Actions → New repository secret，加两条：

| Name | Value |
| --- | --- |
| `PRIVATE_REPO` | `<你的账号>/apkdemo-ios-private` |
| `PRIVATE_TOKEN` | 刚复制的 `github_pat_...` |

## 跑构建

公开仓库 → Actions → 左侧 **Build iOS IPA** → 右边 **Run workflow** → 分支选 `main` → 运行。

一次完整编译大约 10~20 分钟。完成后：

- 点进那次运行，最后一步日志会打印 `已上传到 https://github.com/<账号>/apkdemo-ios-private/releases/tag/build-xxxx`
- 去私有仓库的 **Releases** 页面下载 `ApkDemo.ipa`

## 装到 iPhone

ipa 是**无签名**的，需要自己签：

1. 把 ipa 传到 iPhone（AirDrop / 文件 App / 数据线都行）
2. 用 **Esign** 打开 → 导入 ipa → 选证书签名 → 安装
   - 免费证书（Apple ID 自签）7 天有效，需要每 7 天重签一次
   - 付费开发者证书 1 年有效
3. 或者在电脑上用 **AltStore / Sideloadly** 直接装

## 改代码后怎么重新编

私有仓库改完 push，然后到公开仓库 Actions 手动点一次 Run workflow。流水线不会因为私有仓库的提交自动触发（跨仓库触发需要额外配 webhook），手动点一下最省事。
