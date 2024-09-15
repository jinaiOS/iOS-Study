# StoreKit
- 앱을 대신하여 앱 스토어에 연결하여 결제를 요청하고 안전하게 처리합니다.
- 앱 스토어는 서버와도 통신할 수 있습니다.

# StoreKit Test
- 인앱 구매 테스트를 진행하는 방법입니다.

## StoreKit 파일 생성

<img width="1082" alt="스크린샷 2024-09-09 오후 1 27 49" src="https://github.com/user-attachments/assets/e614b901-4e0a-47ff-9aee-6c98aef5f338">

- 이미 앱 스토어 커넥트에 인앱 구매를 설정한 경우 `Sync this file with an app in App Store Connect` 를 체크해서 테스트 할 수 있습니다.

<img width="925" alt="스크린샷 2024-09-09 오후 1 31 05" src="https://github.com/user-attachments/assets/bb9b4df6-3874-4fe8-b2a3-41962a302fbc">


## StoreKit 파일 등록

<img width="387" alt="스크린샷 2024-09-09 오후 1 36 09" src="https://github.com/user-attachments/assets/68d64b96-c7fa-4b8d-be7c-8d104e4e8258">


- 생긴 storekit 파일의 왼쪽 아래를 보면 `+` 버튼이 있을 겁니다!
    - **`add consumable in-app purchase`** : 구매한 후 한 번만 사용할 수 있는 제품
    - **`add non-consumable in-app purchase`** : 한 번 구매하면 영구적으로 사용할 수 있는 제품
    - **`add non-renewing subscription`** : 갱신되지 않는 구독
    - **`add auto-renewable subscription`** : 자동 갱신 구독

## StoreKit 테스트 활성화

- `Product - Scheme - Edit Scheme - Run - Options - StoreKit Configuration` 을 만든 파일에 설정합니다.
- `none`으로 다시 변경하면 기본 상태로 복원할 수 있습니다.

<img width="1250" alt="스크린샷 2024-09-09 오후 1 24 49" src="https://github.com/user-attachments/assets/111f09b5-eb70-4322-b71d-64e0c8c9cbb1">


```swift
@MainActor
class StoreManager: ObservableObject {
    // 인앱 구매 제품 정보를 저장하는 배열
    @Published var products: [Product] = []
    
    // 제품 로드 메서드
    // 비동기적으로 지정된 제품 ID에 대한 제품 정보 가져옴
    func loadProducts() async {
        do {
            // 실제 사용할 제품 ID 목록 정의
            let productIDs = ["com.store.subscription"]
            
            // 제품 정보 가져옴
            products = try await Product.products(for: productIDs)
        } catch {
            print("Error loading products: \(error)")
        }
    }
    
    // 구매 메서드
    // 사용자가 특정 제품을 구매하려고 할 때 호출
    func purchase(product: Product) async {
        do {
            // 선택한 제품에 대한 구매 요청 시도
            let result = try await product.purchase()
            
            // 구매 결과에 따라 적절한 처리 수행
            switch result {
            case .success(let verificationResult):
                // 구매가 성공한 경우
                print("Purchase successful: \(verificationResult)")
            case .userCancelled:
                // 사용자가 구매를 취소한 경우
                print("User cancelled the purchase.")
            case .pending:
                // 구매가 대기 중인 경우
                print("Purchase is pending.")
            default:
                break
            }
        } catch {
            print("Purchase failed: \(error)")
        }
    }
}
```

```swift
Purchase successful: {
  "header" : {
    "alg" : "ES256",
    "kid" : "Apple_Xcode_Key",
    "typ" : "JWT",
    "x5c" : [
      "MIIBzDCCAXGgAwIBAgIBATAKBggqhkjOPQQDAjBIMSIwIAYDVQQDExlTdG9yZUtpdCBUZXN0aW5nIGluIFhjb2RlMSIwIAYDVQQKExlTdG9yZUtpdCBUZXN0aW5nIGluIFhjb2RlMB4XDTI0MDkxNTA2MzAzOFoXDTI1MDkxNTA2MzAzOFowSDEiMCAGA1UEAxMZU3RvcmVLaXQgVGVzdGluZyBpbiBYY29kZTEiMCAGA1UEChMZU3RvcmVLaXQgVGVzdGluZyBpbiBYY29kZTBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABLLnbBDRuQdoWJM+wEOJFyxKmnOKgZRrzJftGd8IasOpPWH7YwxAUAm+QK8RiOlfzBuw5ad9Df+MZapvYDzsCNyjTDBKMBIGA1UdEwEB\/wQIMAYBAf8CAQAwJAYDVR0RBB0wG4EZU3RvcmVLaXQgVGVzdGluZyBpbiBYY29kZTAOBgNVHQ8BAf8EBAMCB4AwCgYIKoZIzj0EAwIDSQAwRgIhAIhC7ZWM1lTIz+MYMey4w5BTzpO1m6O4AmwCp+EliWc6AiEA833ABUnSIvgAs1s0OEqsPVCgFqJP5htOhHWFLUJAqfw="
    ]
  },
  "payload" : {
    "bundleId" : "com.jinaiOS.Study-StoreKit",
    "currency" : "USD",
    "deviceVerification" : "1TVh5wPUJSDrfvqY4yAPA7wOrCl5e+X2UJmktxdJcwoGUVWhJf1A3KnFYeHhD9wu",
    "deviceVerificationNonce" : "4bc7eaee-4578-49a9-b410-7e56737d3d49",
    "environment" : "Xcode",
    "expiresDate" : 1728973838495,
    "inAppOwnershipType" : "PURCHASED",
    "isUpgraded" : false,
    "originalPurchaseDate" : 1726381838495,
    "originalTransactionId" : "0",
    "price" : 0,
    "productId" : "com.store.subscription",
    "purchaseDate" : 1726381838495,
    "quantity" : 1,
    "signedDate" : 1726381838503,
    "storefront" : "USA",
    "storefrontId" : "143441",
    "subscriptionGroupIdentifier" : "816BC19D",
    "transactionId" : "0",
    "transactionReason" : "PURCHASE",
    "type" : "Auto-Renewable Subscription",
    "webOrderLineItemId" : "0"
  },
  "signature" : "64 bytes (verified)"
}
```
