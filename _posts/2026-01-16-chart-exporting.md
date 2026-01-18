---
title: 보고서 규격에 맞는 차트와 글꼴 크기 설정하는 법
author: Rvinci
description: 내가 만든 차트는 왜 보고서에 넣으면 글씨가 작고 깨질까 고민한 적이 있나요? 문서 레이아웃에 맞춘 크기 설정, 고해상도 인쇄를 위한 dpi 조절 등 R을 활용해 고품질 차트를 생성하는 방법을 소개합니다.
excerpt: 내가 만든 차트는 왜 보고서에 넣으면 글씨가 작고 깨질까 고민한 적이 있나요? 문서 레이아웃에 맞춘 크기 설정, 고해상도 인쇄를 위한 dpi 조절 등 R을 활용해 고품질 차트를 생성하는 방법을 소개합니다.
categories: [chart]
tags: [r, tidyverse, ggplot2, ggsave]
published: true
---

## 공들여 만든 내 차트, 왜 저장만 하면 깨지고 흐릿할까?

보고서나 발표 자료를 만들 때 가장 공을 들이는 부분 중 하나가 바로
차트입니다. 하지만 엑셀에서 차트를 복사해서 붙여넣거나 화면을 캡처해
사용하다 보면, 문서 안에서 글씨가 너무 작아 보이거나 해상도가 낮아져
이미지가 깨지는 경험을 한 번쯤 해보셨을 겁니다.

R의 `ggplot2` 패키지는 그래프를 그리는 데 그치지 않고, 우리가 작성하는
문서의 규격에 딱 맞춰 저장하는 `ggsave`라는 강력한 도구를 제공합니다.
이번 글에서는 보고서의 가독성을 극대화할 수 있는 글꼴 크기 설정법부터,
인쇄물에서도 선명함을 유지하는 고해상도 저장 방법까지 다뤄보겠습니다.

## 글꼴 크기 설정하기

전문적인 보고서의 특징은 본문 텍스트와 차트 내 텍스트의 크기가
조화롭다는 점입니다. 예를 들어 보고서 본문이 11pt라면, 차트 내부는
이보다 살짝 작은 10pt 정도로 설정하는 것이 시각적으로 안정적입니다.

R에서는 `showtext` 패키지를 통해 시스템 글꼴을 불러오고,
`theme_minimal()` 등의 함수 내 `base_size` 옵션으로 전체적인 글꼴 크기를
조절할 수 있습니다. 이때 주의할 점은 R의 기본 단위와 실제 출력 단위(mm)
사이의 차이를 보정해주는 작업입니다.

``` r
# 패키지 로드
library(tidyverse)
library(scales)
library(showtext)

# 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/Kopub dotum medium.ttf")
showtext_auto() # showtext 자동 활성화
showtext_opts(dpi=300) # dpi 300 설정

# 글꼴 크기 설정
theme.size = 10 # pt 단위
text.size = theme.size / .pt # mm 단위
```

## 그래프 그리기

글꼴 설정이 끝났다면 이제 저장할 그래프를 그려야 합니다. 여기서는
1995년부터 2023년까지 서울과 경기도의 주민등록인구 변화 추이를 선
그래프로 나타내 보겠습니다. 나중에 파일로 저장하기 위해 완성된 그래프
객체를 `p`라는 변수에 저장하겠습니다.

``` r
p <- data2 %>% 
  ggplot(aes(x = 시점, y = 주민등록인구수, 
             group = 시도, color = 시도,
             label = comma(라벨, accuracy = 1))) +
  geom_line(linewidth = 0.4) +
  geom_point(size = 3) +
  geom_text(vjust = data2$vjust,
            family = "kopub",
            size = text.size,
            color = "gray10") +
  scale_x_discrete(name = "",
                   expand = expansion(mult = 0.05)) +
  scale_y_continuous(name = "주민등록인구 수(만 명)",
                     expand = expansion(mult = 0.2),
                     labels = comma_format()) +
  scale_color_brewer(palette = "Dark2") +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.line = element_line(linewidth = 0.5, color = "gray10"),
    axis.ticks = element_line(linewidth = 0.1, color = "gray10"),
    
    panel.grid.minor = element_blank(),
    
    axis.text = element_text(color = "gray10"),
    axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1),
    
    legend.title = element_blank(),
    legend.position = "bottom"
    )

# 그래프 확인
p
```

<img src="{{ site.baseurl }}/assets/images/2026-01-16-chart-exporting/unnamed-chunk-2-1.png" alt="" width="100%" style="display: block; margin: auto;" />

## `ggsave`로 그림 파일 저장하기

이제 가장 중요한 저장 단계입니다. `ggsave`는 파일 형식, 물리적 크기,
해상도(DPI)를 직접 지정할 수 있습니다.

- `filename`: 파일 경로, 이름, 확장자를 지정합니다
- `plot`: 저장할 그래프 객체를 지정합니다
- `width`, `height`: 그림파일 크기를 설정하며, 단위는 `unit`으로
  지정합니다
- `dpi`: 해상도를 설정하며, 300 정도면 고품질 출력에 적합합니다

아래 코드는 한글 문서(A4)의 일반적인 레이아웃에 맞춰 가로 145mm, 세로
70mm 크기로 차트를 저장하는 예시입니다. 해상도를 300dpi로 설정하면 실제
보고서를 인쇄했을 때 선명하게 출력됩니다.

``` r
ggsave(
  filename = "아웃풋/1995~2023년 서울, 경기 주민등록인구 추이.jpeg",
  plot = p,
  width = 145,
  height = 70,
  unit = "mm",
  dpi = 300
)
```

차트를 잘 그리는 것만큼이나 중요한 것이 바로 ’어떻게 보여주느냐’입니다.
`ggsave`를 활용해 차트의 크기와 해상도를 조정하면, 보고서나 발표자료
안에서 차트가 훨씬 더 신뢰감 있게 전달될 것입니다. 오늘 배운 팁을 활용해
여러분의 문서 레이아웃에 딱 맞는, 깨지지 않는 차트를 직접 만들어 보시길
바랍니다!