 Win7使用teredo连接IPv6的方法

在 ” 开始 ”->” 运行 ” 中输入 cmd 打开 windows 命令行。 在命令行中输入
netsh int teredo show state
出现 Teredo 参数 ：
若“ 状态 ”为 dormant / qualified ，则表示已连接服务器并获得 IPv6 地址。
若“ 状态 ”为 offline ，同时提示错误“无法访问主服务器地址”或其他错误，则表示未连接上服务器。

在命令行状态下输入：
netsh interface teredo set state server=teredo.remlab.net

https://github.com/XX-net/XX-Net/issues/12333#issuecomment-470865251
