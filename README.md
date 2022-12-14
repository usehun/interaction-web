# interaction-web

### 각 스크롤 섹션의 높이 세팅

    sceneInfo[i].scrollHeight = sceneInfo[i].heightNum(5) * window.innerHeight;
    윈도우 높이에 heightNum = 5를 곱하여 sceneInfo[i] 배열마다 높이를 세팅.

    type: "sticky" 와 type: "normal" = sticky는 고정 normal은 스크롤이 올라가도록 세팅.

### 스크롤 이벤트 추가

    윈도우 이벤트 scroll, resize를 사용하여 스크롤과 화면 크기 변환 시 이벤트가 적용되도록 세팅.\

### 전체 스크롤 높이중 현재 씬의 높이 계산

    let prevScrollHeight = 0;
    현재 스크롤 위치 (yOffset)보다 이전에 위치한 스크롤 섹션들의 높이값의 합

    let currentScene = 0;
    현재 활성화된(눈 앞에 보고있는) 씬(scroll-section)

    prevScrollHeight += sceneInfo[i].scrollHeight;
    반복문을 사용하여 섹션들의 높이를 더해주었다.

### 섹션 구분 수정, 바운스 방지

    yOffset > prevScrollHeight + sceneInfo[currentScene].scrollHeight
    총 스크롤 한 높이가 현재를 제외한 이전 스크롤된 총 높이와 현재 섹션의 높이 합보다 클 경우 currentScene++ 현재 씬을 올려주고

    yOffset < prevScrollHeight
    총 스크롤 한 높이가 현재를 제외한 이전 스크롤된 총 높이보다 작을 경우 currentScene-- 현재 씬을 내린다.
    if (currentScene === 0) return; 현재 씬이 0일 경우 리턴하여 0보다 아래 값을 가질 수 없도록 한다. (바운스 경우)

### 화면 크기에 따른 캔버스 크기 작업

<img width="592" alt="screen-size" src="https://user-images.githubusercontent.com/88027485/205633914-0bcae537-a92c-4a9b-b43f-c63c49c53e2f.png">
    
    Window : 600 * 900 = 6:9
    Canvas : 1920 * 1080 = 16:9

    현재 창 사이즈와 캔버스 사이즈가 이렇다고 할 때,

    const recalculatedInnerWidth(0.31) = window.innerWidth(600) / canvasScaleRatio(1920);
    const recalculatedInnerHeight(0.83) = window.innerHeight(900) / canvasScaleRatio(1080);

    let canvasScaleRatio;
    if (widthRatio <= heightRatio) {
        // 캔버스보다 브라우저 창이 홀쭉한 경우
        canvasScaleRatio = heightRatio;
        console.log("heightRatio로 결정");
    } else {
        // 캔버스보다 브라우저 창이 납작한 경우
        canvasScaleRatio = widthRatio;
        console.log("widthRatio로 결정");
    }

    여기서 heightRatio가 더 크므로 scale(0.83)을 적용하면
    1920 * 0.833 = 1600, 1080 * 0.833 = 900, 즉 1600 * 900의 크기가 된다. (윈도우의 6:9 비율에 맞춰 캔버스의 크기가 줄어듦)


    이 1600 * 900 캔버스의 원본 높이를 유지한 채 ratio를 16:9(캔버스) → 6:9(윈도우)로 변환하면 720 * 1080이 된다. (윈도우와 비슷하게 6:9 비율로 홀쭉해짐)
    이걸 구하는 게 600 / 0.833, 900 / 0.833 계산 식. 줄어든 scale 만큼 나눠서 원래 비율을 찾는다.

<img width="869" alt="screen-size2" src="https://user-images.githubusercontent.com/88027485/205633920-717262e8-2896-48a8-bdf9-90f733123f21.png">

### 섹션2 -> 섹션3 갑작스러운 이미지 노출 수정, 스크롤에 따른 이미지 블렌딩 준비

    섹션2에서 scrollRatio > 0.9 일때 섹션 3의 애니메이션을 뺀 캔버스 이미지를 사용하여 섹션3에서 갑작스럽게 나타나는 이미지 노출을 수정

    섹션3에서 이미지가 윈도우 상단에 닿기 전과 후로 나누어 조건문을 만들고 CSS sticky 클래스를 추가하거나 제거한다.

    상단에 닿은 후 (step = 2)를 주고
    objs.canvas.style.top = `${-(objs.canvas.height - objs.canvas.height * canvasScaleRatio) / 2}px`
    Transform으로 줄어든 캔버스 비율에 맞게 계산하여 캔버스의 상단 위치를 수정한다.

### 이미지 블렌딩 추가

    values.rect1X[2].end 로 주어 이미지가 전체 화면에 꽉찬 끝나는 지점을 이미지 블렌딩 시작 지점으로 선택한다.

    drawImage 함수의 현재 이미지의 원하는 영역과 넓이, 높이를 캔버스 이미지의 영역에 넓이, 높이를 그린다.

    스크롤에 따른 이미지를 그리는 것이고 현재 이미지의 높이 - blendHeight를 Y값 시작점으로 두어 밑에서 부터 이미지를 그린다.

### 캔버스 스크롤과 크기변경

    이미지 블렌딩 끝부분보다 스크롤비율이 더 크다면 이미지 블렌딩이 끝났다는 것을 나타내기 때문에
    캔버스의 크기를 변경해주고 스크롤의(0.4배) 된 만큼 마진 탑을 주도록 한다.

    캔버스 고정을 풀고 스크롤처리 후 밑에 있는 글 내용을 translate, opacity를 스크롤에 따른 효과를 추가해준다.

    네비게이션 바에 백드롭 필터를 추가하였고 로컬 네비게이션은 스크롤 후에도 고정을 두었다.

### 부드러운 감속 비디오 적용

    let acc = 0.1; // 가속도
    let delayedYOffset = 0;

    delayedYOffset = yOffset + (yOffset - delayedYOffset) * acc;

    현재 거리와 이동후 스크롤간의 거리에 0.1씩 곱하며 현재 거리를 더해는 방법으로 부드러운 감속 이미지를 만들어준다.

    조건문을 사용하여 절대값을 씌운 거리오차가 1미만일 경우 requestAnimationFrame 작업을 멈춰주고(성능 개선) rafState = false;
    스크롤 이벤트에 !rafState 일때 다시 requestAnimationFrame 실행하고 rafState = ture;

### 디버깅

    Scene이 바뀌는 순간에 건너뛰는 조건문 추가. (enterNewScene = false)

    scene의 변경이 yOffset으로 인해 발생하여 prevScrollHeight의 시점이 잠깐동안 delayedYOffset에 맞춰주지 못해 yOffset -> delayedYOffset 변경

    윈도우의 width가 600보다 클 경우 resize 이벤트를 실행하고
    모바일 기기의 방향을 회전했을 때 resize 이벤트를 실행한다.

    sceneInfo[3] case3에서 창사이즈 변경이 일어났을 때 크기가 안 맞는 상황을 대비하여
    sceneInfo[3].values.rectStartY = 0을 resize 이벤트를 실행할 때 지정해준다.

### 로딩 애니메이션, 페이지 로드 디버깅

    로딩 애니메이션
    HTML에 SVG 태그와 SVG 태그 안 circle 태그를 만든다.
    CSS로 circle 클래스의 반지름과 크기에 맞게 원을 그리고 stroke-dashoffset을 이용하여 애니메이션을 준다.

    중간 스크롤에서 페이지 새로고침 했을 때 이미지 로드나 컨텐츠 높이로 인한 오류 수정

    페이지 로드 후 리사이즈 했을 때 오류 처리

    스크롤 섹션이 끝난 후 일반 섹션이 들어왔을 때 처리

#### 2022-12-20

#### 2022-12-13 SVG로딩 애니메이션, 페이지 디버깅

#### 2022-12-12 디버깅, SVG로딩 애니메이션 준비

#### 2022-12-08 부드러운 감속 비디오 적용(requestAnimationFrame)

#### 2022-12-07 이미지 블렌딩 후 캔버스 스크롤로 변경과 크기변경, 캔버스 스크롤로 변경 후 나타나는 내용 효과 추가, CSS작업

#### 2022-12-06 이미지 블렌딩 추가

#### 2022-12-05 섹션2 -> 섹션3 갑작스러운 이미지 노출 수정, 스크롤에 따른 이미지 블렌딩 준비 (scroll-section-3)

#### 2022-12-01 화면 크기에 따른 캔버스 크기 작업 (scroll-section-3)

#### 2022-11-30 사진 추가, 스크롤에 따른 비디오 인터랙션 적용 (scroll-section-1,2)

#### 2022-11-29

#### 2022-11-28

#### 2022-11-27

#### 2022-11-26

#### 2022-11-25

#### 2022-11-23

#### 2022-11-22

#### 2022-11-21 네비게이션 바 position absolute 고정, 섹션 구분 수정, 바운스 방지

#### 2022-11-20 전체 스크롤 높이중 현재 씬의 높이 계산

#### 2022-11-18 스크롤 이벤트 추가

#### 2022-11-17 HTML, CSS, default CSS 추가, 각 스크롤 섹션의 높이 세팅
