#### 文档标题：OpenClaw 配置与 WhatsApp 登录操作记录
#### 环境信息
- 操作系统：Linux (WSL)
- 操作用户：root
- OpenClaw 版本：2026.2.24 (df9a474)
- 代理服务器：172.26.48.1:14388 (HTTP/SOCKS5)

#### 一、代理环境配置
##### 1.1 问题排查：初始代理配置失败
```bash
# 初始配置（变量未定义导致失败）
export http_proxy="http://$WIN_PHYSICAL_IP:14388"
export https_proxy="http://$WIN_PHYSICAL_IP:14388"
export all_proxy="socks5://$WIN_PHYSICAL_IP:14388"
export no_proxy="localhost,127.0.0.1,::1,$WIN_PHYSICAL_IP,172.16.0.0/12,192.168.0.0/16,10.0.0.0/8,10.255.255.254"

# 测试失败（变量未赋值）
curl ipinfo.io  # 错误：Unsupported proxy syntax in 'http://:14388'
```

##### 1.2 正确的代理配置步骤
```bash
# 1. 清空旧的代理配置
unset http_proxy https_proxy all_proxy ALL_PROXY no_proxy NO_PROXY

# 2. 定义物理机IP变量
WIN_PHYSICAL_IP=172.26.48.1

# 3. 配置代理环境变量
export http_proxy="http://$WIN_PHYSICAL_IP:14388"
export https_proxy="http://$WIN_PHYSICAL_IP:14388"
export all_proxy="socks5://$WIN_PHYSICAL_IP:14388"
export no_proxy="localhost,127.0.0.1,::1,$WIN_PHYSICAL_IP,172.16.0.0/12,192.168.0.0/16,10.0.0.0/8,10.255.255.254"

# 4. 验证代理配置
curl ipinfo.io  # 成功返回IP信息（新加坡IP：205.198.86.36）
curl -I https://github.com  # 成功返回200状态码
ping $WIN_PHYSICAL_IP -c 4  # 验证物理机连通性（0%丢包）
```

#### 二、OpenClaw 服务启动
##### 2.1 启动 OpenClaw Gateway 服务
```bash
# 启动网关服务
openclaw gateway start

# 检查服务状态
systemctl --user status openclaw-gateway.service
```

##### 2.2 服务状态验证结果
```
● openclaw-gateway.service - OpenClaw Gateway (v2026.2.24)
     Loaded: loaded (/root/.config/systemd/user/openclaw-gateway.service)
     Active: active (running)  # 服务正常运行
     Main PID: 9780 (openclaw-gatewa)
     Memory: 403.3M
     CPU: 27.237s
```

#### 三、OpenClaw 渠道管理操作
##### 3.1 错误的命令尝试（记录）
```bash
# 错误1：参数格式错误
openclaw channels login whatsapp --verbose  # 错误：too many arguments
openclaw channels whatsapp login            # 错误：unknown command 'whatsapp'

# 错误2：无效选项
openclaw channels list --all                # 错误：unknown option '--all'
openclaw gateway exec --channel whatsapp -- command link  # 错误：unknown option '--channel'
```

##### 3.2 正确的渠道列表查看
```bash
# 查看已配置渠道
openclaw channels list
```

##### 3.3 渠道列表输出结果
```
🦞 OpenClaw 2026.2.24 (df9a474)
   I don't judge, but your missing API keys are absolutely judging you.

Chat channels:
- WhatsApp default: not linked, enabled

Auth providers (OAuth + API keys):
- minimax-cn:default (api_key)

Usage: no provider usage available.

Docs: gateway/configuration
```

##### 3.4 WhatsApp 登录尝试（失败）
```bash
# 执行 WhatsApp 登录命令
openclaw channels login --channel whatsapp
```

##### 3.5 登录失败结果
```
🦞 OpenClaw 2026.2.24 (df9a474)
   I'll refactor your busywork like it owes me money.

Waiting for WhatsApp connection...
WhatsApp Web connection ended before fully opening. status=408 Request Time-out WebSocket Error ()
Channel login failed: Error: status=408 Request Time-out WebSocket Error ()
```

#### 四、网络连通性验证
```bash
# 验证 GitHub 访问
curl -I https://github.com  # 返回200状态码，访问正常

# 验证 WhatsApp 官网访问
curl -I -s -m 5 https://www.whatsapp.com | head -1  # 返回200 Connection established
```

#### 五、问题总结
1. **已解决问题**：代理环境配置完成，网络访问正常（GitHub/WhatsApp官网均可访问），OpenClaw Gateway 服务启动成功。
2. **未解决问题**：WhatsApp 渠道登录失败，错误原因：WebSocket 连接超时（status=408）。
3. **待排查方向**：
   - WhatsApp Web 的 WebSocket 连接是否被代理拦截
   - OpenClaw 的 WhatsApp 适配配置是否完整
   - 代理服务器是否支持 WebSocket 协议转发

### 总结
1. **核心操作流程**：先清空旧代理配置 → 定义IP变量 → 配置新代理 → 启动OpenClaw服务 → 尝试WhatsApp登录，该流程是排查和配置的核心逻辑。
2. **关键问题点**：初始代理失败是因为`WIN_PHYSICAL_IP`变量未提前定义，WhatsApp登录失败是WebSocket连接超时（408错误），需重点排查代理对WebSocket的支持。
3. **验证方法**：通过`curl ipinfo.io`验证代理有效性，通过`systemctl --user status`验证服务状态，通过`curl -I`验证目标网站连通性。
