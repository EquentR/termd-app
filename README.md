# termd-app

A persistent terminal emulation system consisting of a daemon, a relay, and a mobile app.

> 状态：项目初始化阶段（M0），尚无可运行代码。路线见 [docs/ROADMAP.md](docs/ROADMAP.md)。

## 核心思路

- **Daemon 跑在家里的机器上**，持有所有 PTY，并在 **daemon 端维护虚拟终端（VT 状态机）并完成渲染**。
- **客户端只发送输入**（键盘、鼠标）与 **resize**，接收 daemon 渲染后的结果（屏幕状态/增量）。因此"状态重放"问题由 daemon 一处解决，客户端无需重放原始字节流，也不需要 libssh 一类现成协议栈。
- **Relay 部署在公网服务器**，不可信（zero-trust）：客户端凭 daemon 生成的密钥，经 relay 连到 daemon，并做**端到端加密**；relay 只转发密文。
- **协议自研**，保持可扩展，后续可对接其他方案。

## 架构

```
┌────────────┐      E2E 加密（relay 不可信）      ┌─────────┐   daemon 主动外连   ┌──────────────┐
│ Mobile App │ ◄────────────────────────────► │  Relay  │ ◄─────────────────► │    Daemon    │
│ 输入/resize │      渲染结果（屏幕状态/增量）       │ (转发密文) │                     │ PTY + 虚拟终端 │
└────────────┘                                 └─────────┘                     └──────────────┘
```

| 组件 | 目录（规划） | 职责 |
|------|-------------|------|
| **Daemon** | `daemon/` | 管理多个 PTY；维护虚拟终端状态并渲染；生成并持有密钥；出站连接 Relay。 |
| **Relay** | `relay/` | 公网转发器；按连接标识路由密文帧；不持有明文与密钥。 |
| **Mobile App** | `app/` | 基础终端字符渲染、输入（键盘/鼠标）、resize、会话切换、重连。 |
| **Protocol** | `proto/` | 自研、可扩展的协议定义（帧、消息、版本协商）与测试向量。 |

## 设计原则

1. **渲染权在 daemon**：客户端不解析 VT 序列，不保存权威终端状态。
2. **零信任 relay**：端到端加密，relay 被攻破也不泄露终端内容。
3. **出站连接**：Daemon 不监听公网端口。
4. **断线可恢复**：重连后直接获取当前屏幕状态，无需回放原始流。
5. **协议先行、可扩展**：版本化，新增消息类型不破坏旧端。

## 开发

见 [AGENTS.md](AGENTS.md)。

## License

MIT，见 [LICENSE.md](LICENSE.md)。
