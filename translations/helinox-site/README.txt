Helinox 브랜드 사이트

outputs/design/Helinox_웹디자인_기획안.txt (컨셉: "Featherweight Craft")를
기반으로 제작한 Helinox 브랜드소개 정적 웹사이트입니다.

페이지 구성
- index.html — 단일 스크롤 메인 페이지 (히어로, 헤리티지, 기술·소재,
  브랜드 무드, 협업, CTA)
- heritage.html — 브랜드 스토리/연혁 전체 보기 (Depth 2)
- collaborations.html — 협업 아카이브 전체 보기 (Depth 2)

폴더 구조
helinox-site/
├── index.html
├── heritage.html
├── collaborations.html
├── css/
│   └── style.css
├── js/
│   └── main.js
└── README.md

기술 스택
순수 HTML/CSS/JS로 제작했으며 별도 빌드 단계나 프레임워크는 사용하지
않습니다. 폰트(Inter, Noto Sans KR)는 Google Fonts CDN에서 불러옵니다.
모든 이미지는 SVG 라인아트, CSS 그라디언트, 컬러 블록 등 플레이스홀더로
대체했습니다 — 기획안에 실제 촬영 이미지 자산은 포함되어 있지 않습니다.

로컬 실행 방법
index.html을 브라우저에서 바로 열거나, 아래와 같이 정적 파일 서버로
실행할 수 있습니다.

  npx serve helinox-site

참고 사항
- 컬러 팔레트, 타이포그래피, 섹션 구조는 기획안의 컬러 HEX 값과 정보
  구조도(IA)를 그대로 반영했습니다.
- js/main.js에 구현된 인터랙션: 모바일 GNB 토글, 스크롤 기반 GNB
  링크 active 표시(IntersectionObserver), 헤리티지 타임라인 항목
  포커스/클릭 강조, 모바일 스와이프 캐러셀용 협업 카드 포커스 강조.
