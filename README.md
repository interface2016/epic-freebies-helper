# Epic Awesome Gamer

自动领取 Epic Games 每周免费游戏，支持 GitHub Actions 定时运行，验证码识别支持 `Gemini / AiHubMix / GLM`。

这份文档分两部分：

- 前半部分面向普通用户：照着做就能跑起来。
- 最后是开发者附录：解释项目结构，以及这次适配里踩过的坑。

## 这是什么

这个项目会做 4 件事：

1. 登录 Epic 账号。
2. 拉取当周免费游戏。
3. 自动进入商品页并完成领取。
4. 在需要时处理 hCaptcha 和结账页二次安全校验。

默认最推荐的运行方式是 GitHub Actions：

- 不需要自己开电脑挂机。
- 仓库已经带好工作流。
- 适合绝大多数用户。

## 使用前先确认

开始之前，请先确认下面 4 件事：

| 项目 | 是否必须 | 说明 |
| --- | --- | --- |
| Epic 账号邮箱 | 是 | 用来登录 Epic |
| Epic 账号密码 | 是 | 用来登录 Epic |
| 关闭 2FA | 是 | 必须关闭邮箱 / 短信 / App 二次验证 |
| 多模态模型 API Key | 是 | 用来识别 hCaptcha |

### 为什么必须关闭 2FA

这个项目运行在无头自动化环境里。  
如果账号开启了邮箱验证码、短信验证码、验证器 App 等二次验证，流程通常会被卡住。

建议到 Epic 账号安全设置里，把这些验证方式先全部关闭，再运行项目。

## 5 分钟快速开始

### 第 1 步：Fork 仓库

把这个仓库 Fork 到你自己的 GitHub 账号下。

建议 Fork 完后立刻改成私有仓库，原因很简单：

- 更适合保存自己的 Actions 配置。
- 不容易误暴露运行记录。
- 后续改 Secrets 更安心。

### 第 2 步：打开 GitHub Actions

进入你自己的 Fork 仓库后：

1. 打开 `Actions`
2. 如果 GitHub 提示是否启用工作流，点击启用

这个仓库里的工作流名称是：

- `Epic Awesome Gamer (Scheduled)`

它默认每天执行一次，你也可以手动触发。

### 第 3 步：配置 Secrets

进入：

- `Settings`
- `Secrets and variables`
- `Actions`

然后按下面的表格添加。

## 推荐配置：GLM

如果你想少折腾，优先推荐 GLM。

至少需要这些 Secrets：

| Secret | 必填 | 示例 | 说明 |
| --- | --- | --- | --- |
| `EPIC_EMAIL` | 是 | `you@example.com` | Epic 登录邮箱 |
| `EPIC_PASSWORD` | 是 | `your-password` | Epic 登录密码 |
| `LLM_PROVIDER` | 是 | `glm` | 明确指定使用 GLM |
| `GLM_API_KEY` | 是 | `xxxxxxxx` | 智谱 API Key |
| `GLM_MODEL` | 是 | `glm-4.6v-flash` | 视觉模型名 |

通常不需要手动填写下面这些，留空即可：

- `GLM_BASE_URL`
- `CHALLENGE_CLASSIFIER_MODEL`
- `IMAGE_CLASSIFIER_MODEL`
- `SPATIAL_POINT_REASONER_MODEL`
- `SPATIAL_PATH_REASONER_MODEL`

原因是当前代码会自动兜底：

- `GLM_BASE_URL` 默认走 `https://open.bigmodel.cn/api/paas/v4`
- 4 个任务模型默认跟随 `GLM_MODEL`

### 一份最小可用的 GLM Secrets 清单

```text
EPIC_EMAIL=你的 Epic 邮箱
EPIC_PASSWORD=你的 Epic 密码
LLM_PROVIDER=glm
GLM_API_KEY=你的智谱 API Key
GLM_MODEL=glm-4.6v-flash
```

如果你已经验证过其他模型名在自己账号下可用，也可以继续用你自己的模型名。  
代码不会强制限制模型名，最终以智谱接口是否接受为准。

## 可选配置：Gemini / AiHubMix

如果你不用 GLM，也可以用 Gemini 或 AiHubMix。

需要这些 Secrets：

| Secret | 必填 | 示例 | 说明 |
| --- | --- | --- | --- |
| `EPIC_EMAIL` | 是 | `you@example.com` | Epic 登录邮箱 |
| `EPIC_PASSWORD` | 是 | `your-password` | Epic 登录密码 |
| `LLM_PROVIDER` | 建议 | `gemini` | 建议显式指定 |
| `GEMINI_API_KEY` | 是 | `xxxxxxxx` | Gemini 或 AiHubMix Key |
| `GEMINI_BASE_URL` | 否 | `https://aihubmix.com` | 不填就走默认值 |
| `GEMINI_MODEL` | 否 | `gemini-2.5-pro` | 不填就走默认值 |

## 第 4 步：手动跑一次

进入 `Actions` 页面后：

1. 选择 `Epic Awesome Gamer (Scheduled)`
2. 点击 `Run workflow`
3. 等待运行完成

第一次手动跑的目的很明确：

- 检查 Secrets 是否配对了
- 检查 Epic 账号能不能正常登录
- 检查模型是否真的能识别当前验证码

## 第 5 步：看成功没有

如果领取成功，常见日志通常会接近下面这种状态：

```text
Login success
Right account validation success
Authentication completed
Starting free games collection process
All week-free games are already in the library
```

注意最后这句：

- `All week-free games are already in the library`

它的意思是：

- 当前周免已经在库里
- 这次运行没有发现还没领的内容

如果是第一次运行，也可能会看到加购、结账、验证码和确认领取相关日志，这都正常。

## 运行后去哪里看截图和日志

每次 GitHub Actions 运行结束后，工作流会自动上传两个 artifact：

- `epic-runtime-<run_id>`
- `epic-logs-<run_id>`

下载位置：

1. 进入本次 Actions 运行页面
2. 拉到页面底部
3. 找到 `Artifacts`
4. 下载 zip 文件

下载后建议先看这两个目录：

- `app/volumes/runtime/purchase_debug/`
- `app/volumes/logs/`

如果你碰到“明明没领到，但日志里看不明白”，这些文件通常最有用。

## 常见问题

### 1. 登录偶尔失败，一次成功一次失败

这是正常现象之一。

GitHub Actions 使用的是共享云 IP，Epic 对风控比较敏感。常见现象包括：

- 登录页验证码一次过，一次不过
- `captcha_invalid`
- 登录响应超时
- 同一个账号隔一会儿又能成功

建议做法：

- 先不要连续狂点十几次重跑
- 失败后稍等几分钟再跑一次
- 确认账号已经关闭 2FA

### 2. 页面弹出 `One more step`

这不是异常，是 Epic 结账阶段追加的人机校验。

现在项目已经能处理这类二次安全验证。  
你看到下面这种弹窗，不代表脚本坏了：

![Checkout Security Check](docs/images/checkout-security-check.png)

常见题型包括：

- 拖拽题
- 点选题
- 多选图片题

### 3. 页面提示 `Device not supported`

这个提示通常出现在：

- 商品只支持 Windows
- GitHub Actions 运行环境是 Linux

这不代表没法领。  
现在代码会自动尝试点击 `Continue` 继续流程。

### 4. 为什么工作流显示成功，但游戏没入库

这类问题过去确实出现过，常见根因包括：

- 商品页状态识别不准
- `Place Order` 已点击，但结账页还在二次验证
- 页面上出现了额外弹窗
- 以前某些文案误判成“已拥有”

现在仓库已经把这类问题尽量改成：

- 不确认成功，就不报成功
- 自动保存截图和文本
- 把 checkout 中间状态保留下来方便排查

### 5. 为什么日志里会看到 `btoa is read-only`

这是 `hcaptcha-challenger` 在某些页面注入 HSW 脚本时的兼容性噪声。  
它不一定会导致本次运行失败。

如果最后仍然成功登录或成功领取，这条通常可以先忽略。

### 6. 模型额度用完会是什么表现

如果真的是模型额度、鉴权或者限流问题，通常更像下面这些：

- HTTP `429`
- 智谱业务错误码
- 明确的 auth / quota / rate-limit 提示

如果日志已经显示模型正常返回了坐标或题型，一般就不是“额度耗尽”。

## 推荐的使用方式

如果你只是想稳定领取周免，推荐下面这套：

1. 使用 GitHub Actions
2. 使用 GLM 视觉模型
3. 账号关闭 2FA
4. Secrets 只保留最少必需项
5. 首次手动跑通，再交给定时任务

这套配置最省心，也最容易排错。

## Docker 用法

如果你不想用 GitHub Actions，也可以在自己的服务器、NAS 或本地 Docker 环境里跑。

主要入口是：

- [`docker/docker-compose.yaml`](docker/docker-compose.yaml)

GLM 示例：

```yaml
environment:
  - LLM_PROVIDER=glm
  - GLM_API_KEY=your_glm_key
  - GLM_BASE_URL=https://open.bigmodel.cn/api/paas/v4
  - GLM_MODEL=glm-4.6v-flash
```

Gemini / AiHubMix 示例：

```yaml
environment:
  - LLM_PROVIDER=gemini
  - GEMINI_API_KEY=your_key
  - GEMINI_BASE_URL=https://aihubmix.com
  - GEMINI_MODEL=gemini-2.5-pro
```

## 开发者附录

普通用户看到这里基本就够了。  
如果你想继续改项目、提 PR、或者自己二次开发，再看这一部分。

### 项目结构

- [`app/deploy.py`](app/deploy.py)
  运行入口，负责浏览器启动、登录、领取和调度。
- [`app/services/epic_authorization_service.py`](app/services/epic_authorization_service.py)
  负责登录、登录结果监听和登录后验证。
- [`app/services/epic_games_service.py`](app/services/epic_games_service.py)
  负责周免发现、商品页进入、加购、结账、checkout 验证处理。
- [`app/settings.py`](app/settings.py)
  负责环境变量、模型路由和默认值。
- [`app/extensions/llm_adapter.py`](app/extensions/llm_adapter.py)
  负责 Gemini/AiHubMix/GLM 兼容适配。
- [`.github/workflows/epic-gamer.yml`](.github/workflows/epic-gamer.yml)
  GitHub Actions 工作流入口。

### 这次适配里真正踩过的坑

下面这些不是猜测，都是这次实际遇到并修过的问题。

#### 1. GLM 不是简单改个 Base URL 就能用

`hcaptcha-challenger` 内部调用的是 `google-genai` 风格的多模态接口。  
所以接 GLM 时，不能只把 `GEMINI_BASE_URL` 改成智谱地址。

真正要做的是：

- 保留上层调用方式
- 在适配层里把图片、消息和结构化输出转换成 GLM 能接受的格式

#### 2. 不同验证码阶段，题型真的会变

登录阶段和 checkout 阶段的题型不一定一样。  
这次实际碰到过：

- `image_drag_single`
- `image_label_multi_select`

如果适配层只对拖拽题做兼容，结账时就会死在第二道验证上。

#### 3. GLM 的输出格式并不稳定

这次遇到过的返回形式包括：

- `Source Position: (...)`
- `{"source": [...], "target": [...]}`
- `{"answer":"..."}`
- 只返回题型名，比如 `image_label_multi_select`
- 坏掉的半结构化 JSON

所以 [`llm_adapter.py`](app/extensions/llm_adapter.py) 现在做了很多“解包和转 schema”的兼容。

#### 4. Epic checkout 不只会弹 hCaptcha

这次已经确认过，结账过程中可能出现：

- `Device not supported`
- `One more step`
- 额外的 checkout iframe
- 页面仍停留在 `Place Order`，但实际上还没确认成功

因此 [`epic_games_service.py`](app/services/epic_games_service.py) 现在做了这些事：

- 检查设备不支持弹窗并尝试继续
- 识别 checkout 安全校验
- 在 `Place Order` 后循环观察真实结果
- 没确认成功就不误报成功

#### 5. “已拥有”判断不能扫整页文本

曾经误把页面里的版权文本 `owned by ...` 当成了“已拥有”。  
后来修正成：

- 优先看按钮和 checkout 状态
- 只识别高精度成功文案

#### 6. artifact 非常重要

真正把问题定位清楚，靠的不只是控制台日志，还包括：

- `app/volumes/runtime/purchase_debug/*.png`
- `app/volumes/runtime/purchase_debug/*.txt`
- `app/volumes/logs/`

如果没有这些 artifact，很多 checkout 问题只能靠猜。

### 本地开发命令

```bash
uv sync
uv run black . -C -l 100
uv run ruff check --fix
```

### 注意事项

- 这个仓库当前不建议补跑测试。
- 改动验证码链路时，优先保留日志和截图。
- 改动 checkout 流程时，优先保证“未确认成功就不要报成功”。

## 免责声明

- 本项目仅用于学习和研究自动化流程。
- 自动化操作可能违反相关平台的服务条款，请自行评估风险。
- 使用本项目产生的后果由使用者自行承担。
