# Bitcoin-Private-Key-Collision
比特币私钥碰撞器研究

这是一个初始页面，版本更新会在这个页面进行通知

软件使用的时候运行exe文件后载入下载好的adress.txt文件即可

联系邮箱：  hightbiger@gmail.com 

联系微信：   hightbiger

联系推特：  https://twitter.com/hightbiger

youtube：  https://www.youtube.com/channel/UCS6f79bKHQoYjD0sz4nOMog

linktree： https://linktr.ee/hightbiger

比特币私钥碰撞器原理说明
一、核心原理

本工具基于比特币地址生成的数学原理，通过暴力搜索的方式在比特币地址空间中寻找与目标地址匹配的私钥。其核心逻辑遵循以下公式推导：

私钥 (private key) → 公钥 (public key) → 比特币地址 (address)

当且仅当：

address = Hash160(SHA256(public_key)) 

与目标地址匹配时，即完成有效碰撞。
二、关键技术组成

    椭圆曲线加密 (ECC)

    使用secp256k1曲线：y² = x³ + 7
    私钥生成：32字节随机数 k ∈ [1, n-1]，其中 n = FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE BAAEDCE6 AF48A03B BFD25E8C D0364141
    公钥计算：Q = k * G，其中G为生成点

    哈希运算

    SHA-256：对公钥进行首次哈希
    RIPEMD-160：生成160位哈希摘要
    双重SHA-256：用于校验和计算

    地址编码

    Legacy地址（Base58编码）：
    地址 = Base58(0x00 || Hash160 || Checksum)
    校验和计算：Checksum = SHA256(SHA256(version + Hash160)).substr(0,4)

三、碰撞流程

sequenceDiagram
    participant 用户端
    participant Worker线程
    participant 加密模块

    用户端->>Worker线程: 启动碰撞任务
    loop 持续生成
        Worker线程->>加密模块: 生成随机私钥
        加密模块->>加密模块: 计算公钥(Q = k*G)
        加密模块->>加密模块: 计算Hash160(SHA256(Q))
        加密模块->>加密模块: 生成Legacy地址
        Worker线程->>Worker线程: 比对目标地址列表
        alt 地址匹配
            Worker线程->>用户端: 返回匹配结果
        else 不匹配
            Worker线程->>Worker线程: 累计计数
        end
    end

四、性能优化

    Web Worker多线程

    独立线程执行计算任务
    避免阻塞主线程

    动态批处理控制

    每批处理10个私钥
    计算公式控制速度：
    延迟时间 = max( (1000ms * 批处理量) / 目标速度 - 实际耗时, 0 )
    示例：当实际处理耗时15ms时：
    理论耗时 = (10/50)*1000 = 200ms 实际延迟 = 200 - 15 = 185ms

    内存优化

    使用TypedArray处理二进制数据
    地址列表预加载为Set结构（O(1)查询复杂度）

五、安全机制

    本地化处理

    所有计算在浏览器端完成
    无网络传输

    私钥生命周期

    仅存在于内存中
    成功匹配后立即清除内存数据

    资源限制

    最大地址数限制：1,000,000
    自动内存回收机制

六、技术限制

    理论碰撞概率

    比特币私钥空间：2²⁵⁶ ≈ 1.15e77
    每秒50次的碰撞效率：
    年碰撞概率 = 50*60*60*24*365 / 2²⁵⁶ ≈ 4.4e-69

    实际应用场景

    仅适用于教学演示
    可用于验证已知地址的私钥
    不适合实际挖矿或破解

七、风险提示

本工具演示的暴力破解方式在实际中不可行，比特币网络的安全性基于ECC算法在现有计算能力下的不可逆性。开发者及使用者应严格遵守当地法律法规，仅将本工具用于密码学学习和技术研究。
