---
title: 범례 위치 조정하기
author: Rvinci
description: 범례 위치를 조정하고 싶었던 적이 있나요? 이번 글에서는 theme()으로 범례를 오른쪽에서 아래로 옮기고, 패널 안쪽에 배치하는 방법을 알려드립니다. 범례 키 크기, 간격, 배경 등을 바꾸는 방법도 다룹니다.
excerpt: 범례 위치를 조정하고 싶었던 적이 있나요? 이번 글에서는 theme()으로 범례를 오른쪽에서 아래로 옮기고, 패널 안쪽에 배치하는 방법을 알려드립니다. 범례 키 크기, 간격, 배경 등을 바꾸는 방법도 다룹니다.
categories: [chart]
tags: [r, tidyverse, legends, theme]
published: true
---

범례는 차트를 읽을 때 필요한 ’설명서’입니다. 그런데 범례가 애매한 곳에
있으면 데이터보다 범례부터 눈에 들어오거나, 반대로 범례를 찾느라 시선이
자꾸 왔다 갔다 합니다. 위치를 조금만 바꿔도 정돈된 인상을 줄 수
있습니다.

`ggplot2` 그래프는 `aes()`로 매핑을 정하고 geom 레이어를 쌓는 방식으로
만들어집니다. 범례는 보통 `color`나 `fill` 같은 매핑에서 자동으로
생성됩니다. 겉보기에는 같은 범례처럼 보여도, `fill`과 `color`는 서로
다른 범례로 만들어질 수 있으니 먼저 어떤 aesthetic에 연결되어 있는지
확인하는 게 좋습니다.

`ggplot2`에서는 `theme()`가 차트의 비데이터 요소를 관리합니다. 범례도
`theme()` 아래에서 `legend.title`, `legend.text`, `legend.key`처럼 여러
구성 요소로 나뉘어 조정됩니다. ’범례를 어디에 둘지’와 ’범례를 어떻게
보이게 할지’를 나눠서 생각하면 좋습니다.

이번 글에서는 범례를 패널 바깥 오른쪽과 아래쪽으로 옮기는 기본 방법을
먼저 정리하고, `ggplot2` 3.5.0부터 지원하는 `inside` 배치로 패널 안쪽에
안정적으로 넣는 방법까지 이어서 살펴보겠습니다. 예시로 [이전
글](https://rlogrim.github.io/chart/creating-a-bar-and-line-combo-chart/)에서
만든 혼합형 그래프를 사용하겠습니다. 먼저 기본 그래프를 그려줍니다.

``` r
base <- ggplot(data = data2) +
  geom_col(aes(x = 년도, 
               y = `승객 수`, 
               fill = "승객 수"), 
           width = 0.6) +
  geom_line(aes(x = 년도, 
                y = `전년 대비 승객 수 증감률` * coeff, 
                color = "전년 대비 승객 수 증감률")) +
  scale_fill_manual(values = "gray40") +
  scale_color_manual(values = "gray70") +
  scale_x_continuous(n.breaks = 12) +
  scale_y_continuous(
    name = "승객 수(천명)",
    labels = comma_format(),
    expand = expansion(mult=c(0, 0.3)),
    sec.axis = sec_axis(~./coeff, 
                        name = "전년 대비 승객 수 증감률(%)",
                        labels = percent_format())
  ) +
  theme_bw(base_family="kopub", base_size=theme.size)

base
```

<img src="{{ site.baseurl }}/assets/images/2026-01-11-positioning-legends/unnamed-chunk-1-1.png" width="80%" style="display: block; margin: auto;" alt="기본 범례 그래프" />

## 오른쪽 범례를 기본값으로 두고, 보기 좋게만 다듬기

범례는 기본적으로 패널 바깥의 오른쪽에 놓입니다. 범례 제목을 없애고,
아이콘(키) 높이를 글자 높이와 맞춰 주면 좀 더 깔끔해집니다. 범례는
내부적으로 여러 요소로 나뉘기 때문에, `theme()`에서 필요한 부분만 골라
손보면 됩니다.

이 위치는 무난하지만, 범례가 길어지면 패널의 가로 폭이 줄어서 그래프가
답답해 보일 때가 있습니다.

``` r
base +
  theme(
    legend.title = element_blank(),
    legend.key.height = unit(0.9, "lines"),
    legend.spacing = unit(3, "pt"),
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-11-positioning-legends/unnamed-chunk-2-1.png" width="80%" style="display: block; margin: auto;" alt="기본 범례 다듬은 그래프" />

## 범례를 하단에 배치하기

가로로 긴 그래프의 범례는 하단에 배치하는 것이 잘 맞는 경우가 많습니다.
범례를 아래로 옮길 때는 `legend.position = "bottom"`을 지정하면 됩니다.

``` r
base +
  theme(
    legend.title = element_blank(),
    legend.key.height = unit(0.9, "lines"),
    legend.spacing = unit(3, "pt"),
    legend.position = "bottom"
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-11-positioning-legends/unnamed-chunk-3-1.png" width="80%" style="display: block; margin: auto;" alt="범례를 하단에 배치한 그래프" />

## 범례를 패널 안쪽에 배치하기

패널 안쪽 범례는 시선 이동을 줄이고 싶을 때 유용합니다. 특히 범례가
가까이에 있으면 “이 색이 뭘 의미하지?”를 바로 확인할 수 있습니다.

다만 데이터 위를 가리면 역효과가 날 수 있습니다. 먼저 데이터가 덜 몰려
있는 공간을 찾고, 그쪽에 범례를 놓는 게 안전합니다. 우상향하는
그래프라면 왼쪽 위가 비어 있는 경우가 많아서, 그 위치를 후보로 두고
시작하면 편합니다.

`ggplot2` 3.5.0부터는 범례를 패널 내부로 넣을 때
`legend.position = "inside"`를 쓰고, 실제 위치는
`legend.position.inside`로 따로 지정할 수 있습니다. 예전처럼 좌표를
`legend.position`에 직접 넣는 방식보다 구조가 분명해서, 화면 크기가 바뀔
때도 의도한 위치가 유지되는 편입니다.

`legend.justification`은 ’범례 박스의 어느 점을 기준으로 붙일지’를
정합니다. `c(0, 1)`은 범례 박스의 왼쪽 위를 기준으로 삼겠다는 뜻입니다.

``` r
base +
  theme(
    legend.title = element_blank(),
    legend.key.height = unit(0.9, "lines"),
    legend.spacing = unit(3, "pt"),
    legend.position = "inside",
    legend.position.inside = c(0.05, 0.95),
    legend.justification = c(0, 1),
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-11-positioning-legends/unnamed-chunk-4-1.png" width="80%" style="display: block; margin: auto;" alt="범례를 패널 안쪽에 배치한 그래프" />

## 범례 배경색 없애기

범례를 패널 안에 넣으면, 범례의 흰 배경이 눈에 띄는 경우가 있습니다.
`legend.background = element_rect(fill=NA)`를 추가하여 범례 배경을
투명하게 만들 수 있습니다.

``` r
base +
  theme(
    legend.title = element_blank(),
    legend.key.height = unit(0.9, "lines"),
    legend.spacing = unit(3, "pt"),
    legend.position = "inside",
    legend.position.inside = c(0.05, 0.95),
    legend.justification = c(0, 1),
    legend.background = element_rect(fill = NA),
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-11-positioning-legends/unnamed-chunk-5-1.png" width="80%" style="display: block; margin: auto;" alt="범례 배경색을 없앤 그래프" />

범례 위치는 ’예쁘게 보이게 하는 선택’이라기보다, 독자가 그래프를 어떻게
읽게 될지를 정하는 설정에 가깝습니다. 오른쪽은 익숙해서 무난하지만
범례가 길어지면 패널이 좁아질 수 있습니다. 아래쪽은 가로 공간을 넓게 쓸
수 있어서 시계열처럼 흐름을 따라 읽는 그래프에 잘 맞습니다.
패널 안쪽에 넣는 방식은 범례를 바로 옆에서 확인할 수 있다는 장점이
있지만, 데이터가 겹치는 영역을 피해서 배치하는 게 중요합니다.

범례 위치를 옮겼다면 `legend.key.height`, `legend.spacing`,
`legend.background` 같은 요소도 같이 조정해 보시면 좋습니다. 작은
차이인데도 범례가 훨씬 정돈돼 보입니다. 아래 범례 관련 theme 요소를 참고해서 조정해
보세요!

<img src="{{ site.baseurl }}/assets/images/2026-01-11-positioning-legends/spacing_overview-1.png" width="80%" style="display: block; margin: auto;" alt="범례 관련 theme 요소">
<figcaption style="display:block; font-size:0.9em; color:#555;">
    출처: Tidyverse 블로그(ggplot2 3.5.0: Legends)
</figcaption>