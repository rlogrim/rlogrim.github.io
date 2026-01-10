---
title: 원형차트와 도넛차트 그리기
author: Rvinci
description: 합이 100%가 되는 비율 데이터를 원형차트나 도넛차트로 보여주고 싶은데 어떻게 그려야 할지 막막하신가요? 누적막대그래프를 변형하여 원형차트로 바꾸는 기본 원리부터, 도넛차트를 만드는 방법까지 알려드립니다.
excerpt: 합이 100%가 되는 비율 데이터를 원형차트나 도넛차트로 보여주고 싶은데 어떻게 그려야 할지 막막하신가요? 누적막대그래프를 변형하여 원형차트로 바꾸는 기본 원리부터, 도넛차트를 만드는 방법까지 알려드립니다.
categories: [chart]
tags: [r, tidyverse, pie chart, donut chart, geom_col, coord_polar("y")]
published: true
---

비율의 합이 100%인 데이터를 보여줄 때 원형차트(파이차트)와 도넛차트가 자주
쓰입니다. `ggplot2`에는 원형차트 전용 함수가 없어서, 먼저
’누적막대그래프’를 만든 다음 `coord_polar()`로 좌표계를 원형으로 바꿔
그립니다. 도넛차트는 같은 원리를 그대로 쓰되, 막대가 시작하는 x 위치를
바깥쪽으로 밀어 가운데가 비어 보이게 만들 수 있습니다.

여기서는 2023년 시도별 시, 군, 구 주민등록인구 비중 데이터를 예로
들지만, 범주별 비율 데이터라면 같은 코드를 그대로 적용할 수 있습니다.

## 데이터 준비하기

원형차트와 도넛차트는 범주와 비율만 있으면 그릴 수 있습니다. 아래 예시는
[비율 계산하고 보고서용 표
만들기](https://rlogrim.github.io/data/from-counts-to-shares/) 글에서
만든 데이터를 사용합니다.

먼저 패키지를 불러오고, 그래프에 사용할 폰트를 설정한 뒤 데이터를
확인합니다. 폰트 경로는 운영체제마다 다를 수 있으니, 본인 환경에 맞게
바꿔 사용하시면 됩니다.

``` r
# 패키지 로드
library(tidyverse)
library(showtext)
library(scales)

# 글꼴 설정
font_add("kopub", "C:/Users/.../appdata/local/microsoft/windows/fonts/kopub dotum medium.ttf")
showtext_auto()
showtext_opts(dpi=300)

theme.size = 12
text.size = theme.size / .pt

# 데이터 확인하기
head(share)
```

    ## # A tibble: 6 × 4
    ## # Groups:   시도 [4]
    ##   시도       구분  주민등록인구수   비율
    ##   <fct>      <fct>          <dbl>  <dbl>
    ## 1 서울특별시 구           9386034 1     
    ## 2 부산광역시 군            178729 0.0543
    ## 3 부산광역시 구           3114633 0.946 
    ## 4 대구광역시 군            285072 0.120 
    ## 5 대구광역시 구           2089888 0.880 
    ## 6 인천광역시 군             89382 0.0298

## 원형차트(파이차트) 그리기

부산, 울산, 경남 각각의 시, 군, 구 인구 비중을 원형차트로
그려보겠습니다.

핵심은 두 가지입니다. 먼저 `geom_col()`로 누적막대그래프를 만든 뒤,
`coord_polar("y")`로 좌표계를 원형으로 바꿉니다. 이때 `x = 1`을 주면
모든 조각이 ‘하나의 막대’ 안에 쌓이게 됩니다. 그리고 `y`에 비율을
넣으면, 막대의 높이(비율)가 원의 각도로 바뀌어 원형차트가 됩니다.

조각 안에 퍼센트 라벨을 넣을 때는 누적막대그래프와 같은 방식으로
`position_stack(vjust = 0.5)`를 쓰면 각 조각의 가운데에 텍스트가
놓입니다.

``` r
share %>% 
  filter(시도 %in% c("부산광역시", "울산광역시", "경상남도")) %>% 
  ggplot(aes(x = 1, y = 비율, 
             fill = 구분,
             label = percent(비율, accuracy = .1))) +
    geom_col(width = 1, 
             linewidth = 0.4, 
             color = "white") +
    geom_text(family = "kopub",
              position = position_stack(vjust = 0.5),
              size = text.size) +
    scale_fill_brewer(palette = "Pastel2") +
    coord_polar("y") +
    facet_wrap(~시도, nrow = 1) +
    theme_void(base_size = theme.size, base_family = "kopub")
```

<img src="{{ site.baseurl }}/assets/images/2026-01-10-pie-and-donut-charts/unnamed-chunk-2-1.png" alt="" width="100%" style="display: block; margin: auto;" alt="원형차트" />

## 도넛차트 그리기

도넛차트는 원형차트와 같은 코드에서 시작합니다. 차이는 `x` 값입니다.
`x = 1` 대신 더 큰 값(예: `x = 2.5`)을 주면 막대가 원형의 바깥쪽
배치되어 가운데가 비어 보입니다. 이 빈 공간이 도넛차트의 ’구멍’입니다.

가운데 빈 공간에 시도명을 넣을 수도 있습니다. `geom_text()`에서
`x = 0`을 따로 지정하면, 폴라 좌표에서도 중앙에 텍스트를 배치할 수
있습니다. 대신 `facet_wrap()`이 기본으로 붙이는 시도명(스트립 텍스트)은
중앙 텍스트와 역할이 겹치므로 `theme()`에서 숨깁니다.

``` r
share %>% 
  filter(시도 %in% c("부산광역시", "울산광역시", "경상남도")) %>% 
  ggplot(aes(x = 2.5, 
             y = 비율, 
             fill = 구분)) +
    geom_col(width = 1, 
             linewidth = 0.4, 
             color = "white") +
    geom_text(aes(label = percent(비율, accuracy = .1)),
              family = "kopub",
              position = position_stack(vjust = 0.5),
              size = text.size) +
    geom_text(aes(x = 0, label = 시도),
              size = text.size,
              fontface = "bold") +
    scale_fill_brewer(palette = "Pastel2") +
    coord_polar("y") +
    facet_wrap(~시도, nrow = 1) +
    theme_void(base_size = theme.size, base_family = "kopub") +
    theme(
      strip.text = element_blank(),
      strip.background = element_blank()
    )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-10-pie-and-donut-charts/unnamed-chunk-3-1.png" alt="" width="100%" style="display: block; margin: auto;" alt="도넛차트" />

## 막대그래프, 원형차트, 도넛차트는 언제 쓰면 좋을까?

비율의 합이 100%인 데이터를 보여줄 때는 막대그래프, 원형차트, 도넛차트
중에서 고를 수 있습니다. 어떤 그래프가 더 좋은지는 ’독자가 무엇을 가장
쉽게 알아채길 원하는지’에 따라 달라집니다.

막대그래프는 막대의 ’길이’로 비율을 비교합니다. 사람은 각도나 넓이보다
길이를 비교할 때 더 정확하게 차이를 느낍니다. 그래서 항목들 사이에 ’누가
더 크고, 얼마나 더 큰지’를 보여주고 싶다면 막대그래프가 적합합니다.

원형차트는 ’전체가 100%이고, 그 안에서 어떻게 나뉘는지’를 직관적으로
보여줍니다. 다만, 조각의 각도를 눈으로 비교해야 해서, 비율이 비슷한
항목이 여러 개 있으면 차이가 잘 안 드러날 수 있습니다.

도넛차트는 원형차트처럼 조각의 ’각도’로 비율을 보여주면서도, 조각이
차지하는 ’둘레 길이’도 함께 눈에 들어옵니다. 그래서 각도만으로 판단해야
하는 원형차트보다 값 차이를 비교할 때 조금 더 수월한 편입니다. 그래도
막대그래프처럼 같은 기준선에서 길이를 재는 방식은 아니라서, 비율이
비슷한 항목이 많다면 막대그래프를 사용하는 것이 더 좋을 수 있습니다.