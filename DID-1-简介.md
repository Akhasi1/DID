# DID去中心化分布式数字身份学习1——简介

## 数字身份

国际电子技术委员会将“身份”定义为“一组与实体关联的属性”。这里的实体不仅仅是人，对于机器或者物体都可以是实体，甚至网络中虚拟的东西也可以是实体并拥有身份。

随着互联网的出现和普及，传统的身份有了另外一种表现形式，即数字身份。一般认为，数字身份的演进经历了四个阶段分别是：中心化身份、联盟身份、以用户为中心的身份以及自我主权身份。

- 中心化身份是由单一的权威机构进行管理和控制的，现在互联网上的大多数身份还是中心化身份。
- 联盟身份的出现解决了中心化身份中身份数据零碎混乱的弊端，此种身份是有多个机构或者联盟进行管理和控制的，用户的身份数据具备了一定程度的可移植性，例如允许用户登录某个网站时，可以使用其他网站的账户信息，类似于QQ、微信或者微博的跨平台登录。
- **以用户为中心的身份**则将重点集中在去中心化上，通过授权和许可进行身份数据的共享，例如**OpenID**。
- 自我主权身份才是真正意义上的**去中心化的**、完全由个人所拥有和控制的身份。

## 现阶段的弊端

在传统的数字身份体系中，我们通常依赖**中心化的机构**来发行和验证我们的身份。这些机构掌控着身份数据的存储、管理和验证权限，存在以下几方面的问题：

1. **单点故障**：如果中心化系统遭到攻击或出现故障，用户的身份和相关服务可能会受到影响。
2. **隐私问题**：用户数据集中存储在少数几个组织中，容易出现隐私泄露问题。
3. **可控制性差**：用户对自己身份数据的控制权有限，难以自由地在不同平台间携带和管理这些信息。

DID应运而生，旨在解决这些问题，通过区块链和分布式账本技术实现身份的去中心化管理。

## 分布式账本和区块链

### 定义

1. **分布式账本技术（DLT）**
   - 概念：DLT是一种通用框架，用于记录和分享数据跨多个独立系统。其本质是一个分布式数据库，所有参与节点共享账本。
   - 特性：去中心化、节点间数据复制、分散存储。
2. **区块链（Blockchain）**
   - 概念：区块链是DLT的一种特殊类型，它将数据记录成一个一个的“区块”，并按照时间顺序串联起来，形成链条。
   - 特性：数据按块存储、每个区块包含前一个区块的哈希、采用共识算法确保数据的完整性和安全性。

### 技术特性

1. **数据结构**
   - **DLT**：数据可以按照多种方式存储，不局限于区块结构。
   - **区块链**：数据以区块形式存储，每个区块包含前一个区块的哈希值，形成链式结构。
2. **共识算法**
   - **DLT**：可以使用多种共识算法，如Raft、PBFT（实用拜占庭容错）等，这取决于具体的实现。
   - **区块链**：通常使用特定的共识算法，如PoW（工作量证明）、PoS（权益证明）等。
3. **适用范围**
   - **DLT**：广泛适用于金融、供应链、医疗等多个领域，可以有多种实现形式。
   - **区块链**：目前广泛应用于数字货币（如比特币）、智能合约（如以太坊）、去中心化应用等。

### 实际应用

1. **DLT**：适用于需要高性能、高吞吐量以及可快速达成共识的企业级应用。例如，Corda是一个常用于金融服务的DLT平台。
2. **区块链**：更适合需要透明性、永久性和抗篡改性的公共记录系统，如比特币、以太坊。这些系统强调去信任化和去中心化。

## DID(去中心化身份)

**D**ecentralized **ID**entity去中心化身份，简称DID，相对于传统的基于PKI的身份体系，基于区块链建立的DID数字身份系统具有保证数据真实可信、保护用户隐私安全、可移植性强等特征，其优势在于：

- **去中心化**：基于区块链，避免了身份数据被单一的中心化权威机构所控制。
- **身份自主可控**：基于DPKI（分布式公钥基础设施），每个用户的身份不是由可信第三方控制，而是由其所有者控制，个人能自主管理自己的身份。
- **可信的数据交换**：身份相关数据锚定在区块链上，认证的过程不需要依赖于提供身份的应用方。

### DID标识

DID标识是一个特定格式的字符串，用来代表一个实体的数字身份，这里的实体可以是人、机、物。DID标识的格式为：

```bash
did:example:123456789abcdefghijk
```

![image-20240707223614471](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240707223614471.png)

> 前缀did:是固定的，表示这个字符串是一个did标识字符串。
> 中间的example被称为DID方法，就是用来表示这个DID标识是用哪一套方案（方法）来进行定义和操作的。
> 这个DID方法我们可以自定义，并且注册到[W3C的网站](https://w3c.github.io/did-spec-registries/#did-methods)中。 
> 最后面的部分是在该DID方法下的**唯一标识字符串**。

假如我们做了一个DID系统，我们把方法就起名叫chead，想把中国公民的身份证信息都DID化，那么我的DID标识就是： did:chead:3715*************************************3这里就使用身份证号码作为chead这个DID方法下的唯一标识。

### DID文档

每一个DID标识都会对应一个DID文档（DID Document）。这个文档就是一个**JSON字符串**，里面一般会包含如下信息：

​	**标识符**

​	DID标识符本身，也就是DID文档所描述的该DID。由于DID的全局唯一特性，因此在DID文档中只能有一个DID。

​	**公钥**

​	公钥用于**数字签名及其他加密**操作，这些操作是实现身份验证以及与服务端点建立安全通信等目的的基础。如果DID文档中不存在公钥，则必须假定密钥已被撤销或无效，同时必须包含或引用密钥的撤销信息（例如，撤销列表）。

​	**身份验证**

​	身份验证的过程是DID主题通过加密方式来证明它们与DID相关联的过程。

​	**授权**

​	授权意味着他人代表DID主题执行操作，例如当密钥丢失的时候，可以授权他人更新DID文档来协助恢复密钥。

​	**服务端点**

​	除了发布身份验证和授权机制之外，DID文档的另一个主要目的是为主题发现服务端点。服务端点可以表示主题希望公告的任何类型的服务，包括用于进一步发现、身份验证、授权或交互的去中心化身份管理服务。

​	**时间戳**

​	文档创建时间和更新时间。

![image-20240709205544113](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240709205544113.png)

以上信息并不是必须有，不过一般建议包含。我们来看一个具体的DID文档示例：

```json
{
"@context": "https://w3id.org/did/v1",
"id": "did:example:123456789abcdefghi",
"authentication": [{
// 本DID文档对应的DID标识
"id": "did:example:123456789abcdefghi#keys-1",
"type": "RsaVerificationKey2018",
"controller": "did:example:123456789abcdefghi",
//本DID对应的公钥信息
"publicKeyPem": "-----BEGIN PUBLIC KEY...END PUBLIC KEY-----\r\n"
}],
"service": [{
// 获取本DID对应的VC的服务接口
"id":"did:example:123456789abcdefghi#vcs",
"type": "VerifiableCredentialService",
"serviceEndpoint": "https://example.com/vc/"
}]
}
```

DID文档中最重要的就是公钥信息，这是我们接下来要进行VC和VP验证的基础。

我们一般是把DID标识作为Key，把DID文档作为Value存储到区块链中，利用区块链不可篡改、共享数据访问的特点，实现接下来在验证身份时能快速访问获取可信数据。

## VC(可验证声明/可验证凭证)

(Verifiable Claims 或 Verifiable Credentials，简称VC)是一个 DID 给另一个 DID 的某些属性做**背书**而发出的描述性声明，并附加自己的数字签名，用以证明这些属性的真实性，可以认为是一种数字证书。 传统的PKI数字证书体系需要CA来颁发，而在DID中也是分为颁发者、持有者、验证者、DID注册系统（也就是**区块链或者其他分布式账本**），具体关系如图：

![image-20240708114723369](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240708114723369.png)

- 颁发者Issuer就是VC的颁发机构，比如身份证就是公安机关作为颁发者，毕业证书就是大学作为颁发者。
- 持有者Holder就是VC的持有人，就是我们这些普通人。
- 验证者Verifier就是在我们使用证书时查看我们VC的人或者机构。比如我们入住酒店，前台要验证我们的身份证，那么酒店前台就是验证者；再比如我们入职新公司时需要提供大学毕业证书，新公司HR就是验证者。
- DID注册系统Verifiable Data Registry就是我们存储了**DID标识**和**DID文档**的地方，通过DID标识可以查询到对应的DID文档。

一个VC也是一个JSON字符串，里面包含如下信息：

![image-20240708135715759](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240708135715759.png)

- Credential Metadata：VC元数据，主要就是发行人、发行日期、声明的类型等信息。
- Claim(s)：声明，一个或者多个关于主体的说明。比如身份证作为公安机关颁发给我的VC，在声明中会包含：姓名、性别、出生日期、民族、住址等信息。
- Proof(s)：证明，通常就是颁发者的数字签名，保证了本VC能够被验证，防止VC内容被篡改以及验证VC的颁发者。

下面是官方给出的一个VC的具体样例：

```json
{
  // VC内容所遵循的JSON-LD标准
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  // 本VC的唯一标识，也就是证书ID
  "id": "http://example.edu/credentials/1872",
  // VC内容的格式
  "type": ["VerifiableCredential", "AlumniCredential"],
  // 本VC的发行人
  "issuer": "https://example.edu/issuers/565049",
  // 本VC的发行时间
  "issuanceDate": "2010-01-01T19:73:24Z",
  // VC声明的具体内容
  "credentialSubject": {
    // 被声明的人的DID
    "id": "did:example:ebfeb1f712ebc6f1c276e12ec21",
    // 声明的断言内容
    "alumniOf": {
      "id": "did:example:c276e12ec21ebfeb1f712ebc6f1",
      "name": [{
        "value": "Example University",
        "lang": "en"
      }, {
        "value": "Exemple d'Université",
        "lang": "fr"
      }]
    }
  },
  // 对本VC的证明
  "proof": {
    // 签名算法
    "type": "RsaSignature2018",
    // 签名创建时间
    "created": "2017-06-18T21:19:10Z",
    // 本证明的目的
    "proofPurpose": "assertionMethod",
    // 验证本签名的公钥的ID
    "verificationMethod": "https://example.edu/issuers/keys/1",
    // 数字签名的内容
    "jws": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..TCYt5XsITJX1CxPCT8yAV-TVkIEq_PbChOMqsLfRoPsnsgw5WEuts01mq-pQy7UJiN5mgRxD-WUcX16dUEMGlv50aqzpqh4Qktb3rk-BuQy72IFLOqV0G_zS245-kronKb78cPN25DGlcTwLtjPAYuNzVBAh4vGHSrQyHUdBBPM"
  }
}
```

因为VC中具有用户的隐私信息，所以VC一般保存在私有的存储中，比如用户自己的手机中，或者需要授权的网络地址中。除了前面示例中给出的数据外，VC还可以有失效日期，比如我们的身份证一般10年有效，过期后就需要重新向颁发者申请新的VC。

## VP(可验证表达)

Verifiable presentation简称VP，可验证表达是VC持有者向验证者表名自己身份的数据。一般情况下，我们直接出示VC全文即可，但是在某些情况下，出于隐私保护的需要，我们并**不需要出示完整的VC内容**，只希望选择性披露某些属性，或者不披露任何属性，只需要证明某个断言即可。 

比如一个求职者要进入某写字楼面试，写字楼的保安要求登记身份证号码和姓名，但是我们的VC中还包含了民族、住址等信息，我们的求职者不希望将自己的住址暴露给保安，所以他提供给保安的VP中应该只选择性的披露的身份证号码和姓名，其他信息都不披露。 

再比如我们规定必须年满18岁才有资格购买香烟，所以一个消费者在购买香烟时必须证明自己已经年满18岁，但是直接出示身份证给收银员又会暴露太多隐私信息，就算选择性披露生日属性，也会让收银员知道了消费者具体的年龄和生日日期，这种情况下消费者只希望在VP中证明自己大于18岁，其他什么信息都不能暴露。 

VP的格式为：

![image-20240708134216227](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240708134216227.png)

- Presentation Metadata：VP元数据，主要包含了版本，本JSON对象的类型等信息
- Verifiable Credential(s)：VC列表，要对外展示的VC的内容，如果是选择性披露或者隐私保护的情形，可能就不包含任何VC。
- Proof(s)：证明，主要就是持有者对本VP的签名信息。

VP的官方样例：

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  "type": "VerifiablePresentation",
  // 本VP包含的VC的内容
  "verifiableCredential": [{
    "@context": [
      "https://www.w3.org/2018/credentials/v1",
      "https://www.w3.org/2018/credentials/examples/v1"
    ],
    "id": "http://example.edu/credentials/1872",
    "type": ["VerifiableCredential", "AlumniCredential"],
    "issuer": "https://example.edu/issuers/565049",
    "issuanceDate": "2010-01-01T19:73:24Z",
    "credentialSubject": {
      "id": "did:example:ebfeb1f712ebc6f1c276e12ec21",
      "alumniOf": {
        "id": "did:example:c276e12ec21ebfeb1f712ebc6f1",
        "name": [{
          "value": "Example University",
          "lang": "en"
        }, {
          "value": "Exemple d'Université",
          "lang": "fr"
        }]
      }
    },
    "proof": {
      "type": "RsaSignature2018",
      "created": "2017-06-18T21:19:10Z",
      "proofPurpose": "assertionMethod",
      "verificationMethod": "https://example.edu/issuers/keys/1",
      "jws": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..TCYt5XsITJX1CxPCT8yAV-TVkIEq_PbChOMqsLfRoPsnsgw5WEuts01mq-pQy7UJiN5mgRxD-WUcX16dUEMGlv50aqzpqh4Qktb3rk-BuQy72IFLOqV0G_zS245-kronKb78cPN25DGlcTwLtjPAYuNzVBAh4vGHSrQyHUdBBPM"
    }
  }],
  // Holder对本VP的签名信息
  "proof": {
    "type": "RsaSignature2018",
    "created": "2018-09-14T21:19:10Z",
    "proofPurpose": "authentication",
    "verificationMethod": "did:example:ebfeb1f712ebc6f1c276e12ec21#keys-1",
    // challenge和domain是为了防止重放攻击而设计的
    "challenge": "1f44d55f-f161-4938-a659-f8026467f126",
    "domain": "4jt78h47fh47",
    "jws": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..kTCYt5XsITJX1CxPCT8yAV-TVIw5WEuts01mq-pQy7UJiN5mgREEMGlv50aqzpqh4Qq_PbChOMqsLfRoPsnsgxD-WUcX16dUOqV0G_zS245-kronKb78cPktb3rk-BuQy72IFLN25DYuNzVBAh4vGHSrQyHUGlcTwLtjPAnKb78"
  }
}
```

## 小结

以上主要是DID的背景、定义、相关技术等。
