---
description: 快速保护节点的步骤。
---

# 指南\| ETH2验证器信标链节点的安全最佳实践

## 🤖 前提条件

* 已安装 Ubuntu 服务器或 Ubuntu 桌面
* 已安装 SSH 服务器
* 一个 SSH 客户端或终端窗口访问

如果您需要安装 SSH 服务器，请参考：

如果您的操作系统需要一个 SSH 客户端，请参考：

## 🧙创建一个非根用户，拥有sudo 权限

{% hint style="info" %}
使用非根帐户登录到您的服务器的习惯。 如果发生错误，这将防止文件被意外删除。 例如，如果root 用户错误地运行，命令 `rm` 可以擦除整个服务器。
{% endhint %}

{% hint style="danger" %}
🔥**提示**: 不要经常使用根帐户。 使用 `su` 或 `sudo`, 总是.
{% endhint %}

通过您的 SSH 客户端到您的服务器

```bash
ssh username@server.public.ip.address
# 示例
# ssh myUsername@77.22.161.10
```

创建一个叫做ethereum的新用户

```text
sudo useradd -m -s /bin/bash ethereum
```

设置ethereum用户密码

```text
sudo passwd ethereum
```

将ethereum添加到sudo组

```text
sudo usermod -aG sudo ethereum
```

## 🔐 **禁用 SSH 密码验证并仅使用 SSH 密钥**

{% hint style="info" %}
加强SSH的基本规则是：

* 没有访问SSH 的密码 \(使用私钥\)
* 不允许 SSH \(适当用户应该在 SSH 中，然后 `su` 或 `sudo`\)
* 对用户使用 `sudo` 以便命令被记录
* 记录未经授权的登录尝试 \(并考虑软件到屏蔽/封禁用户，他们试图访问您的服务器太多次，如失败2ban\)
* 锁定SSH 到只需要的 IP 范围\(如果你觉得它喜欢它\)
{% endhint %}

在您的本地机器上创建新的 SSH 密钥对。 在您的本地机器上运行它。 您将被要求输入要保存密钥的文件名。 这将是您的 **关键名**。

您选择 [ED25519 或 RSA](https://goteleport.com/blog/comparing-ssh-keys/) 公钥算法。

{% tabs %}
{% tab title="ED25519" %}
```text
ssh-keygen -t ed25519
```
{% endtab %}

{% tab title="RSA" %}
```bash
ssh-keygen -t rsa -b 4096
```
{% endtab %}
{% endtabs %}

将公钥传输到您的远程节点。 正确更新 **keyname.pub**。

```bash
ssh-copy-id -i $HOME/.ssh/keyname.pub ethereum@server.public.ip.address
```

使用您的新用户登录

```text
ssh ethereum@server.public.ip.address
```

禁用基于根登录和密码的登录。 编辑 `/etc/ssh/sshd_config 文件`

```text
sudo nano /etc/ssh/sshd_config
```

找到 **个挑战响应身份验证** 并更新为 no

```text
ChallengeResponseAuthentication no
```

找到 **个密码认证** 更新至无

```text
PasswordAuthentication no
```

找到 **PermitRoot登录** 并更新为无

```text
PermitRootLogin no
```

找到 **PermittemptyPasswords** 并更新为 no

```text
PermitEmptyPasswords no
```

**可选的**: 定位 **端口** 并自定义您的 **随机** 端口。

{% hint style="info" %}
使用 **随机** 端口 \# 从 1024 推送到49141 。 [检查可能发生的冲突。 ](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)
{% endhint %}

```bash
Port <port number>
```

验证您新的 SSH 配置的语法。

```text
sudo sshd -t
```

如果语法验证没有错误，则重新加载 SSH 进程

```text
sudo service sshd reload
```

验证登录仍然工作

{% tabs %}
{% tab title="Standard SSH Port 22" %}
```text
ssh ethereum@server.public.ip.address
```
{% endtab %}

{% tab title="Custom SSH Port" %}
```bash
ssh ethereum@server.public.ip.address -p <custom port number>
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
或者，如果使用了自定义的SSH端口，则可能需要添加-p 标志。

```bash
ssh -i <path to your SSH_key_name.pub> ethereum@server.public.ip.address
```
{% endhint %}

**可选的**: 通过更新本地ssh配置，使登录更加轻松。

为了简化登录服务器所需的ssh命令，请考虑更新本地$ HOME / .ssh / config文件：

```bash
Host ethereum-server
  User ethereum
  HostName <server.public.ip.address>
  Port <custom port number>
```

这将允许您使用ssh ethereum-server登录，而无需显式传递所有ssh参数。

## 🤖 更新您的系统

{% hint style="warning" %}
至关重要的是，使系统具有最新的补丁程序以保持最新状态，以防止入侵者访问您的系统。
{% endhint %}

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get autoremove
sudo apt-get autoclean
```

启用自动更新，因此您不必手动安装它们。

```text
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

## 🐻 禁用root帐户

系统管理员不应经常以root用户身份登录以维护服务器安全性。 相反，您可以使用需要低级特权的sudo execute。

```bash
＃要禁用root帐户，只需使用-l选项。
sudo passwd -l root
```

```bash
＃如果出于某些正当理由需要重新启用该帐户，则只需使用-u选项。
sudo passwd -u root
```

## 🛠 设置SSH的两因素身份验证\[可选\]

{% hint style="info" %}
SSH（安全外壳）通常用于访问远程Linux系统。 由于我们经常使用它来连接包含重要数据的计算机，因此建议添加另一个安全层。 这是两因素身份验证（2FA）。
{% endhint %}

```text
sudo apt install libpam-google-authenticator -y
```

要使SSH使用Google Authenticator PAM模块，请编辑/etc/pam.d/sshd文件：

```text
sudo nano /etc/pam.d/sshd
```

添加以下行：

```text
auth required pam_google_authenticator.so
```

现在，您需要使用以下命令重新启动sshd守护程序：

```text
sudo systemctl restart sshd.service
```

调整`/etc/ssh/sshd_config`

```text
sudo nano /etc/ssh/sshd_config
```

找到ChallengeResponseAuthentication并更新为yes

```text
ChallengeResponseAuthentication yes
```

找到UsePAM并更新为

```text
UsePAM yes
```

保存文件并退出。 运行google-authenticator命令。

```text
google-authenticator
```

它将询问您一系列问题，这是建议的配置：

* 让令牌“时间基础”：“是”
* 更新 `.google_author` 文件：是
* 不允许多次使用：是
* 增加最初生成时间限制：无
* 启用频率限制：是

您可能已经注意到在此过程中出现的巨型QR码，下面是您无法使用手机时要使用的紧急刮刮码：将它们写下来并保存在安全的地方。

现在，在手机上打开Google Authenticator，然后添加您的密钥以使两因素身份验证生效。

## 🧩 安全共享内存

{% hint style="info" %}
您应该做的第一件事就是保护系统上使用的共享内存。 如果您不知道，共享内存可用于攻击正在运行的服务。 因此，请保护系统内存的该部分。

要了解有关安全共享内存的更多信息，请阅读此内容 [techrepublic.com article](https://www.techrepublic.com/article/how-to-enable-secure-shared-memory-on-ubuntu-server/).
{% endhint %}

{% hint style="warning" %}
### 一个例外情况

可能由于某些原因，您需要以读/写模式安装该内存空间（例如，需要对共享内存进行此类访问的特定服务器应用程序（例如DappNode）或标准应用程序（例如Google Chrome））。 在这种情况下，请对fstab文件使用以下行，并按以下说明进行操作。

```text
none /run/shm tmpfs rw,noexec,nosuid,nodev 0 0
```

上面的行将安装具有读/写访问权限的共享内存，但无权执行程序，更改正在运行的程序的UID或在名称空间中创建块或字符设备。 这是对默认设置的净安全性改进。

### 谨慎使用

经过反复试验，您可能会发现某些应用程序（例如DappNode）在只读模式下无法与共享内存一起使用。 为了获得最高的安全性并与您的应用程序兼容，进行此安全共享内存设置是值得的。

来源: [techrepublic.com](https://www.techrepublic.com/article/how-to-enable-secure-shared-memory-on-ubuntu-server/)
{% endhint %}

编辑`/etc/fstab`

```text
sudo nano /etc/fstab
```

将以下行插入文件底部，然后保存/关闭。 这会将共享内存设置为只读模式。

```text
tmpfs    /run/shm    tmpfs    ro,noexec,nosuid    0 0
```

重新引导节点以使更改生效。

```text
sudo reboot
```

## ⛓安装 **Fail2ban**

{% hint style="info" %}
Fail2ban是一个入侵防御系统，它监视日志文件并搜索与失败的登录尝试相对应的特定模式。 如果从特定IP地址（在指定的时间内）检测到一定数量的失败登录，则fail2ban会阻止从该IP地址进行访问。
{% endhint %}

```text
sudo apt-get install fail2ban -y
```

编辑一个用于监视SSH登录的配置文件。

```text
sudo nano /etc/fail2ban/jail.local
```

将以下行添加到文件的底部。

{% hint style="info" %}
🔥 **将IP地址列入白名单提示**: `ignoreip`参数接受您可以指定允许连接的IP地址，IP范围或DNS主机。 在这里您要指定本地计算机，本地IP范围或本地域，以空格分隔。

```bash
# 例子
ignoreip = 192.168.1.0/24 127.0.0.1/8
```
{% endhint %}

```bash
[sshd]
enabled = true
port = <22 or your random port number>
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
# 列入白名单的IP地址
ignoreip = <list of whitelisted IP address, your local daily laptop/pc>
```

保存/关闭文件。 重新启动fail2ban，以使设置生效。

```text
sudo systemctl restart fail2ban
```

## 🧱配置防火墙

标准UFW防火墙可用于控制对节点的网络访问。

对于任何新安装，默认情况下都禁用ufw。 使用以下设置启用它。

* 端口 22 \(或您的随机端口 \#\) SSH 连接
* P2p 流量的端口
  * 灯塔使用端口 9000 tcp/udp
  * Teku 使用端口 9000 tcp/udp
  * Prysm使用端口 13000 tcp 和 12000 udp
  * Nimbus使用端口 9000 tcp/udp
  * Lodestar使用端口 30607 tcp 和 9000 udp
* 端口 303 tcp/udp eth1 节点

{% tabs %}
{% tab title="Lighthouse" %}
```bash
# 默认情况下，拒绝所有传入和传出流量
sudo ufw default deny incoming
sudo ufw default allow outgoing
# 允许SSH访问
sudo ufw allow ssh #<port 22 or your random ssh port number>/tcp
# 允许p2p端口
sudo ufw allow 9000/tcp
sudo ufw allow 9000/udp
# 允许 eth1 端口
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
# 启用防火墙
sudo ufw enable
```
{% endtab %}

{% tab title="Prysm" %}
```bash
# 默认情况下，拒绝所有传入和传出流量
sudo ufw default deny incoming
sudo ufw default allow outgoing
# 允许SSH访问
sudo ufw allow ssh #<port 22 or your random ssh port number>/tcp
# 允许p2p端口
sudo ufw allow 13000/tcp
sudo ufw allow 12000/udp
# 允许 eth1 端口
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
# 启用防火墙
sudo ufw enable
```
{% endtab %}

{% tab title="Teku" %}
```bash
# 默认情况下，拒绝所有传入和传出流量
sudo ufw default deny incoming
sudo ufw default allow outgoing
# 允许SSH访问
sudo ufw allow ssh #<port 22 or your random ssh port number>/tcp
# 允许p2p端口
sudo ufw allow 9000/tcp
sudo ufw allow 9000/udp
# 允许 eth1 端口
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
# 启用防火墙
sudo ufw enable
```
{% endtab %}

{% tab title="Nimbus" %}
```bash
# 默认情况下，拒绝所有传入和传出流量
sudo ufw default deny incoming
sudo ufw default allow outgoing
# 允许SSH访问
sudo ufw allow ssh #<port 22 or your random ssh port number>/tcp
# 允许p2p端口
sudo ufw allow 9000/tcp
sudo ufw allow 9000/udp
# 允许 eth1 端口
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
# 启用防火墙
sudo ufw enable
```
{% endtab %}

{% tab title="Lodestar" %}
```bash
# 默认情况下，拒绝所有传入和传出流量
sudo ufw default deny incoming
sudo ufw default allow outgoing
# 允许SSH访问
sudo ufw allow ssh #<port 22 or your random ssh port number>/tcp
# 允许p2p端口
sudo ufw allow 30607/tcp
sudo ufw allow 9000/udp
# 允许 eth1 端口
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
# 启用防火墙
sudo ufw enable
```
{% endtab %}
{% endtabs %}

```bash
# 验证状态
sudo ufw status numbered
```

{% hint style="danger" %}
不要将Grafana（端口3000）和Prometheus端点（端口9090）暴露在公共互联网上，因为这会引起新的攻击面！ 一个安全的解决方案是使用Wireguard通过ssh隧道访问Grafana。
{% endhint %}

仅在家庭路由器防火墙或其他网络防火墙后面的本地家庭放样设置上打开以下端口。

🔥 在VPS /云节点上打开这些端口很危险。

```bash
# 允许grafana Web服务器端口
sudo ufw allow 3000/tcp
# 启用Prometheus端点端口
sudo ufw allow 9090/tcp
```

确认设置有效。

> ```csharp
> # 验证状态
> sudo ufw status numbered
>      To                         Action      From
>      --                         ------      ----
> [ 1] 22/tcp                     ALLOW IN    Anywhere
> # SSH
> [ 2] 3000/tcp                   ALLOW IN    Anywhere
> # Grafana
> [ 3] 9000/tcp                   ALLOW IN    Anywhere
> # eth2 p2p traffic
> [ 4] 9090/tcp                   ALLOW IN    Anywhere
> # Prometheus
> [ 5] 30303/tcp                  ALLOW IN    Anywhere
> # eth1 node
> [ 6] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
> # SSH
> [ 7] 3000/tcp (v6)              ALLOW IN    Anywhere (v6)
> # Grafana
> [ 8] 9000/tcp (v6)              ALLOW IN    Anywhere (v6)
> # eth2 p2p traffic
> [ 9] 9090/tcp (v6)              ALLOW IN    Anywhere (v6)
> # Prometheus
> [10] 30303/tcp (v6)             ALLOW IN    Anywhere (v6)
> # eth1 node
> ```

**\[** 可选，但推荐 **\]** 可以通过以下命令设置白名单（或允许来自特定IP的连接）。

```bash
sudo ufw allow from <您当地的日常笔记本电脑/电脑>
# 例子
# sudo ufw allow from 192.168.50.22
```

{% hint style="info" %}
🎊 端口转发技巧**:** 您需要将端口转发并打开到验证器. 验证它使用 [https://www.yugetsignal.com/tools/open-ports/](https://www.yougetsignal.com/tools/open-ports/) 或 [https://canyousee.org/](https://canyouseeme.org/)。
{% endhint %}

## 📞 验证监听端口

If you want to maintain a secure server, you should validate the listening network ports every once in a while. This will provide you essential information about your network.

```bash
sudo ss -tulpn
# 示例输出。 确保端口号是正确的。
# Netid  State    Recv-Q  Send-Q    Local Address:Port   Peer Address:Port   Process
# tcp    LISTEN   0       128       127.0.0.1:5052       0.0.0.0:*           users:(("lighthouse",pid=12160,fd=22))
# tcp    LISTEN   0       128       127.0.0.1:5054       0.0.0.0:*           users:(("lighthouse",pid=12160,fd=23))
# tcp    LISTEN   0       1024      0.0.0.0:9000         0.0.0.0:*           users:(("lighthouse",pid=12160,fd=21))
# udp    UNCONN   0       0         *:30303              *:*                 users:(("geth",pid=22117,fd=158))
# tcp    LISTEN   0       4096      *:30303              *:*                 users:(("geth",pid=22117,fd=156))
```

或者，您可以使用 `netstat`

```bash
sudo netstat -tulpn
# Example output. 确保端口号是正确的。
# Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
# tcp        0      0 127.0.0.1:5052          0.0.0.0:*               LISTEN      12160/lighthouse
# tcp        0      0 127.0.0.1:5054          0.0.0.0:*               LISTEN      12160/lighthouse
# tcp        0      0 0.0.0.0:9000            0.0.0.0:*               LISTEN      12160/lighthouse
# tcp6       0      0 :::30303                :::*                    LISTEN      22117/geth
# udp6       0      0 :::30303                :::*                    LISTEN      22117/geth
```

## 👩🚀 使用系统用户帐户-最小特权原则\[高级用户/可选\]

{% hint style="info" %}
**仅建议高级用户使用**

**最小特权原则**: 每个eth2进程都分配有一个系统用户帐户，并以正常运行所需的最少特权数运行。 此最佳做法可防止在特定进程中发现的漏洞或利用可能导致访问其他系统进程的情况。
{% endhint %}

```bash
# 为eth1服务创建系统用户帐户
sudo adduser --system --no-create-home eth1

# 为验证器服务创建系统用户帐户
sudo adduser --system --no-create-home validator

# 创建信标链服务的系统用户帐户
sudo adduser --system --no-create-home beacon-chain

# 为slasher创建系统用户帐户
sudo adduser --system --no-create-home slasher
```

{% hint style="danger" %}
🔥 **给高级用户的警告**

如果决定使用系统用户帐户，请记住用相应的用户替换systemd单位文件。

```bash
# beacon-chain.service单位文件的示例
User            = beacon-chain
```

此外，在适用的情况下，确保将正确的文件所有权分配给您的系统用户帐户。

```bash
# prysm验证程序的密码文件示例
sudo chown validator:validator -R $HOME/.eth2validators/validators-password.txt
```
{% endhint %}

## ✨ 其他验证程序节点最佳实践

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left"></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x5EFA;&#x7ACB;&#x7F51;&#x7EDC;</td>
      <td style="text-align:left">&#x5C06;&#x9759;&#x6001;&#x5185;&#x90E8;IP&#x5206;&#x914D;&#x7ED9;&#x60A8;&#x7684;&#x9A8C;&#x8BC1;&#x5668;&#x8282;&#x70B9;&#x548C;&#x6BCF;&#x65E5;&#x7B14;&#x8BB0;&#x672C;/PC&#x3002;
        &#x8FD9;&#x4E0E;ufw &#x548C; Fail2ban&apos;s &#x767D;&#x540D;&#x5355; &#x529F;&#x80FD;&#x662F;&#x6709;&#x7528;&#x7684;&#x3002;
        &#x901A;&#x5E38;&#xFF0C;&#x8FD9;&#x53EF;&#x4EE5;&#x5728;&#x60A8;&#x7684;&#x8DEF;&#x7531;&#x5668;&apos;s&#x8BBE;&#x7F6E;&#x4E2D;&#x8FDB;&#x884C;&#x914D;&#x7F6E;&#x3002;
        &#x8BF7;&#x53C2;&#x8003;&#x60A8;&#x7684;&#x8DEF;&#x7531;&#x5668;&apos;s
        &#x624B;&#x518C;&#x4E86;&#x89E3;&#x8BF4;&#x660E;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7535;&#x6E90;&#x6545;&#x969C;</td>
      <td style="text-align:left">&#x5728;&#x505C;&#x7535;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F60;&#x60F3;&#x8981;&#x4F60;&#x7684;&#x9A8C;&#x8BC1;&#x673A;&#x5668;&#x5728;
        &#x53EF;&#x7528;&#x65F6;&#x91CD;&#x65B0;&#x542F;&#x52A8;&#x3002; &#x5728;
        BIOS &#x8BBE;&#x7F6E;&#x4E2D;&#xFF0C;&#x66F4;&#x6539;&#x7535;&#x6E90;&#x4E22;&#x5931;&#x65F6;&#x7684; <b>&#x8FD8;&#x539F;</b> &#x6216; <b>&#x7535;&#x6E90;&#x4E22;&#x5931;&#x540E;</b> &#x8BBE;&#x7F6E;
        &#x4EE5;&#x4FBF;&#x7EE7;&#x7EED;&#x5F00;&#x542F;&#x3002; &#x66F4;&#x597D;&#x7684;&#x662F;&#xFF0C;&#x5B89;&#x88C5;&#x4E00;&#x4E2A;&#x4E0D;&#x53EF;&#x4E2D;&#x65AD;&#x7684;&#x7535;&#x6E90;(UPS)&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x6E05;&#x9664;&#x57FA;&#x7840;&#x5386;&#x53F2;&#x8BB0;&#x5F55;</td>
      <td
      style="text-align:left">
        <p>&#x5F53;&#x6309;&#x4E0B;&#x4E0A;&#x7BAD;&#x5934;&#x952E;&#x65F6;&#xFF0C;&#x60A8;&#x53EF;&#x4EE5;&#x770B;&#x5230;&#x5148;&#x524D;&#x7684;&#x547D;&#x4EE4;&#x53EF;&#x80FD;&#x5305;&#x542B;
          &#x4E2A;&#x654F;&#x611F;&#x6570;&#x636E;&#x3002; &#x82E5;&#x8981;&#x6E05;&#x9664;&#x6B64;&#x9879;&#xFF0C;&#x8BF7;&#x8FD0;&#x884C;&#x4EE5;&#x4E0B;&#x64CD;&#x4F5C;&#xFF1A;</p>
        <p><code>shred -u ~/.bash_history &amp;&amp; tount ~/.bash_history</code>
        </p>
        </td>
    </tr>
  </tbody>
</table>

{% hint style="info" %}
请务必查看 [清单 \| 如何确认一个健康的 ETH2 验证器。](guide-or-how-to-setup-a-validator-on-eth2-mainnet/checklist-or-how-to-confirm-a-healthy-functional-eth2-validator.md)
{% endhint %}

## 🤖 通过构建验证器开始下注

### 为我们的 主要指南 [访问此处](guide-or-how-to-setup-a-validator-on-eth2-mainnet/)，为我们的 测试网指南 访问此处。

{% hint style="success" %}
恭喜您完成了指南。 ✨

你觉得我们的指南有用吗？ 向我们发送一个带有提示的信息，我们将不断更新它。

它真正激励我们继续创建最好的加密指南。

使用 [cointr.ee 找到我们的捐赠 ](https://cointr.ee/coincashew)个地址。 🙏

任何反馈和所有拉取请求都非常感激。 🌛

在 Discord 上挂起并与他人聊天 @

[https://discord.gg/w8Bx8W2HPW](https://discord.gg/w8Bx8W2HPW) :grinning\_face\_with\_big\_eyes：
{% endhint %}

🎊 **2020-12 更新**: 感谢所有 [Gitcoin](https://gitcoin.co/grants/1653/eth2-staking-guides-by-coincashew) 贡献者 您可以通过 [二次供资](https://vitalik.ca/general/2019/12/07/quadratic.html) 做出贡献并产生重大影响。 融资已完成！ 谢谢

{% embed url="https://gitcoin.co/grants/1653/eth2-staking-guides-by-coincashew" caption="" %}

## 🚀 参考

{% embed url="https://medium.com/@BaneBiddix/how-to-harden-your-ubuntu-18-04-server-ffc4b6658fe7" caption="" %}

{% embed url="https://linux-audit.com/ubuntu-server-hardening-guide-quick-and-secure/" caption="" %}

{% embed url="https://www.digitalocean.com/community/tutorials/how-to-harden-openssh-on-ubuntu-18-04" caption="" %}

{% embed url="https://ubuntu.com/tutorials/configure-ssh-2fa\#1-overview" caption="" %}

{% embed url="https://linuxize.com/post/install-configure-fail2ban-on-ubuntu-20-04/" caption="" %}

[https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3\#file-ubuntu-hardening-md](https://gist.github.com/lokhman/cc716d2e2d373dd696b2d9264c0287a3#file-ubuntu-hardening-md)

{% embed url="https://www.lifewire.com/harden-ubuntu-server-security-4178243" caption="" %}

{% embed url="https://www.ubuntupit.com/best-linux-hardening-security-tips-a-comprehensive-checklist/" caption="" %}

