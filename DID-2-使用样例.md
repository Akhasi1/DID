# DID去中心化分布式数字身份学习2——DID使用样例

## 场景描述

小明是一个刚刚从大学毕业的应届毕业生，在毕业当天学校颁发了毕业证给小明对应的数字身份，小明拿到毕业证后第二天去公司入职，其中一个环节，公司HR要求验证小明的学历信息，验证完成，小明入职成功。

那么人员关系就如下表所示：

|  学校  |  小明  |   公司   |
| :----: | :----: | :------: |
| Issuer | Holder | Verifier |

## Holder小明生成DID标识和DID文档

小明要想获得学校颁发的毕业证，那么他必须要有自己的DID，所以他先下载一个数字身份的手机APP，然后创建账号。创建账号的过程就是在手机中生成一个随机是私钥和对应的公钥。这里我们假设我们的DID标识的规则是“did:cid:身份证号”，所以小明在APP中输入自己的身份证号码，生成了一个DID标识：

```json
did:cid:511112200001010015
```

那么DID文档就是：

```json
{
"@context": "https://w3id.org/did/v1",
"id": "did:cid:511112200001010015",
"version": 1,
"created": “2020-12-08T16:02:20Z",
"updated": “2020-12-08T16:02:20Z",
"publicKey": [
{
"id": "did:cid:511112200001010015#keys-1", "type": "Secp256k1", "publicKeyHex": "02b97c30de767f084ce3080168ee293053ba33b235d7116a3263d29f1450936b71" },
{ 
"id": "did:cid:511112200001010015#keys-2", "type": "Secp256k1", "publicKeyHex": "e3080168ee293053ba33b235d7116a3263d29f1450936b71" } ],
"authentication": ["did:cid:511112200001010015#key-1"],
"recovery": ["did:cid:511112200001010015#key-2"],
"service": [
{
"id": "did:cid:511112200001010015#resolver", "type": "DIDResolve", "serviceEndpoint": "https://did.studyzydemo.com" } ],
"proof": {
"type": "Secp256k1", "creator": "did:cid:511112200001010015#keys-1", "signatureValue": "QNB13Y7Q9...1tzjn4w==" } }
```

这里我们为该DID设置了两个公钥，一个是小明自己的，用于认证签名等，Key2是系统托管的，用于手机丢失或者系统崩溃导致用户私钥丢失的情况下，帮忙小明找回自己的DID，绑定成一个新的公钥。

Proof部分是小明自己用自己APP里面的私钥对该DID文档的签名。如果我们想进一步增强安全性，可以将proof部分改为由公安机关进行签名。

DID文档生成完成后，APP会将该DID和文档上链到区块链存证，一旦上链完成，所有人都能够查询到小明的这个DID和文档。这里的区块链我们一般都是一个**联盟链**，并不是任意数据都可以随意写入的，所以小明必须是使用自己的身份证号经过验证确实是本人后才会上链，防止其他人冒用小明身份证号的情况。

## Issuer学校颁发毕业证VC给小明

学校本身也有自己的DID，由于学校是教育系统里面颁发的DID，所以规则和小明作为中国公民的DID规则不一样，比如学校的DID标识是：

```json
did:cedu:xdu
```

这个DID不是学校自己签名的，而是被教育部签名的(教育部的DID是did:corg:moe)，我们在区块链中可以找到该DID对应的DID文档如下：

```json
{
"@context": "https://w3id.org/did/v1",
"id": “did:cedu:uestc",
"version": 1,
"created": “2020-12-08T16:02:20Z",
"updated": “2020-12-08T16:02:20Z",
"publicKey": [
{
"id": “did:cedu:uestc#keys-1", "type": "Secp256k1", "publicKeyHex": "3053ba33b235d7116a3e3080168ee293053ba33b235d7116a33053ba33b235d7116a3" },
{ "id": “did:cedu:uestc#keys-2", "type": "Secp256k1", "publicKeyHex": "e293053ba3053ba33b235d7116a3263d29fe293053ba" } ],
"authentication": [“did:cedu:uestc#key-1"],
"recovery": [“did:cedu:uestc#key-2"],
"service": [
{
"id": “did:cedu:uestc#resolver", "type": "DIDResolve", "serviceEndpoint": "https://did.studyzydemo.com" } ],
"proof": {
"type": "Secp256k1", "creator": "did:corg:moe#keys-1", "signatureValue": "QNB13Y7Q9...1tzjn4w==" } }
```

所有认证过的高校的DID都是由did:corg:moe这个DID创建的，所以这相当于传统的根CA，我们只需要信任这个DID创建的DID，就可以认为是正规的高校。
继续说回颁发毕业证VC，学校根据小明的学习情况（入学时间、毕业时间、专业、是否结业等信息）以及小明提交的自己的DID，生成VC如下：

```json
{
  // VC内容所遵循的JSON-LD标准
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  // 本VC的唯一标识，也就是证书ID
  "id": "uestc/alumni/12345",
  // VC内容的格式
  "type": ["VerifiableCredential", "AlumniCredential"],
  // 本VC的发行人
  "issuer": "did:cedu:xdu",
  // 本VC的发行时间
  "issuanceDate": "2020-07-01T19:73:24Z",
  // VC声明的具体内容
  "credentialSubject": {
    // 被声明的人的DID
    "id": "did:cid:511112200001010015",
    // 声明内容:毕业院校、专业、学位等
    "name":"小明",
    "alumniOf": {
      "id": "did:cedu:xdu",
      "name": [{
        "value": "西安电子科技大学",
        "lang": "cn"
      }]
    },
    "degree":"硕士研究生",
    "degreeType":"工科",
    "college":"网络与信息安全学院"
  },
  // 对本VC的证明
  "proof": {
    "creator": "did:cedu:xdu#keys-1",
    "type": "Secp256k1",
    "signatureValue": "3044022051757c2de7032a0c887c3fcef02ca3812fede7ca748254771b9513d8e2bb"
  }
}
```

这里最主要的就是credentialSubject证书的内容和proof颁发学校给出的证明。这个VC生成后会传给小明，小明可以选择将这个内容存储到自己的手机APP中，也可以选择存储到云上，以后需要使用时再读取。

## Holder小明提交学历证明VP给Verifier公司HR

接下来小明来到新公司入职，入职当天需要提交学历证明给公司HR，于是小明基于上一步骤生成的VC再封装一下，生成VP，VP的内容如下：

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
  "id": "uestc/alumni/12345",
  "type": ["VerifiableCredential", "AlumniCredential"],
  "issuer": "did:cedu:xdu",
  "issuanceDate": "2020-07-01T19:73:24Z",
  "credentialSubject": {
    "id": "did:cid:511112200001010015",
    "name":"小明",
    "alumniOf": {
      "id": "did:cedu:xdu",
      "name": [{
        "value": "西安电子科技大学",
        "lang": "cn"
      }]
    },
    "degree":"硕士研究生",
    "degreeType":"工科",
    "college":"网络与信息安全学院"
  },
  "proof": {
    "creator": "did:cedu:xdu#keys-1",
    "type": "Secp256k1",
    "signatureValue": "3044022051757c2de7032a0c887c3fcef02ca3812fede7ca748254771b9513d8e2bb"
  }
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

这里比较简单，只是简单的提交整个毕业证全部内容，所以VP中只需要把VC签入进去，最后proof的地方小明用自己APP里面的私钥进行签名，表名这个VP是小明自己生成的即可。这个VP生成后小明需要将整个VP的内容提交给新入职公司HR。

## Verifier公司HR验证通过小明的VP

公司HR作为验证者在收到小明提交的VP后首先验证Proof字段，保证VP是小明提交的，而且没有被篡改。接下来提取出其中的VC内容，对VC进行验证，验证过程如下：

1. 从proof的creator中获得颁发者的DID：did:cedu:xdu
2. 通过区块链查询到该DID的文档，在文档中有其创建人和公钥列表，其中我们取keys-1对应的公钥。
3. 创建人是did:corg:moe，是可信的DID，所以它创建的DID都是可信的。
4. 我们进一步的再用did:corg:moe去区块链读取DID文档，并获得其中的公钥，使用该公钥对did:cedu:xdu对应的文档进行签名验证，确保其没有被篡改。
5. 验证通过，我们再用did:cedu:xdu的公钥对小明提交的VC进行签名验证，验证通过，则说明这个证书确实是可信的UESTC颁发的。
6. HR检查VC中提供的内容，与小明提交的履历上是否一致，检查完成，进行下一步的入职手续。

以上验证过程中我们至少需要进行三次签名验证。验证VP是小明提交的，验证VC是XDU颁发的，验证XDU的DID是MOE创建的，而MOE是在验证方系统可信列表中的，所以整个就保证了小明提交的证书的可信。

## 小结

以上为简化版的DID从生成到**申请VC**再到**验证VP**的过程。