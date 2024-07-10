# DID去中心化分布式数字身份学习3——隐私保护和属性选择性披露

在实际生活中，我们并不总是希望直接将整个证件VC亮给验证者看，比如我们去住酒店时，需要登记姓名、身份证号信息，但是如果我们直接把身份证给前台人员的话，前台人员就可以看到我们的民族、住址等信息，对于我们普通人来说，也许觉得没什么，那要是明星、公众人物去住酒店，那么可能前台人员就可能出于各方面的原因偷偷把住址信息记下了或者泄露到网上，给证照本人的生活带来各种麻烦。那么我们有什么办法呢？用户属性的选择性披露能够降低风险。

我们以小明从公安机关获得身份证VC，然后在住酒店时，只出示姓名、照片和身份证号，不对外暴露民族和住址（因为身份证编号里面已经有生日了，所以我们就忽略掉出生日期属性）为例，说明**用户属性的选择性披露**的处理过程。

## Merkle树

Merkle Tree（默克尔树），又称哈希树，是一种树形数据结构，它用于验证**数据的完整性和一致性**。Merkle Tree最初由Ralph Merkle在1980年代提出，广泛应用于区块链、分布式系统、数据存储等领域。其关键特点是它能够高效且安全地验证大量数据。

使用默克尔树的目的是为了能够将一个区块中的所有交易形成一个短小的指纹（默克尔根，哈希值），并将这个指纹放到区块头，任何对交易的篡改都会导致指纹变化。而我们使用默克尔树而不是直接将区块中所有的交易直接算哈希的原因是因为我们希望能够进行快速的简单支付验证（SPV）。

### Merkle树结构

>1. **叶子节点（Leaf Nodes）**：
>   - 最底层的节点，每个叶子节点通常存储数据块的哈希值。在区块链中，这些数据块可以是交易记录。
>2. **非叶子节点（Non-Leaf Nodes）**：
>   - 上层节点，它们的值是其子节点的哈希值。具体来说，每个**非叶子节点**存储其子节点值的**串联后的哈希**。
>3. **根节点（Root Node）**：
>   - 树的顶层节点，即Merkle根（Merkle Root）。它是通过一系列的哈希运算从叶子节点逐层往上计算所得。Merkle Root唯一且确定地代表了整个树的数据。

### Merkle树的构造

假设有一组数据块，构造Merkle Tree的步骤如下：

1. 按照**固定的顺序**排列数据
2. 对**每个数据块**进行哈希计算，得到**哈希值**
3. 逐层将**相邻的哈希值**合并成新的哈希值，直到最终得到**根哈希值**

![image-20240708203415179](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240708203415179.png)

### Merkle树验证

假设我们要验证区块中存在Hash值为**9Dog:64（绿色框）**的交易，我们仅需要知道**1FXq:18、ec20、8f74（黄色框）**即可计算出781a、5c71与Root节点（藕粉色框）的哈希，如果最终计算得到的Root节点哈希与区块头中记录的哈希**（6c0a）**一致，即代表该交易在区块中存在。这是因为我上文提到的两个点，一个是默克尔树是**从下往上逐层计算**的，所以只要知道**相邻的另一个节点的hash值**就可以**一直往上计算直到根节点**，另一个是**根节点的hash值**可以准确的**作为一组交易的唯一摘要**，依据这两点就可以来验证一笔交易是否存在。

![image-20240708204836600](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240708204836600.png)

## 生成基于用户属性的Merkle树

基于前面提到的默克尔树和默克尔验证，我们可以将用户的属性作为Data部分计算默克尔树，比如我们要对身份证上的属性构建默克尔树：

![image-20240709101507675](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240709101507675.png)

基于上面的默克尔树，我们可以只暴露生日，而不暴露其他字段，然后给出验证路径，从而证明生日数据的真实性。但是这里有个潜在的隐私泄露问题，就是我们会暴露Hash1，而攻击者是可以穷举所有民族数据，算出每个民族的Hash然后进行比对，从而得出我并不想暴露的”民族“属性。那么怎么办呢？最简单的办法就是我们为**每个字段都加点”盐“**。

用户在生成默克尔树之前，需要先生成一个**随机的种子**，并将这个种子数据保存下来，然后基于这个种子生成**N个序列**（N取决与我们默克尔树的叶子节点数），因为我们的种子是随机生成的，所以我们可以认为这个序列也是随机的。最简单的办法就是用哈希函数，不断的对上一个数据进行Hash，从而得到下一个数据。以下是我的序列生成函数：

```go
package main

import (
    "crypto/sha256"
    "fmt"
)

// GenerateSequence256 生成一个包含 count 个哈希值的序列
// 使用 seed 作为初始输入，并通过调用 getHash 函数来生成每个后续的哈希值
func GenerateSequence256(seed []byte, count int) [][]byte {
    // 初始化结果切片
    result := [][]byte{}
    // current 变量保存当前的哈希值，初始化为种子值
    current := seed
    // 循环 count 次
    for i := 0; i < count; i++ {
        // 计算当前哈希值的 SHA-256，得到下一个哈希值
        current = getHash(current)
        // 将当前的哈希值添加到结果切片
        result = append(result, current)
    }
    // 返回包含所有哈希值的结果切片
    return result
}

// getHash 计算输入字节切片的 SHA-256 哈希值
func getHash(input []byte) []byte {
    // 创建一个新的 SHA-256 哈希对象
    h := sha256.New()
    // 将输入数据写入哈希对象
    h.Write(input)
    // 计算哈希并返回结果，作为字节切片
    return h.Sum(nil)
}

func main() {
    // 初始化种子
    seed := []byte("initial seed")
    // 定义需要生成的哈希值数量
    count := 10
    // 调用 GenerateSequence256 函数生成哈希序列
    sequence := GenerateSequence256(seed, count)
    // 遍历哈希序列并打印每个哈希值
    for _, hash := range sequence {
        // 格式化打印哈希值为十六进制字符串
        fmt.Printf("%x\n", hash)
    }
}
```

有了这个无限长的随机序列，那么我们就可以将每一个默克尔树叶子节点**加盐**了，如下图所示：

![image-20240709102750953](C:\Users\11325\AppData\Roaming\Typora\typora-user-images\image-20240709102750953.png)

现在我们Hash值都是加了盐后计算的结果，所以不可能再被碰撞出原始数据。

## 生成VC

我们在准备VC数据时，除了给出证件中的每个属性外，还需要给出：随机种子seed，默克尔根，发证机关对默克尔根的签名。我们以身份证为例，那么公安机关颁发给我们的身份证VC如下所示：

```json
{
  // VC内容所遵循的JSON-LD标准
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://studyzyexamples.com/identity/v1"
  ],
  // 本VC的唯一标识，也就是证书ID
  "id": "vc511112200001010015",
  // VC内容的格式
  "type": ["VerifiableCredential", "Identity"],
  // 本VC的发行人
  "issuer": "did:公安部门ID",
  // 本VC的发行时间
  "issuanceDate": "2010-07-01T19:73:24Z",
  // VC声明的具体内容
  "credentialSubject": {
    // 被声明的人的DID
    "id": "did:cid:511112200001010015",
    // 声明内容:姓名、性别、生日、民族、住址等
    "name":"小明",
    "gender":"男",
    "birthdate":"2000-01-01",
    "nation":"汉",
    "address":"A省B市C区D街道xxx号",
    //接下来是种子数、默克尔根、公安的签名
    "seed":"23523865082340324",
    "merkleRoot":"ea59a369466be42d1a4783f09ae0721a5a157d6dba9c4b053d407b5a4b9af145",
    "rootSignature":"3066022051757c2de7032a0c887c3fcef02ca3812fede7ca748254771b9513d8e266",
    "signer":"did:公安部门ID#keys-1"
  },
  // 对本VC的证明
  "proof": {
    "creator": "did:公安部门ID#keys-1",
    "type": "Secp256k1",
    "signatureValue": "3044022051757c2de7032a0c887c3fcef02ca3812fede7ca748254771b9513d8e2bb"
  }
}
```

这里我们需要注意的是对默克尔根的签名并不是在proof字段里面，而是在credentialSubject里面，proof里面的签名是对整个VC数据的签名，而rootSignature是公安机关对构造的默克尔树的根哈希进行的签名。

## 生成VP

接下来假如小明要去参加一个生日当天免费送礼品的活动，活动方要验证小明的出生日期，于是小明可根据上一步骤的VC，生成对应的VP，其中只暴露生日字段，其他身份属性不暴露，示例如下：

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://studyzyexamples.com/identity/v1"
  ],
  "type": "VerifiablePresentation",
  // 本VP包含的VC的内容
  "verifiableCredential": [{
    "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://studyzyexamples.com/identity/v1"
  ],
  "id": "vc511112200001010015",
  "type": ["VerifiableCredential", "Identity"],
  "issuer": "did:公安部门ID",
  "issuanceDate": "2010-07-01T19:73:24Z",
  "credentialSubject": {
    "id": "did:cid:511112200001010015",
    //以下是要选择性披露的内容
    "birthdate":"2000-01-01",
    //以下是验证披露字段有效性的数据
    //数据在默克尔树中的索引
    "dataIndex":2,
    //本数据加盐的值
    "salt":"6b264354ed367ced527a86d38f75f9c3888bd3939f548cc48d93af435890b84a",
    //默克尔验证路径
    "merklesibling":"34b64151443c3124620bf4ff69a05e97d580f0878b374b8343c6a5c3d8223435 9d2b5b35ccb5bf18747c1f5dc05771c68ce613e6eb0c5f5ef77cec8ba3e9da67 bb82c63d4e21525125bf66a6724fbb4dcbded26aae2baa2633235dc12730016e",
    //默克尔根哈希
    "merkleRoot":"ea59a369466be42d1a4783f09ae0721a5a157d6dba9c4b053d407b5a4b9af145",
    //公安机关对默克尔根的签名
    "rootSignature":"3066022051757c2de7032a0c887c3fcef02ca3812fede7ca748254771b9513d8e266",
    //用的公安机关哪个Key进行的签名
    "signer":"did:公安部门ID#keys-1"
  },
  
  }],
  // Holder小明对本VP的签名信息
  "proof": {
    "type": "Secp256k1",
    "created": "2010-07-02T21:19:10Z",
    "proofPurpose": "authentication",
    "verificationMethod": "did:cid:511112200001010015#keys-1",
    // challenge和domain是为了防止重放攻击而设计的
    "challenge": "1f44d55f-f161-4938-a659-f8026467f126",
    "domain": "4jt78h47fh47",
    "jws": "eyJhbGciOiJSUzI1NiIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..kTCYt5XsITJX1CxPCT8yAV-TVIw5WEuts01mq-pQy7UJiN5mgREEMGlv50aqzpqh4Qq_PbChOMqsLfRoPsnsgxD-WUcX16dUOqV0G_zS245-kronKb78cPktb3rk-BuQy72IFLN25DYuNzVBAh4vGHSrQyHUGlcTwLtjPAnKb78"
  }
}
```

这里我们给出了选择披露的字段“birthdate”，然后给出了要进行默克尔验证的几个必不可少的字段：

>- "birthdate":"2000-01-01”，要披露的原始数据内容。
>- dataIndex，披露字段在构架默克尔树时在叶子节的索引，因为我们这里按：姓名、性别、生日、民族、住址排序，所以生日的索引值是2.
>- salt，对birthdate这个字段加的盐，这个数据小明可以根据自己VC中的**seed和dataIndex**计算得到。
>- merkleSibling是进行默克尔**验证所需的路径**，以前面提到的4个Data为例，如果我们要算**Data2的验证路径**，那么就是**Hash1，Hash34**。
>- merkleRoot默克尔根，这个就不再累述。
>- rootSignature和signer是用来验证默克尔根的合法性的。merkleRoot、rootSignature、signer验证通过，则说明这个默克尔根是公安部认证过的。

## 验证VP

商家在收到用户提交的VP后，需要进行逐步的验证，主要包括以下步骤：

>1.根据小明的DID从区块链中获取小明的DID文档，从中获得公钥，验证VP签名真实有效。
>
>2.根据VC中的issuer，从**区块链或分类账中**获得公安机关的DID文档，从文档中获得公钥，另外也验证该DID是一个可信的DID。
>
>3.根据公安部门的公钥，验证**默克尔根的签名**是否正确。
>
>4.对披露字段、Dataindex、Salt、MerkleSibling、MerkleRoot等进行**默克尔验证**，保证披露字段是公安机关认证过的。
>
>5.以上所有步骤验证通过，显示可信的披露内容："birthdate":"2000-01-01”。

商家验证完成了小明的出生日期，但是并没有获得除了出生日期之外的其他身份信息，从而实现了选择性披露。

## 小结

在用户身份中具有多个属性时，用户只选择性的暴露其中某个属性，而且基于默克尔证明，给出了可信的证明字段，任何用户在收到VP后都可以进行合法性验证。同时我们基于随机种子生成了一个随机序列，并将随机序列作为盐添加到每个字段中，从而防止了潜在的默克尔验证时暴露的哈希值导致其他身份属性被碰撞的事情发生。
