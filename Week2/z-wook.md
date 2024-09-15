### 개요
대부분의 경우 사용자에게 적절한 보안을 제공하는 것이 필수적입니다.
이는 E2EE(End-to-End Encryption) 암호화와 같은 암호화 방법을 사용하여 구현할 수 있습니다.
<br>

### [종단간 암호화(E2EE)](https://docs.tosspayments.com/resources/glossary/e2ee)
종단간 암호화는 **메시지를 처음부터 끝까지 평문으로 저장하지 않고 암호화하는 안전한 통신 방법**입니다. 
클라이언트에서 메시지를 전송하는 단계부터 최종적으로 수신자에 전달되는 단계까지 메시지를 암호화하기 때문에 'End to End Encryption(E2EE)'으로 불리기도 합니다.

종단간 암호화 방법은 메시지를 복호화할 수 있는 키를 서버에 저장하지 않고, 수신자 장치에서 저장합니다. 그래서 수신자에게 발송되는 동안 데이터가 제 3자에 의해 읽히거나 조작될 수 없습니다. 데이터를 전달하는 서버조차 데이터를 복호화할 수 없습니다.

[E2EE에 대한 자세한 설명을 보려면 클릭해 주세요.](https://www.cloudflare.com/ko-kr/learning/privacy/what-is-end-to-end-encryption/)

---

### [CryptoKit](https://developer.apple.com/documentation/cryptokit/)


CryptoKit은 애플이 암호화를 위해 제공하는 프레임워크 입니다.


---

### Salt
```swift
// salt 값
let salt = "메시지 암호화!".data(using: .utf8)
```
<br>


### 개인키 생성
```swift
func generatePrivateKey() -> Curve25519.KeyAgreement.PrivateKey {
    let privateKey = Curve25519.KeyAgreement.PrivateKey()
    return privateKey
}
```
<br>


### 공개키 생성
```swift
func generatePublicKeyData(_ privateKey: Curve25519.KeyAgreement.PrivateKey) -> Data {
    let publicKeyData = privateKey.publicKey.rawRepresentation
    return publicKeyData
}
```
<br>


### 대칭키 생성
```swift
func generateSymmetricKey(privateKey: Curve25519.KeyAgreement.PrivateKey, 
                          publicKey: Curve25519.KeyAgreement.PublicKey) throws -> SymmetricKey {
    let sharedSecret = try privateKey.sharedSecretFromKeyAgreement(with: publicKey)
    let symmetricKey = sharedSecret.hkdfDerivedSymmetricKey(
        using: SHA256.self,
        salt: salt!,
        sharedInfo: Data(),
        outputByteCount: 32
    )
    
    return symmetricKey
}
```
<br>


### 암호화
```swift
func encrypt(text: String, 
             symmetricKey: SymmetricKey) throws -> ChaChaPoly.SealedBox {
    let textData = text.data(using: .utf8)!
    let encryptedData = try ChaChaPoly.seal(textData, using: symmetricKey).combined
    let encryptedSealBox = try ChaChaPoly.SealedBox(combined: encryptedData)
    
    return encryptedSealBox
}
```
<br>


### 복호화
```swift
func decrypt(sealBox: ChaChaPoly.SealedBox, 
             symmetricKey: SymmetricKey) -> String {
    do {
        let decryptedData = try ChaChaPoly.open(sealBox, using: symmetricKey)
        guard let text = String(data: decryptedData, encoding: .utf8) else {
            return "Could not decode data : \(decryptedData)"
        }
        return text
        
    } catch {
        return "error : \(error)"
    }
}
```

---

### E2EE 사용 예시(채팅)

```swift
// MARK: - 유저 생성 시 Public, Private Key 생성
let karinaPrivateKey = generatePrivateKey()
let karinaPublicKeyData = generatePublicKeyData(karinaPrivateKey)

let jiwookPrivateKey = generatePrivateKey()
let jiwookPublicKeyData = generatePublicKeyData(jiwookPrivateKey)

let messageText = "안녕하세요~ !"
var sealBox: ChaChaPoly.SealedBox?

// MARK: - message 송신 (jiwook -> karina)
do {
    // 대칭키 생성
    let karinaPublicKey = try Curve25519.KeyAgreement.PublicKey(rawRepresentation: karinaPublicKeyData)
    let jiwookSymmetricKey = try generateSymmetricKey(privateKey: jiwookPrivateKey,
                                                      publicKey: karinaPublicKey)
    // message 암호화
    let jiwookSealBox = try encrypt(text: messageText,
                                    symmetricKey: jiwookSymmetricKey)
    
    // 서버로 jiwookSealBox 전송(해당 예시에서는 서버 전송 대신 로컬로 표현)
    sealBox = jiwookSealBox
    
} catch {
    print("error : \(error)")
}

// MARK: - message 수신 (karina <- jiwook)
// 서버에 전송된 jiwookSealBox를 karina가 송신받은 상황
do {
    // 대칭키 생성
    let jiwookPublicKey = try Curve25519.KeyAgreement.PublicKey(rawRepresentation: jiwookPublicKeyData)
    let karinaSymmetricKey = try generateSymmetricKey(privateKey: karinaPrivateKey, 
                                                      publicKey: jiwookPublicKey)
    
    if let sealBox {
        let message = decrypt(sealBox: sealBox, 
                              symmetricKey: karinaSymmetricKey)
        print("receive message : \(message)")
    }
} catch {
    print("error : \(error)")
}
```

![](https://velog.velcdn.com/images/oasis444/post/34c13ace-64a0-42d1-9b1d-6ae4ea17dc0e/image.png)

---

### 참고자료
https://getstream.io/blog/ios-cryptokit-framework-chat/

https://dadahae0320.tistory.com/48
