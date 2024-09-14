# AVFoundation 알아보기

# 기능

- 미디어 재생 - 오디오 및 비디오 파일을 재생
- 미디어 녹화 - 카메라 및 마이크를 사용하여 오디오 및 비디오를 녹화
- 미디어 편집 - AVAsset 및 AVAssetTrack 클래스를 사용하여 미디어 파일을 자르거나 병합, 수정이 가능
- 미디어 스트리밍 - 네트워크를 통해 미디어를 스트리밍 가능
- 메타데이터 관리 - 미디어 파일의 메타데이터를 읽거나 쓰기 가능

![](https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/AVFoundationPG/Art/frameworksBlockDiagram_2x.png)

## **AVAsset**

- url로 미디어를 객체화하는 역할
- track - AVAsset에 있는 미디어 데이터의 각 부분
- 일반적으로  AVAsset은 데이터가 큼 → 비동기 처리 필요

## **AVKit**

- AVKit은 AVFoundation위에 존재
- 미디어 플레이어 Interface를 쉽게 제공하는 SDK
- 화면에 자막을 표출하는 기능 제공