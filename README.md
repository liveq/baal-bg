# #18 배경 제거

**URL:** bg.baal.co.kr

## 서비스 내용

AI 기반 이미지 배경 제거. 사람/물체 자동 인식. 투명 PNG 다운로드. 브라우저에서 처리

## 기능 요구사항

- [ ] 이미지 업로드 (드래그 앤 드롭)
- [ ] AI 배경 제거 처리
- [ ] 객체 감지 (사람/물체)
- [ ] 배경 옵션:
  - [ ] 투명 (PNG)
  - [ ] 단색 배경 (색상 선택)
  - [ ] 흐림 효과 (Blur)
- [ ] Before/After 미리보기 (슬라이더)
- [ ] 다운로드 (PNG)
- [ ] 파일 크기 제한 (5MB)
- [ ] 처리 진행률 표시
- [ ] 에지 다듬기 (선택사항)
- [ ] 일괄 처리 (최대 5개, 선택사항)

## 경쟁사 분석 (2025년 기준)

### 인기 사이트 TOP 5

1. **Remove.bg** - 시장 1위
   - 강점: 정확도 최고, API 제공, 빠름
   - 약점: 무료는 50회/월, HD는 유료

2. **PhotoScissors** - Teorex 제품
   - 강점: AI + 수동 보정, 데스크톱 버전
   - 약점: 완전 유료, 복잡한 UI

3. **Slazzer** - AI 배경 제거
   - 강점: 정확도 높음, 대량 처리
   - 약점: 무료는 5회 제한, HD 유료

4. **Background Burner** - Bonfire 서비스
   - 강점: 완전 무료, 간단
   - 약점: 정확도 낮음, 광고 많음

5. **Clipping Magic** - 수동+AI
   - 강점: 정밀 편집 가능
   - 약점: 완전 유료, 학습 곡선

### 우리의 차별화 전략

- ✅ **완전 무료** - 횟수 제한 없음
- ✅ **브라우저 기반** - 서버 업로드 불필요, 프라이버시
- ✅ **AI 로컬 처리** - ONNX Runtime (클라이언트 사이드)
- ✅ **빠른 처리** - 10-30초 (이미지 크기에 따라)
- ✅ **다양한 출력** - 투명/단색/흐림
- ✅ **다크모드** 지원
- ✅ **간단한 UI** - 1클릭으로 완료

## 주요 라이브러리

### 옵션 1: @imgly/background-removal (추천!)

ONNX Runtime 기반 브라우저 AI 배경 제거

```html
<script type="module">
  import { removeBackground } from 'https://cdn.jsdelivr.net/npm/@imgly/background-removal@1.4.5/dist/index.mjs';
</script>
```

또는 NPM:
```bash
npm install @imgly/background-removal
```

### 기본 사용법

```javascript
import { removeBackground } from '@imgly/background-removal';

async function removeBgFromImage(imageFile) {
  try {
    // 진행률 콜백
    const config = {
      progress: (key, current, total) => {
        const percent = Math.round((current / total) * 100);
        updateProgress(`${key}: ${percent}%`);
      }
    };

    // 배경 제거 실행
    const blob = await removeBackground(imageFile, config);

    // 결과 미리보기
    const url = URL.createObjectURL(blob);
    document.getElementById('result-image').src = url;

    // 다운로드 준비
    return blob;

  } catch (error) {
    console.error('배경 제거 실패:', error);
    showToast('배경 제거에 실패했습니다.', 'error');
    throw error;
  }
}

// 사용 예
const fileInput = document.getElementById('image-input');
fileInput.addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (file) {
    showLoading('AI가 배경을 제거하는 중...');
    try {
      const resultBlob = await removeBgFromImage(file);
      hideLoading();
      showToast('배경 제거 완료!', 'success');
    } catch (error) {
      hideLoading();
    }
  }
});
```

### 고급 설정 (모델 선택, 품질)

```javascript
import { removeBackground, preload, Config } from '@imgly/background-removal';

// 모델 프리로드 (첫 실행 속도 개선)
async function preloadModel() {
  const config = {
    model: 'medium', // 'small', 'medium', 'large'
  };

  await preload(config);
  showToast('AI 모델 로딩 완료', 'success');
}

// 페이지 로드 시 모델 프리로드
window.addEventListener('load', () => {
  preloadModel();
});

// 배경 제거 (고급 옵션)
async function removeBgAdvanced(imageFile) {
  const config = {
    model: 'medium', // 모델 크기 (정확도 vs 속도)
    output: {
      format: 'image/png',
      quality: 0.9, // PNG 압축 품질
      type: 'blob'
    },
    progress: (key, current, total) => {
      const percent = Math.round((current / total) * 100);
      updateProgressBar(percent);

      // 단계별 메시지
      const messages = {
        'fetch:model': 'AI 모델 다운로드 중...',
        'compute:inference': '이미지 분석 중...',
        'compute:mask': '배경 감지 중...',
        'encode:result': '결과 생성 중...'
      };
      updateProgressMessage(messages[key] || '처리 중...');
    }
  };

  const blob = await removeBackground(imageFile, config);
  return blob;
}
```

### Canvas를 이용한 배경 색상 변경

```javascript
// 투명 배경 → 단색 배경
async function changeBackground(transparentBlob, backgroundColor = '#ffffff') {
  // 투명 이미지 로드
  const img = new Image();
  img.src = URL.createObjectURL(transparentBlob);
  await img.decode();

  // Canvas 생성
  const canvas = document.createElement('canvas');
  canvas.width = img.width;
  canvas.height = img.height;
  const ctx = canvas.getContext('2d');

  // 배경색 채우기
  ctx.fillStyle = backgroundColor;
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // 투명 이미지 그리기
  ctx.drawImage(img, 0, 0);

  // PNG로 변환
  const blob = await new Promise((resolve) => {
    canvas.toBlob(resolve, 'image/png');
  });

  URL.revokeObjectURL(img.src);
  return blob;
}

// 배경 흐림 효과
async function blurBackground(originalImage, maskBlob) {
  // 원본 이미지 로드
  const original = new Image();
  original.src = URL.createObjectURL(originalImage);
  await original.decode();

  // 마스크 로드
  const mask = new Image();
  mask.src = URL.createObjectURL(maskBlob);
  await mask.decode();

  // Canvas 생성
  const canvas = document.createElement('canvas');
  canvas.width = original.width;
  canvas.height = original.height;
  const ctx = canvas.getContext('2d');

  // 배경 흐림 효과
  ctx.filter = 'blur(10px)';
  ctx.drawImage(original, 0, 0);

  // 전경 (사람) 그리기
  ctx.filter = 'none';
  ctx.globalCompositeOperation = 'destination-in';
  ctx.drawImage(mask, 0, 0);

  ctx.globalCompositeOperation = 'source-over';

  const blob = await new Promise((resolve) => {
    canvas.toBlob(resolve, 'image/png');
  });

  return blob;
}
```

### 옵션 2: TensorFlow.js + BodyPix

사람 세그멘테이션 전용 (성능 낮음, 대안)

```html
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.11.0"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/body-pix@2.2.0"></script>
```

```javascript
// BodyPix 모델 로드
let bodyPixModel = null;

async function loadBodyPix() {
  bodyPixModel = await bodyPix.load({
    architecture: 'MobileNetV1',
    outputStride: 16,
    multiplier: 0.75,
    quantBytes: 2
  });
  console.log('BodyPix 모델 로딩 완료');
}

// 배경 제거 (사람만)
async function removeBackgroundBodyPix(imageFile) {
  if (!bodyPixModel) {
    await loadBodyPix();
  }

  // 이미지 로드
  const img = new Image();
  img.src = URL.createObjectURL(imageFile);
  await img.decode();

  // 세그멘테이션 실행
  const segmentation = await bodyPixModel.segmentPerson(img, {
    flipHorizontal: false,
    internalResolution: 'medium',
    segmentationThreshold: 0.7
  });

  // Canvas 생성
  const canvas = document.createElement('canvas');
  canvas.width = img.width;
  canvas.height = img.height;
  const ctx = canvas.getContext('2d');

  // 원본 그리기
  ctx.drawImage(img, 0, 0);

  // 마스크 데이터
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  const pixels = imageData.data;

  // 배경 투명 처리
  for (let i = 0; i < segmentation.data.length; i++) {
    if (segmentation.data[i] === 0) { // 배경
      pixels[i * 4 + 3] = 0; // Alpha = 0
    }
  }

  ctx.putImageData(imageData, 0, 0);

  const blob = await new Promise((resolve) => {
    canvas.toBlob(resolve, 'image/png');
  });

  URL.revokeObjectURL(img.src);
  return blob;
}
```

**@imgly/background-removal vs TensorFlow BodyPix:**

| 특징 | @imgly/background-removal | BodyPix |
|------|---------------------------|---------|
| 정확도 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 속도 | 10-30초 | 5-15초 |
| 모델 크기 | 20-40MB | 10-20MB |
| 대상 | 사람 + 물체 | 사람만 |
| 추천 | **일반적** | 사람 전용 |

## UI/UX 디자인 패턴

### 화면 구성

```
┌─────────────────────────────────┐
│  배경 제거                        │
│  AI로 자동 배경 투명화             │
├─────────────────────────────────┤
│  1️⃣ 이미지 업로드                 │
│  ┌───────────────────────────┐  │
│  │  🖼️ 드래그 앤 드롭         │  │
│  │  또는 클릭하여 선택         │  │
│  │  (최대 5MB)               │  │
│  └───────────────────────────┘  │
├─────────────────────────────────┤
│  2️⃣ 배경 옵션 선택                │
│  ● 투명 (PNG)                   │
│  ○ 단색 배경: [⚪ 색상 선택]     │
│  ○ 흐림 효과                    │
├─────────────────────────────────┤
│  3️⃣ 처리 진행률                   │
│  [━━━━━━━●──────] 75%          │
│  AI 모델 분석 중...              │
├─────────────────────────────────┤
│  4️⃣ 결과 미리보기                 │
│  [원본] ← ━●━━ → [결과]         │
│  (슬라이더로 비교)               │
│                                 │
│  [다운로드 PNG]                  │
└─────────────────────────────────┘
```

### Before/After 슬라이더

```html
<div class="comparison-container">
  <div class="comparison-slider">
    <div class="before-image" style="background-image: url(original.jpg)">
      <span class="label">원본</span>
    </div>
    <div class="after-image" style="background-image: url(removed.png); clip-path: inset(0 50% 0 0)">
      <span class="label">배경 제거</span>
    </div>
    <input type="range" min="0" max="100" value="50" class="slider" id="comparison-slider">
  </div>
</div>

<style>
.comparison-container {
  position: relative;
  width: 100%;
  max-width: 800px;
}

.comparison-slider {
  position: relative;
  width: 100%;
  height: 500px;
  overflow: hidden;
}

.before-image, .after-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
}

.slider {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  width: 80%;
  z-index: 10;
}
</style>

<script>
const slider = document.getElementById('comparison-slider');
const afterImage = document.querySelector('.after-image');

slider.addEventListener('input', (e) => {
  const value = e.target.value;
  afterImage.style.clipPath = `inset(0 ${100 - value}% 0 0)`;
});
</script>
```

### 주요 기능 순서

1. 이미지 업로드 (드래그 앤 드롭)
2. 배경 옵션 선택 (투명/단색/흐림)
3. AI 처리 (진행률 표시)
4. Before/After 미리보기
5. 다운로드 (PNG)

## 난이도 & 예상 기간

- **난이도:** 어려움 (AI 모델, 성능 최적화)
- **예상 기간:** 3.5일
- **실제 기간:** (작업 후 기록)

## 개발 일정

- [ ] Day 1 오전: UI 구성, 이미지 업로드
- [ ] Day 1 오후: @imgly/background-removal 통합, 기본 배경 제거
- [ ] Day 2 오전: 진행률 표시, 에러 핸들링
- [ ] Day 2 오후: Before/After 슬라이더, 미리보기
- [ ] Day 3 오전: 배경 옵션 (단색, 흐림), Canvas 처리
- [ ] Day 3 오후: 성능 최적화, 메모리 관리, 테스트

## 트래픽 예상

⭐⭐⭐⭐ 매우 높음 - "배경 제거" 검색량 폭발적 (전자상거래, 콘텐츠 제작)

## SEO 키워드

- 배경 제거
- 이미지 배경 투명
- AI 배경 제거
- 무료 배경 제거
- Remove Background
- 누끼 따기
- 사진 배경 삭제
- PNG 투명 배경

## 이슈 & 해결방안

### 실제 문제점 (경쟁사 분석 & @imgly/background-removal 기반)

1. **모델 다운로드 시간 (20-40MB)**
   - 원인: ONNX 모델 파일 크기
   - 해결: 첫 실행 시 모델 프리로드
   - 해결: 로딩 메시지 표시 (투명하게 안내)
   - 코드:
     ```javascript
     import { preload } from '@imgly/background-removal';

     let modelLoaded = false;

     async function initializeModel() {
       if (modelLoaded) return;

       showToast('AI 모델을 다운로드하는 중... (최초 1회)', 'info');

       try {
         await preload({
           model: 'medium',
           progress: (key, current, total) => {
             const percent = Math.round((current / total) * 100);
             updateLoadingProgress(`모델 다운로드: ${percent}%`);
           }
         });

         modelLoaded = true;
         showToast('AI 준비 완료!', 'success');
         hideLoadingProgress();

       } catch (error) {
         showToast('모델 다운로드 실패. 네트워크를 확인하세요.', 'error');
       }
     }

     // 페이지 로드 시 백그라운드에서 프리로드
     window.addEventListener('load', () => {
       setTimeout(initializeModel, 1000);
     });
     ```

2. **처리 시간이 너무 길다 (10-30초)**
   - 원인: AI 추론이 CPU 집약적
   - 해결: 진행률 표시로 사용자 대기감 완화
   - 해결: 이미지 크기 자동 조정 (최대 2000px)
   - 코드:
     ```javascript
     const MAX_DIMENSION = 2000;

     async function preprocessImage(file) {
       const img = new Image();
       img.src = URL.createObjectURL(file);
       await img.decode();

       // 크기 확인
       if (img.width <= MAX_DIMENSION && img.height <= MAX_DIMENSION) {
         return file; // 원본 사용
       }

       // 리사이즈 필요
       showToast('이미지가 큽니다. 처리 속도 향상을 위해 리사이즈합니다.', 'info');

       const canvas = document.createElement('canvas');
       const ratio = Math.min(MAX_DIMENSION / img.width, MAX_DIMENSION / img.height);
       canvas.width = img.width * ratio;
       canvas.height = img.height * ratio;

       const ctx = canvas.getContext('2d');
       ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

       const resizedBlob = await new Promise((resolve) => {
         canvas.toBlob(resolve, 'image/jpeg', 0.9);
       });

       URL.revokeObjectURL(img.src);
       return resizedBlob;
     }

     // 사용 예
     const file = e.target.files[0];
     const processedFile = await preprocessImage(file);
     const result = await removeBackground(processedFile);
     ```

3. **메모리 부족으로 브라우저 크래시**
   - 원인: 대용량 이미지 + AI 모델
   - 해결: 파일 크기 제한 (5MB)
   - 해결: 이미지 크기 제한 (3000x3000px)
   - 코드:
     ```javascript
     const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
     const MAX_IMAGE_SIZE = 3000;

     async function validateImage(file) {
       // 파일 크기 검증
       if (file.size > MAX_FILE_SIZE) {
         showToast('파일이 너무 큽니다. (최대 5MB)', 'error');
         return false;
       }

       // 이미지 크기 검증
       const img = new Image();
       img.src = URL.createObjectURL(file);
       await img.decode();

       if (img.width > MAX_IMAGE_SIZE || img.height > MAX_IMAGE_SIZE) {
         showToast(`이미지 크기가 너무 큽니다. (최대 ${MAX_IMAGE_SIZE}x${MAX_IMAGE_SIZE})`, 'error');
         URL.revokeObjectURL(img.src);
         return false;
       }

       URL.revokeObjectURL(img.src);
       return true;
     }

     // 사용 예
     const file = e.target.files[0];
     if (await validateImage(file)) {
       await removeBgFromImage(file);
     }
     ```

4. **복잡한 배경에서 정확도 낮음**
   - 원인: AI 모델 한계 (머리카락, 투명 물체)
   - 해결: 사용자에게 안내 (단순한 배경 권장)
   - 해결: 모델 크기 선택 옵션 (small/medium/large)
   - 코드:
     ```javascript
     // UI 안내
     const tips = [
       '단순한 배경에서 가장 정확합니다.',
       '사람과 배경의 대비가 클수록 좋습니다.',
       '머리카락이 복잡하면 다듬기가 필요할 수 있습니다.'
     ];

     // 모델 크기 선택
     const modelSelector = document.getElementById('model-size');
     modelSelector.addEventListener('change', (e) => {
       const size = e.target.value; // 'small', 'medium', 'large'

       if (size === 'large') {
         showToast('정확도가 높아지지만 처리 시간이 길어집니다.', 'info');
       } else if (size === 'small') {
         showToast('빠르지만 정확도가 낮을 수 있습니다.', 'info');
       }
     });

     // 배경 제거 시 모델 크기 적용
     const config = {
       model: modelSelector.value
     };
     const result = await removeBackground(file, config);
     ```

5. **투명 배경 PNG가 너무 크다**
   - 원인: PNG 압축 부족
   - 해결: 품질 조정 (0.8-0.9)
   - 해결: pngquant 같은 추가 압축 (선택사항)
   - 코드:
     ```javascript
     // PNG 품질 조정
     const config = {
       output: {
         format: 'image/png',
         quality: 0.8 // 0.1~1.0 (낮을수록 작은 용량)
       }
     };

     const blob = await removeBackground(file, config);

     // 파일 크기 확인
     const sizeMB = (blob.size / 1024 / 1024).toFixed(2);
     showToast(`결과 파일 크기: ${sizeMB}MB`, 'info');

     // 너무 크면 경고
     if (blob.size > 10 * 1024 * 1024) {
       showToast('파일이 큽니다. 품질을 낮추시겠습니까?', 'warning');
     }
     ```

6. **에지(테두리)가 거칠다**
   - 원인: 세그멘테이션 정확도
   - 해결: Canvas로 에지 부드럽게 처리 (feathering)
   - 코드:
     ```javascript
     async function smoothEdges(transparentBlob, featherPixels = 2) {
       const img = new Image();
       img.src = URL.createObjectURL(transparentBlob);
       await img.decode();

       const canvas = document.createElement('canvas');
       canvas.width = img.width;
       canvas.height = img.height;
       const ctx = canvas.getContext('2d');

       ctx.drawImage(img, 0, 0);

       // 에지 부드럽게 (간단한 blur)
       if (featherPixels > 0) {
         ctx.filter = `blur(${featherPixels}px)`;
         const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
         ctx.putImageData(imageData, 0, 0);
       }

       const smoothBlob = await new Promise((resolve) => {
         canvas.toBlob(resolve, 'image/png');
       });

       URL.revokeObjectURL(img.src);
       return smoothBlob;
     }

     // 사용 예
     const rawResult = await removeBackground(file);
     const smoothResult = await smoothEdges(rawResult, 2);
     ```

7. **WebGL 지원 안되는 브라우저**
   - 원인: 구형 브라우저, WebGL 비활성화
   - 해결: WebGL 지원 체크 후 폴백 안내
   - 코드:
     ```javascript
     function checkWebGLSupport() {
       try {
         const canvas = document.createElement('canvas');
         const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
         return !!gl;
       } catch (e) {
         return false;
       }
     }

     // 페이지 로드 시 체크
     if (!checkWebGLSupport()) {
       showToast('WebGL이 지원되지 않습니다. 최신 브라우저를 사용하세요.', 'error');
       document.getElementById('upload-section').style.display = 'none';
       document.getElementById('not-supported').style.display = 'block';
     }
     ```

8. **일괄 처리 시 메모리 폭증**
   - 원인: 여러 이미지 동시 처리
   - 해결: 순차 처리 (한 번에 1개)
   - 해결: 처리 후 메모리 정리
   - 코드:
     ```javascript
     async function batchRemoveBackground(files) {
       const results = [];
       const maxFiles = Math.min(files.length, 5); // 최대 5개

       for (let i = 0; i < maxFiles; i++) {
         const file = files[i];

         try {
           updateProgress(`처리 중: ${i + 1} / ${maxFiles}`);

           const blob = await removeBackground(file);
           results.push({ name: file.name, blob });

           // 메모리 정리
           URL.revokeObjectURL(file);
           await new Promise(resolve => setTimeout(resolve, 500));

         } catch (error) {
           console.error(`${file.name} 처리 실패:`, error);
           results.push({ name: file.name, error });
         }
       }

       return results;
     }
     ```

9. **모바일에서 메모리 부족**
   - 원인: 모바일 RAM 제한 (2-4GB)
   - 해결: 모바일은 small 모델 자동 선택
   - 해결: 이미지 크기 더 작게 (1500px)
   - 코드:
     ```javascript
     function isMobile() {
       return /Android|iPhone|iPad|iPod/i.test(navigator.userAgent);
     }

     function getOptimalConfig() {
       if (isMobile()) {
         return {
           model: 'small',
           maxDimension: 1500,
           maxFileSize: 3 * 1024 * 1024 // 3MB
         };
       } else {
         return {
           model: 'medium',
           maxDimension: 2000,
           maxFileSize: 5 * 1024 * 1024 // 5MB
         };
       }
     }

     // 사용 예
     const config = getOptimalConfig();
     showToast(isMobile() ? '모바일 최적화 모드' : '데스크톱 모드', 'info');
     ```

10. **CORS 문제 (외부 이미지 URL)**
    - 원인: 외부 URL 이미지 로드 시 CORS
    - 해결: 파일 업로드만 지원 (URL 입력 불가)
    - 해결: 또는 프록시 서버 사용 (복잡)
    - 코드:
      ```javascript
      // URL 입력 기능 (CORS 주의)
      async function loadImageFromURL(url) {
        try {
          // fetch로 Blob 가져오기 (CORS 허용 시)
          const response = await fetch(url, {
            mode: 'cors',
            credentials: 'omit'
          });

          if (!response.ok) {
            throw new Error('이미지 로드 실패');
          }

          const blob = await response.blob();
          return blob;

        } catch (error) {
          showToast('외부 이미지는 CORS 제한으로 불가능합니다. 파일 업로드를 사용하세요.', 'error');
          throw error;
        }
      }

      // 권장: 파일 업로드만 허용
      document.getElementById('url-input').style.display = 'none';
      ```

## 통합 전략

### 전자상거래 연계

**메시지:**
- "상품 사진 배경 제거"
- "쇼핑몰 이미지 편집"

**워크플로우:**
1. 상품 사진 촬영
2. bg.baal.co.kr에서 배경 제거
3. 흰색 배경으로 변경
4. 쇼핑몰에 업로드

### 다른 도구와 연계

**resize.baal.co.kr (리사이즈):**
- 배경 제거 → 리사이즈 (썸네일 생성)

**compress.baal.co.kr (압축):**
- 배경 제거 → 압축 (용량 절감)

## 추가 기능 (선택사항)

### 1. 배경 템플릿 제공

```javascript
const backgroundTemplates = {
  white: '#ffffff',
  black: '#000000',
  gradient: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
  blur: 'original-blur'
};

// UI에서 템플릿 선택
document.getElementById('template-selector').addEventListener('change', async (e) => {
  const template = e.target.value;
  const newBackground = await applyBackground(transparentBlob, backgroundTemplates[template]);
  previewResult(newBackground);
});
```

### 2. 히스토리 저장

```javascript
const history = JSON.parse(localStorage.getItem('bg-removal-history') || '[]');
history.unshift({
  date: new Date().toISOString(),
  filename: file.name,
  size: blob.size
});
localStorage.setItem('bg-removal-history', JSON.stringify(history.slice(0, 10)));
```

### 3. 드래그 앤 드롭 개선

```javascript
const dropZone = document.getElementById('drop-zone');

dropZone.addEventListener('dragover', (e) => {
  e.preventDefault();
  dropZone.classList.add('drag-over');
});

dropZone.addEventListener('dragleave', () => {
  dropZone.classList.remove('drag-over');
});

dropZone.addEventListener('drop', async (e) => {
  e.preventDefault();
  dropZone.classList.remove('drag-over');

  const files = e.dataTransfer.files;
  if (files.length > 0) {
    const file = files[0];
    if (file.type.startsWith('image/')) {
      await removeBgFromImage(file);
    } else {
      showToast('이미지 파일만 지원합니다.', 'error');
    }
  }
});
```

## 개발 로그

### 2025-10-25
- 프로젝트 폴더 생성
- README.md 작성
- **경쟁사 분석 완료:**
  - Remove.bg, PhotoScissors, Slazzer, Background Burner, Clipping Magic 조사
  - 대부분 횟수 제한 또는 유료, HD는 유료
  - 차별화: 완전 무료, 브라우저 AI, 프라이버시
- **라이브러리 조사 완료:**
  - @imgly/background-removal (추천) - ONNX Runtime, 정확도 최고
  - TensorFlow.js BodyPix - 사람 전용, 성능 낮음
  - Best practices: 모델 프리로드, 이미지 크기 제한, 진행률 표시
- **실제 이슈 파악:**
  - 모델 다운로드 시간, 처리 시간, 메모리 부족
  - 정확도 문제, PNG 용량, 에지 처리
- **UI/UX 패턴:**
  - 드래그 앤 드롭, Before/After 슬라이더
  - 배경 옵션 (투명/단색/흐림), 진행률 표시

## 참고 자료

- [@imgly/background-removal GitHub](https://github.com/imgly/background-removal-js)
- [@imgly/background-removal NPM](https://www.npmjs.com/package/@imgly/background-removal)
- [ONNX Runtime Web](https://onnxruntime.ai/docs/tutorials/web/)
- [TensorFlow.js BodyPix](https://github.com/tensorflow/tfjs-models/tree/master/body-pix)
- [Remove.bg API 문서](https://www.remove.bg/api)
- [Image Segmentation Best Practices](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
