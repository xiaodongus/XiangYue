# 想曰

**想曰(yuē)** 是基于现代加密技术的文本加密工具，使用多算法级联加密方案，确保数据在本地完成加密/解密，保护隐私安全。

## 🌟 特点

- ㊙️**密文**：支持 `中文 / Base64 / Emoji` 密文
- 🔐**密钥**：`PBKDF2-SHA256 + HKDF-SHA256`，有效抵御暴力破解
- 🔒**级联算法**：采用 `AES-CTR` 与 `ChaCha20-Poly1305-IETF` 级联加密，**安全性极高**
- 📄**数据**：所有操作在本地完成，数据不离开设备

## 📋密文示例

以下密文使用 **默认密码** 加密

```plaintext
雷柜箱慕虎斜灯无羞站愁层梁条人岂解显无靠走峰抬旋吵所扶诉旗晃接勤哈袜方错美过晕盆拧奔随梦疏清跃蝶拍说海鸟房清烟月急压非片抖呱棒千说呱海晨也梯读盘压太甜旗狐向画颗得池厨又沉叉托茶暖峦虚小料叹门跨桂已闹李竖咯棒愿咯迷首馆
```

```plaintext
J7ni11NnCUEe1+GtZcIWoJcKNgzsyN8K8BQBKnDn/1mLPkv2ul1VUcedyoIgZpXcNUKfy3HhZI6soaa54UcqLtJs52caSPuVo3EBOYvMqYS2
```
```plaintext
🧕🛕🐱🌉🛐🤴🌄🏸🚆🎇🤴🦈🛸🧭🚡💒🤑🚤🔁🚬💰🍣⛴️🎽🔣😚❣️♻️🍖🧺🚨⛪️🛁📞🍤👦🍊🦘🦀🚅💓🏏🚪☪️😠💲🦊🧭🐠🎻🪣🚢⏲️⏯️😒🗻🧂🚠👻💗🪲🦽🐍🚲⏭️⏸️😍🛖🫐🛫🥓👴🐪👰⏰🏬🍱🤎🧄ℹ️⚾️🉑🚐🕎🐪😜🦖🚭🦐👽🧎🍢🥦🧘🐄🥖🔢🏃🎸🍤♎️🌆🐆🌋🤍☮️🫓🐑
```

## 在线与离线使用

[**想曰**](https://xyue.515188.xyz/)	
[**想说**](https://xshuo.515188.xyz/)（发给朋友，免尴尬）	
[**离线客户端**](https://github.com/fzxx/XiangYue/releases)（安卓暂时还需要在线使用）

## 更新日志

[更新日志](https://github.com/fzxx/XiangYue/blob/main/CHANGELOG.md)

## 🛡️ 技术细节

#### 加密流程

```plaintext
明文 → Deflate压缩 → AES-CTR加密 → ChaCha20-Poly1305加密 → Base64编码 → 密文
                                                            ↳ 映射中文/Emoji → 密文
```

#### 密钥派生流程

```plaintext
密码 + 随机盐(16字节) 
    ↳ PBKDF2-SHA256(50万次迭代) → HKDF-SHA256
                                   ↳ 派生AES-CTR密钥(256位)
                                   ↳ 派生ChaCha20密钥(256位)
```

#### 数据结构

```plaintext
[中文、Emoji密文/Base64密文]
    ↳ 映射→解码/解码
             ↳[二进制数据]
                   ↳ 前16字节 → 盐值(Salt)
                   ↳ 接下来12字节 → ChaCha20的Nonce
                   ↳ 剩余部分 = ChaCha20密文
                                  ↳ 解密 → 前16字节 → AES-CTR的Nonce
                                  ↳ 剩余部分 = AES-CTR密文
                                                 ↳ 解密 → Deflate解压 → [明文]
```

#### 安全要素

| 要素           | 长度    | 方式          | 用途                     |
| -------------- | ------- | ------------- | ------------------------ |
| 加密算法       | -       | 级联          | 增强数据安全性           |
| 密钥派生       | -       | PBKDF2 + HKDF | 防止暴力破解、彩虹表攻击 |
| 盐 (Salt)      | 16 字节 | 随机生成      | 确保每次加密生成唯一密钥 |
| CTR_Nonce      | 16 字节 | 随机生成      | 初始计数器值             |
| ChaCha20_Nonce | 12 字节 | 随机生成      | 一次性随机数             |

## 😕 疑问

#### 少量文字也会生成较长的文本，能缩短吗？

- 因为**追求安全性**，所以添加了随机盐、Nonce等参数，密文中存储这些参数导致的；去掉参数**追求短密文会削弱安全性**，所以无短密文计划。

#### 未来还会添加的算法？

- 暂定密钥派生使用**Argon2id**，加密算法使用三种进行级联。

#### 为什么不使用PBKDF2-SHA512？

- 如果使用`PBKDF2-SHA512`迭代50万次，加密短文本也会感到明显的延迟，而`PBKDF2-SHA256`能平衡速度与安全。

#### 感觉加密/解密慢？

- PBKDF2迭代50万次所以慢，是正常现象；短文本理应1秒内加/解密完成，5M的文本3秒左右，这是可以接受的速度。

#### 支持加密/解密的最大容量？

- 不影响速度的情况下，**建议5M以下**（测试支持50M+）；超大的文本请使用压缩包或者其它方式加密。

#### 可以给密文再次加密？

- `明文 → emoji密文 → 中文密文 → ......`  可以这样无限套娃，但不会增加安全性，安全性取决于你的密码，因此不建议这样做。

## 📖 许可证

[想曰](https://github.com/fzxx/XiangYue) - [私下研究专用许可](https://github.com/fzxx/XiangYue?tab=License-1-ov-file#)

[libsodium.js](https://github.com/jedisct1/libsodium.js/) - ISC 许可证
