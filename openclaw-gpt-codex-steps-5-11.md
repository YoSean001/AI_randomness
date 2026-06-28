# OpenClaw 接入 GPT/Codex 订阅：步骤 5-11

这份文档可以直接丢给 Codex Desktop，让它通过 Computer Use 在腾讯云 OrcaTerm 里照着执行。

目标：让 OpenClaw 使用 GPT/Codex 订阅授权，而不是 OpenAI API Key。

重要规则：

```text
1. 你可以操作浏览器、腾讯云控制台、OrcaTerm 网页终端。
2. 你可以输入普通命令、检查输出、定位错误、继续下一步。
3. 遇到账号密码、验证码、OAuth 授权、扫码、付款、购买确认时，必须停下来让我自己操作。
4. 不要让我输入 OpenAI API Key。
5. GPT/Codex 订阅授权优先使用 codex login --device-auth。
6. 出现验证链接和验证码时，必须清楚提示我复制链接、输入验证码、登录 OpenAI / ChatGPT 并完成授权。
```

## 第 5 步：安装 OpenClaw Codex 插件

在 OrcaTerm 里执行下面命令。

```bash
openclaw plugins install @openclaw/codex
```

检查插件是否存在。

```bash
openclaw plugins list | grep -i codex || true
```

如果插件没有 enabled，尝试启用并重启。

```bash
openclaw plugins enable codex || true
openclaw gateway restart || true
sleep 5
openclaw plugins list | grep -i codex || true
```

## 第 6 步：安装服务器上的 OpenAI Codex CLI

执行下面命令，安装服务器端 Codex CLI，并确认 `--device-auth` 可用。

```bash
node -v || true
npm -v || true
npm install -g @openai/codex
codex --help | head -40
codex login --help | sed -n '1,120p'
```

## 第 7 步：用 device-auth 登录 GPT/Codex 订阅

执行下面命令。它会输出一个验证链接和验证码。

```bash
codex login --device-auth
```

看到链接和验证码后，Codex 必须停下来提醒用户：

```text
请复制终端里的验证链接到浏览器打开。
请复制终端里的验证码到验证页面。
请登录你的 OpenAI / ChatGPT 账号。
请完成授权。
完成后告诉我，我再继续检查。
```

不要替用户输入 OpenAI 密码、验证码、2FA、恢复码或任何敏感信息。

用户完成授权后，继续执行。

```bash
codex login status || true
```

## 第 8 步：重启 OpenClaw 并检查模型

执行下面命令。

```bash
openclaw gateway restart || true
sleep 5
openclaw models status || true
openclaw models status --plain || true
```

如果默认模型不是 GPT/Codex，则先根据 `openclaw models status` 里实际显示的模型 ID 设置默认模型。

常见示例：

```bash
openclaw config set agents.defaults.model.primary '"openai/gpt-5.5"' --strict-json
openclaw config set agents.defaults.model.fallbacks '[]' --strict-json
openclaw gateway restart
sleep 5
openclaw models status --plain
```

注意：模型 ID 必须以本机 `openclaw models status` 实际显示为准。如果 `openai/gpt-5.5` 不存在，不要硬设，先从 status 里找正确 ID。

## 第 9 步：做最小测试

执行下面命令。

```bash
time openclaw agent --agent main --session-id gpt-oauth-test-001 --message "只回复 OK"
```

如果返回 OK，说明 GPT/Codex 订阅接入成功。

再执行：

```bash
openclaw models status --plain
codex login status || true
openclaw gateway status --deep || true
```

## 第 10 步：如果一直要求 OpenAI API Key，按这个排查

先执行。

```bash
openclaw plugins list | grep -i codex || true
codex login status || true
openclaw models status || true
```

如果 Codex 插件缺失：

```bash
openclaw plugins install @openclaw/codex
openclaw gateway restart
sleep 5
```

如果 Codex CLI 未登录：

```bash
codex login --device-auth
```

然后再次提醒用户完成浏览器验证。

如果 OpenClaw 版本太旧：

```bash
openclaw --version
openclaw update
openclaw --version
```

## 第 11 步：腾讯云面板看不到 GPT 时，不要误判失败

即使腾讯云面板没有显示 GPT，也继续以 CLI 为准。

```bash
openclaw models status
openclaw models status --plain
codex login status || true
```

只要 CLI 显示 GPT/Codex 模型可用，并且 agent 测试能返回 OK，就算成功。

## 最终成功标准

最后执行：

```bash
codex login status || true
openclaw models status --plain || true
time openclaw agent --agent main --session-id final-gpt-test --message "只回复 OK"
openclaw gateway status --deep || true
```

成功标准：

```text
1. codex login status 显示 ChatGPT 登录成功。
2. openclaw models status --plain 显示 GPT/Codex 模型。
3. openclaw agent 测试能返回 OK。
4. openclaw gateway 正常运行。
```

完成后，请用简短中文告诉用户：

```text
已完成：
- OpenClaw 版本：
- Codex 插件状态：
- Codex 登录状态：
- 当前默认模型：
- Agent 测试结果：
- 是否还需要配置飞书/微信通道：
```
