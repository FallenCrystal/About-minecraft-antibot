# AntiAttackRL

---

## 未完待续!:

有很多可以专门补充的地方了, 但我不会将其中发生的事情都说进来.

有关更多信息, 可以阅读 [这里](https://www.mcbbs.net/thread-730758-1-1.html)

实际上, AntiAttackRL仍然包含有使用AntiFakePlayer的代码.

---

## 无用且混乱的设计:

这里暂时只说对于BungeeCord平台的代码. 其它平台的实践逻辑也差不多.

### BCUtils:

```java
// Decompiled with: Procyon 0.6.0
// Class Version: 8
package com.relatev.minecraft.antiattack.bungeecord;

import java.lang.reflect.Method;
import java.util.logging.Level;
import java.util.logging.Logger;
import net.md_5.bungee.api.chat.TextComponent;
import java.util.Iterator;
import net.md_5.bungee.BungeeCord;
import net.md_5.bungee.api.connection.ProxiedPlayer;
import net.md_5.bungee.api.connection.Connection;

public class BCUtils
{
    public static ProxiedPlayer getPlayer(final Connection con) {
        for (final ProxiedPlayer player : BungeeCord.getInstance().getPlayers()) {
            if (player.getPendingConnection().getAddress().equals(con.getAddress())) { return player; }
        }
        return null;
    }
    
    public static void setCancelReason(final Object obj, final String reason) {
        for (final Method method : obj.getClass().getMethods()) {
            if (method.getName().equals("setCancelReason")) {
                if (method.getAnnotatedParameterTypes()[0].getType().getTypeName().equals("net.md_5.bungee.api.chat.BaseComponent")) {
                    try { method.invoke(obj, new TextComponent(reason)); }
                    catch (final Exception ex) { Logger.getLogger(BCUtils.class.getName()).log(Level.SEVERE, null, ex); }
                }
                if (method.getAnnotatedParameterTypes()[0].getType().getTypeName().equals("java.lang.String")) {
                    try { method.invoke(obj, reason); }
                    catch (final Exception ex) { Logger.getLogger(BCUtils.class.getName()).log(Level.SEVERE, null, ex); }
                }
            }
        }
    }
    
    public static void disconnect(final Connection con, final String reason) {
        for (final Method method : con.getClass().getMethods()) {
            if (method.getName().equals("disconnect")) {
                if (method.getAnnotatedParameterTypes()[0].getType().getTypeName().equals("net.md_5.bungee.api.chat.BaseComponent")) {
                    try { method.invoke(con, new TextComponent(reason)); }
                    catch (final Exception ex) { Logger.getLogger(BCUtils.class.getName()).log(Level.SEVERE, null, ex); }
                }
                if (method.getAnnotatedParameterTypes()[0].getType().getTypeName().equals("java.lang.String")) {
                    try { method.invoke(con, reason); }
                    catch (final Exception ex) { Logger.getLogger(BCUtils.class.getName()).log(Level.SEVERE, null, ex); }
                }
            }
        }
    }
}
```

无用的反射, 不严谨的判断. 让我们来分析一下:

  - getPlayer
    - 为什么要对Players进行for循环? 这样操作会浪费相当多的时间. 如果玩家已经连接到服务器, 
      为什么不选择将其转换成ProxiedPlayer对象呢? 那玩家已经连接到服务器了, 
      为什么还要再将ProxiedPlayer转换成Connection再尝试得到ProxiedPlayer对象呢?
  - setCancelReason
    - 写一样的方法也不会导致代码不工整, 而是选择使用反射来做同样的工作来降低性能
    - 创建BaseComponent的对象不需要反射.
    - 没有存储BaseComponent类型 而是选择在使用此方法时创建一次性的BaseComponent对象. 
      这类文本组件的操作实际上是相当耗时的.
  - disconnect
    - Connection对象本身就具有断开连接的方法 (就像你在反射中看到的一样). 但他为什么依然坚持使用反射?
      这么做有什么意义吗?
    - 此方法从来没有在代码中使用过.

## 对于不同的检查选择使用自己的监听器.

这好到了哪里去吗? 我不知道  
反射浪费的资源就不叫计算机资源了吗

## 玩家已在线检查使用for循环

```java
// Decompiled with: Procyon 0.6.0
// Class Version: 8
package com.relatev.minecraft.antiattack.bungeecord.module;

import net.md_5.bungee.event.EventHandler;
import java.util.Iterator;
import com.relatev.minecraft.antiattack.bungeecord.BCUtils;
import net.md_5.bungee.api.connection.ProxiedPlayer;
import net.md_5.bungee.api.event.PreLoginEvent;
import net.md_5.bungee.api.plugin.Plugin;
import com.relatev.minecraft.antiattack.bungeecord.AntiAttackBC;
import net.md_5.bungee.BungeeCord;
import com.relatev.minecraft.antiattack.bungeecord.BCConfigManager;
import net.md_5.bungee.api.plugin.Listener;

public class BCAntiKickAttack implements Listener
{
    private static String AntiKickAttackDenyMessage;

    public BCAntiKickAttack() {
        BCAntiKickAttack.AntiKickAttackDenyMessage = BCConfigManager.config.getString("AntiAttack.PluginPrefix") + BCConfigManager.config.getString("AntiKickAttack.DenyMessage");
        if (BCConfigManager.config.getBoolean("AntiKickAttack.enable")) {
            BungeeCord.getInstance().getPluginManager().registerListener(AntiAttackBC.mainPlugin, this);
        }
    }

    @EventHandler
    public void onPreLogin(final PreLoginEvent event) {
        if (event.isCancelled()) {
            return;
        }
        for (final ProxiedPlayer player : BungeeCord.getInstance().getPlayers()) {
            if (player.getName().equals(event.getConnection().getName())) {
                event.setCancelled(true);
                BCUtils.setCancelReason(event, BCAntiKickAttack.AntiKickAttackDenyMessage);
            }
        }
    }
}
```

两次啊两次..

  - 可以使用.getPlayer(玩家名) 可以更节省性能得到ProxiedPlayer对象. 然后使用 `!= null` 判断
  - 大小写不相同的玩家可以绕过此检查.
  - 每次加入都会for循环一遍所有在线玩家 对于大型服务器负担非常重.
  - BCUtils的无用方法buff加成

## 无用的实现抽象类

创建一堆无用的空类和抽象类
我已经不好说什么了

---

## 总结

另一个反向对服务器造成伤害的自动白名单插件 :P

当然 AntiFakePlayer的虚假宣传也没好到哪里去就是了..

由ChatGPT生成:
```markdown
这篇文章作者对一个名为"AntiAttackRL"的项目进行了批评和总结，从作者的角度看，这是一个针对机器人和攻击的反机器人插件。总结作者的观点可以如下：

作者对"AntiAttackRL"提出了以下主要批评点：

1. 无用且混乱的设计： 作者指出项目中存在一些代码设计上的问题，特别是在与BungeeCord平台交互的部分。作者提到了一些具体问题，比如不必要的反射使用、性能浪费以及未使用的方法。

2. 对于不同的检查选择使用自己的监听器： 作者批评了项目中使用多个监听器进行相似的检查，认为这样的设计浪费了计算机资源，特别是反射操作。

3. 玩家已在线检查使用for循环： 作者指出在检查玩家是否已在线时，使用了不必要的for循环来遍历所有在线玩家，这可能在大型服务器上造成性能问题。作者还提到了大小写不敏感的问题和BCUtils的无用方法。

4. 无用的实现抽象类： 作者提到项目中存在许多无用的空类和抽象类，这似乎没有实际用途。

总的来说，作者对"AntiAttackRL"项目的评价是负面的，认为其代码存在许多不合理和浪费性能的设计。作者可能希望项目能够进行改进，以提高效率和代码质量。同时，作者还提到了与"AntiFakePlayer"插件相关的虚假宣传问题，暗示该插件的宣传可能也存在问题。
```

---