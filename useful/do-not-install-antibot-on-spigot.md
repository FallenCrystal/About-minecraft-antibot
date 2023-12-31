# 不要在Spigot等服务器上安装反机器人

---

## 免责声明:

此文不代表您绝对不能在Spigot上安装反机器人.  

如果您的环境允许你运行另一个Minecraft代理 (例如Velocity, BungeeCord)  
**则请不要将后端服务器暴露公网, 并在代理上安装反机器人.**

如果因为特殊原因您的服务器只能运行一个Spigot服务器实例, 
也可以考虑一下下面列出的反机器人插件.  

但请记住, 在代理上安装反机器人是最好的选择. 它们可以接受比Spigot服务器更多的连接,
同时节省更多计算机资源. 您也可以有更多更好的选择.

---

## 推荐的可以在Spigot上运行的反机器人

### 注意!

介于Spigot服务器无法接受大量连接 因此您的服务器仍然可能会遭受因攻击而使主线程负载重重, 甚至崩溃.  
在这种情况下, 请不要指望任何反机器人能够保护您的服务器. 请自行搭建一个BungeeCord/Velocity代理.

### 推荐的插件列表
  
  - [nAntiBot](https://modrinth.com/plugin/nantibot) (可选付费 | 活跃 | 闭源)
    - 个人体验还行的一个支持Spigot的反机器人插件.
    - 该插件一些功能依赖云服务/网站验证码. 如果您服务器的网络不是很好或正在使用免费主机, 则不推荐使用该插件.
    - 由于使用反射修改了NMS, 可能会造成一些ViaVersion / ProtocolLib等与协议相关的插件的功能. 如果非常追求稳定性, 不建议使用.
  - [EpicGuard](https://github.com/4drian3d/EpicGuard) (免费 | 活跃 | 开源)
    - 一个专门为Paper/Velocity服务器制作的反机器人. 由于本身较为良好的代码实践, 算是使用事件的反机器人中较为出色的那一个.
    - 插件本身工作不需要依赖云服务. 但是使用地理IP位置功能需要下载一次数据库. 如果服务器网络不是很好, 可以选择这个.
    - 由于完全依赖于事件方法, 本体是完全支持HAProxy的. 如果使用内网穿透开服的服主可以选择这个.
    - 可能会被一些机器人绕过, 但抵御大量机器人是一个还算不错的选择.
  - [Ultimate AntiBot](https://www.spigotmc.org/resources/ultimate-antibot-%E2%9A%A1-firewall-anti-vpn-%E2%9A%A1-bungeecord-spigot-600-servers.93439/) (免费 | 不活跃 | 开源)
    - 一个还算比较可以的选择, 但由于代码和逻辑本身较为复杂 不是很推荐使用.
    - 可能会遇到一些问题, 且由于开发者不是很活跃, 如果遇到问题可能需要较长时间修复.
    - 需要依赖云服务进行代理检查. 在未完成代理检查时会使用内置白名单.
    - 由于检查本身太过于复杂, 玩家需要复杂的操作才能进入服务器.
    - 由于仅使用事件并且需要云服务, 对于非小型服务器我可能会推荐EpicGuard或nAntiBot而不是这个.
  - ~~[XProtect](https://www.spigotmc.org/resources/xprotect-the-best-protection-plugin-stop-bots-attacks-vpns-mysql-1-8-1-19.67863/) (付费 | 不活跃 | 开源)~~ (待补充)
  - ~~[BotSentry](https://www.spigotmc.org/resources/%E2%9A%A1-botsentry-%E2%9A%A1-antibot-antiproxy-resisting-30k-bots-per-second-bungee-spigot-sponge-velocity.55924/) (付费 | 活跃 | 不开源)~~ ([为什么实际上不推荐?](https://github.com/FallenCrystal/Minecraft-Antibots/blob/main/antibots/botsentry.md))

---

## 我有白名单我自豪!

实际上, 白名单不是坚不可摧的.  
即使您有白名单, 也强烈安装一个反机器人. 因为大多数反机器人处理的占用比白名单还小.

特别是当您有一个BungeeCord / Velocity代理的时候.

---

## 为什么不推荐在Spigot上安装反机器人?

 - Minecraft处理登录和世界内容在同一线程上
   - 这意味着即使在加入到世界前切断连接仍然会影响到服务器的流畅度
   - 尽管在较高版本的服务器中, 加入世界前进行的操作被列入了另一个线程处理. 但这么做仍然会造成一定的卡顿. 
     特别是考虑到它们本身就不是为高效处理连接而设计的
   - 一些Minecraft代理(例如Velocity)为每个连接使用单独的线程, 并且只处理连接和数据包, 
     这意味着要比Spigot等服务器高效的多. 也可以在不生成UUID的情况下切断与机器人的连接.  
     这可以帮助后端服务器摆脱因机器人而造成的不必要的卡顿
 - 事件处理
   - 处理事件将导致当前线程暂停. 即使在加入世界前触发事件并取消, 如果机器人较多 仍然会导致服务器卡顿甚至崩溃.
   - 一些其它插件 (如LuckPerms) 需要处理此类事件, 由于触发事件本身就需要用到反射, 这会额外使用计算机资源.
 - UUID已经生成
   - 在触发事件时, 服务端已经为玩家生成了UUID并写入到了世界文件中.
   - 虽然这不会造成太大的性能影响 但我们肯定不会希望为机器人分配UUID.
 - 难以实现复杂操作
   - 如果我们想要实施数据包检查或者通过纯数据包来完成反机器人检查, 
     就需要通过反射NMS或者其它依赖协议的插件来完成这些操作
 - 服务器上可能会安装ViaVersion
    - 由于ViaVersion是一个一个版本转换的  
      (例如1.19连接到1.20服务器, 就要先翻译成1.19.1, 再翻译成1.20)  
      这么做无异于又增加一份我们不想要的负担.

---