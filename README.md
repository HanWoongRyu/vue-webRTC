# peerjs-vue

## 프로젝트 개요
이 Vue 애플리케이션은 PeerJS를 사용하여 피어-투-피어 비디오 통신을 가능하게 합니다. 사용자는 비디오 통화를 시작하고 수신할 수 있으며 오디오를 음소거하거나 취소하고 비디오를 활성화하거나 비활성화할 수 있는 옵션이 있습니다. 앱은 사용 가능한 미디어 입력 장치를 동적으로 가져오고 사용합니다.

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### 컴포넌트
App.vue
- 피어 연결을 초기화하고 관리하는 주요 컴포넌트입니다. 로컬 및 원격 미디어 스트림을 처리하고 통화 기능을 제어하는 UI를 제공합니다.

CallControls.vue
- 통화를 시작하고 카메라 및 마이크를 토글하는 버튼을 제공합니다.

VideoDisplay.vue
- 로컬 및 원격 비디오 스트림을 표시합니다.

### USES 
- 피어 연결 설정 : 피어 연결을 초기화하려면 사용자 정의 피어 ID를 입력하세요.
- 드롭다운에서 원하는 오디오 및 비디오 입력 장치를 선택하세요.
- 통화하기 :통화하고자 하는 사용자의 피어 ID를 입력하고 "통화" 버튼을 누르세요. 통화 중에 제공된 컨트롤을 사용하여 카메라와 마이크를 토글할 수 있습니다.

### Dependencies
- Vue.js
- PeerJS

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
