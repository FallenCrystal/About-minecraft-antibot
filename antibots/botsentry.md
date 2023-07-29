# BotSentry

---

## ~~前言:~~

> ~~谁问你了?~~

你说得对, 但是《机器人烧饼》是由 OldEraStudios自主研发的一款"全新"の反机器人.
游戏发生在一个被称作「Minecraft」的沙盒游戏的圈子内.
在这里, 被神选中的人将被授予「MCSTORM」, 引导Alpha之力.
你将扮演一位名为「服主」的神秘角色. 
在自由的圈钱之旅中邂逅性格各异, 能力独特的其它反机器人们,
和他们一起"击败"强敌, 找回失散的MCSTORM
——同时,逐步挖掘机器人烧饼的真相.

### 正题:

BotSentry作为一个老牌~~圈钱~~的~~反~~机器人.  
他们dev的故事倒是挺精彩的. 在逻辑游戏上玩了三年了 始终还是被各种乱穿.  

以及.. BotSentry在AntiBotDeluxe倒下之前是免费资源, 由于ABD正好需要云api. 
倒下之后他们只能(以当时的情况来看)使用BotSentry 但dev趁火打劫 
将BotSentry改成了付费资源.

对此我的评价是: 脑子全是钱的人大多数都是干啥啥不行的

---

## 所谓的数据包检查:

控制台Filter:
```botsentry log filter
public class p {
    private static Filter a = null;
    private static boolean b = true;

    public static void a() {
        if (com.lahuca.botsentry.b.a.a.ap()) {
            b = true;
            a = ProxyServer.getInstance().getLogger().getFilter();
            ProxyServer.getInstance().getLogger().setFilter(logRecord -> {
                String string = logRecord.getMessage();
                if (string == null || string.isEmpty()) {
                    return true;
                }
                if (h.a(j.a, string)) {
                    return false;
                }
                if (logRecord.getParameters() != null) {
                    ArrayList<String> arrayList = new ArrayList<String>();
                    for (Object object : logRecord.getParameters()) {
                        if (!(object instanceof String)) continue;
                        arrayList.add((String)object);
                    }
                    if (arrayList.size() > 0 && h.a(j.a, arrayList.toArray(new String[0]))) {
                        return false;
                    }
                }
                if (logRecord.getThrown() != null && (logRecord.getThrown() instanceof IndexOutOfBoundsException || logRecord.getThrown() instanceof BadPacketException || logRecord.getThrown() instanceof IllegalArgumentException || logRecord.getThrown() instanceof IllegalStateException || logRecord.getThrown() instanceof QuietException)) {
                    k.h.incrementAndGet();
                    ++d.a;
                    return false;
                }
                return true;
            });
        }
    }

    public static void b() {
        if (a != null) {
            ProxyServer.getInstance().getLogger().setFilter(a);
        }
        a = null;
    }

    public static void c() {
        if (b) {
            p.b();
        }
        a = ProxyServer.getInstance().getLogger().getFilter();
        ProxyServer.getInstance().getLogger().setFilter(logRecord -> {
            StringJoiner stringJoiner = new StringJoiner("  ");
            stringJoiner.add(logRecord.getMessage());
            if (logRecord.getParameters() != null) {
                for (Object object : logRecord.getParameters()) {
                    if (!(object instanceof String)) continue;
                    stringJoiner.add((String)object);
                }
            }
            if (stringJoiner.toString().trim().isEmpty()) {
                return true;
            }
            if (h.a(j.a, stringJoiner.toString())) {
                return false;
            }
            if (logRecord.getThrown() != null && (logRecord.getThrown() instanceof IndexOutOfBoundsException || logRecord.getThrown() instanceof BadPacketException || logRecord.getThrown() instanceof IllegalArgumentException || logRecord.getThrown() instanceof IllegalStateException || logRecord.getThrown() instanceof QuietException)) {
                k.h.incrementAndGet();
                ++d.a;
                return false;
            }
            return true;
        });
    }

    public static void d() {
        ProxyServer.getInstance().getLogger().setFilter(a);
        a = null;
        p.a();
    }
}
```

他们声称他们甚至不需要将此类攻击列入黑名单的原因实际上是异常中实际上不会提供ip地址  
所以它们所作的只能是忽略异常 而且它们所谓的黑名单也只在加入检查上 无法阻止连接

> 实际上通过handle ClientConnectionEvent 或者 ConnectionInitEvent(Waterfall)就可以阻止地址传入连接

## 使用ConcurrentHashMap做黑名单

```botsentry blacklist
public class k {
    private static final ConcurrentHashMap<String, Boolean> i = new ConcurrentHashMap();
    private static final ConcurrentHashMap.KeySetView<String, Boolean> j = ConcurrentHashMap.newKeySet();
    private static final ConcurrentHashMap.KeySetView<String, Boolean> k = ConcurrentHashMap.newKeySet();
    private static final ConcurrentHashMap.KeySetView<String, Boolean> l = ConcurrentHashMap.newKeySet();
    private static final ConcurrentHashMap.KeySetView<String, Boolean> m = ConcurrentHashMap.newKeySet();
    private static final ConcurrentHashMap<String, g> n = new ConcurrentHashMap();
    private static final ConcurrentHashMap<String, com.lahuca.botsentry.b.a> o = new ConcurrentHashMap();
    public static HashMap<String, a> a = new HashMap();
    public static ConcurrentHashMap.KeySetView<String, Boolean> b = ConcurrentHashMap.newKeySet();
    public static ConcurrentHashMap<UUID, Long> c = new ConcurrentHashMap();
    public static final ConcurrentHashMap.KeySetView<n, Boolean> d = ConcurrentHashMap.newKeySet();
    public static final ConcurrentHashMap<String, c> e = new ConcurrentHashMap();
    private static final ConcurrentHashMap<String, Integer> p = new ConcurrentHashMap();
    public static volatile AtomicInteger f = new AtomicInteger(0);
    public static volatile AtomicInteger g = new AtomicInteger(0);
    public static volatile AtomicInteger h = new AtomicInteger(0);
    private static final ConcurrentHashMap<String, Long> q = new ConcurrentHashMap();
    private static final ConcurrentHashMap<String, Long> r = new ConcurrentHashMap();
    
    // ...
}
```

我不知道dev用ConcurrentHashMap是为了什么 但很明显 HashMap在快速读写时会占用非常高的cpu使用率 或许是想这么写是因为考虑将将黑名单写入到文件内 但真的有写入文件的必要吗?  
攻击者的代理一天换一轮 而且列入黑名单时同样将用户名写入黑名单 双倍伤害

## 其余/未归类的:

### 报告性能问题 而将问题指向代理本身无法承受过多的连接

> 前言:  
> 此评论已被开发者删除 即使我无法提供截图 但确有此事  
> ——如果您是一个BotSentry真爱粉并想否认此时发生过:  
> 请在自己的乌托邦内自娱自乐 至少我认为应该公开此事

Customer: 我的服务器在安装BotSentry之后仍然崩溃 在崩溃前的2秒 它显示收到无效数据包29134..

Dev: 亲爱的xxx, 很抱歉听到此事. 这是由于代理本身无法承受过多连接导致的
您可以考虑到切换Velocity 或使用TCPShield以保证只有合法流量经过服务器

> *"与其他“AntiBot”插件不同，我们创建了一个可行的高性能插件，该插件能够每秒处理大量连接，过滤它们，在几秒钟内将它们列入黑名单，同时允许您的玩家加入服务器。"*  
> 假设我们认为为它在1秒钟涌入了29134个连接 这样的规模应该仍然算中规中矩 当然 对于控制台过滤器来讲 无法防止是意料之内的. 在事实面前 我不在乎TCPShield或者Velocity
> 依靠外部就很大程度上说明BotSentry**不行**, 且**其它大多数反机器人**可以**轻松做到这点**

### 始终过度依赖云服务:

> 前言: 个人就是不喜欢依赖云服务 如果你杠就是你对

大多数东西都通过云服务完成 本体只有FirstJoin检查和一些基础的黑白名单.

Ultimate AntiBot的Lockdown实践原理与其类似 如果你想了解更多 你可以去看看UAB的src.

以及云服务将默认对于所有代理出现次数较高的国家默认加入黑名单 中国就是其一  
直到你验证验证码前 你的IP所在的区将默认被标记为使用代理

如果验证之后的IP周围有代理并刚好有机器人通过此代理加入 他们将被认为通过检查.

GeoIP同样使用云服务检查 明明可以使用MaxMind Database进行检查

### 半吊子IPTables & IPSet

我说啊.. 你用那ConcurrentHashMap读写都够费cpu了 现在还要执行命令 你都保护了什么啊

