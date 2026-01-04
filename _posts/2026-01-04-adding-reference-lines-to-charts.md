---
title: 기준선 추가해 데이터 비교하기
author: Rvinci
description: 그래프에 평균이나 기준값을 함께 보여주고 싶었던 적 있나요? 기준선을 추가하는 방법을 통해 값의 위치를 한눈에 파악하고 해석이 쉬운 차트를 만드는 과정을 알려드립니다.
excerpt: 그래프에 평균이나 기준값을 함께 보여주고 싶었던 적 있나요? 기준선을 추가하는 방법을 통해 값의 위치를 한눈에 파악하고 해석이 쉬운 차트를 만드는 과정을 알려드립니다.
categories: [chart]
tags: [r, tidyverse, ggplot2, geom_col, geom_hline, geom_vline]
published: true
---

값의 크기를 비교할 때 막대그래프를 자주 사용합니다. 한눈에 크고 작은
값을 확인할 수 있기 때문입니다. 다만 막대만 놓고 보면 이 값이 평균보다
높은지 낮은지는 바로 보이지 않는 경우가 많습니다. 막대의 상대적인 크기는
알 수 있어도, 어디를 기준으로 봐야 하는지는 판단하기 어렵습니다.

이럴 때 기준선을 함께 그리면 데이터를 훨씬 쉽게 이해할 수 있습니다.
기준선은 하나의 비교 기준이 되어 주고, 각 값이 그 기준보다 위에 있는지
아래에 있는지를 바로 보여줍니다.

이번 글에서는 서울특별시 25개 구의 주민등록인구를 예로 들어 막대그래프에
평균선을 추가하는 방법을 살펴봅니다. 이 방법은 평균선뿐 아니라
목표값이나 기준값을 표시할 때도 그대로 활용할 수 있습니다.

## 패키지와 글꼴 설정하기

먼저 데이터 처리와 시각화를 위해 필요한 패키지를 불러옵니다.
`tidyverse`에는 `dplyr`와 `ggplot2`가 포함되어 있어 데이터 가공과
시각화를 대부분 처리할 수 있습니다. 여기에 엑셀 파일을 읽기 위한
`readxl`, 한글 글꼴을 사용하기 위한 `showtext`, 숫자 표기를 돕는
`scales` 패키지를 함께 사용합니다.

``` r
library(readxl)
library(tidyverse)
library(showtext)
library(scales)
```

특정한 한글 폰트를 사용하기 위해 글꼴도 함께 설정합니다. `font_add()`로
사용할 글꼴을 등록하고, `showtext_auto()`를 사용해 그래프 출력 시 해당
글꼴이 적용되도록 합니다. 이후 텍스트 크기를 계산해 차트 전반에 일관되게
사용합니다.

``` r
font_add("kopub", "C:/Windows/Fonts/KoPubDotumMedium.ttf")

showtext_auto()
showtext_opts(dpi=300)

theme.size = 12
text.size = theme.size / .pt
```

## 데이터 준비하기

이번 예제에서는 통계청에서 제공하는 [시군구별 주민등록인구
데이터](https://kosis.kr/statHtml/statHtml.do?orgId=101&tblId=DT_1YL20651E&vw_cd=MT_GTITLE01&list_id=101&scrId=&seqNo=&lang_mode=ko&obj_var_id=&itm_id=&conn_path=MT_GTITLE01&path=%252FstatisticsList%252FstatisticsListIndex.do)를
사용합니다. 엑셀 파일을 그대로 불러와 이후 단계에서 필요한 형태로
가공합니다.

``` r
data <- read_xlsx("데이터/주민등록인구_시도_시군구_2023.xlsx", skip=1)
```

이제 서울특별시 데이터만 남기고 차트에 사용할 변수를 정리합니다. 변수
이름을 알아보기 쉽게 바꾸고, 서울특별시가 아닌 행은 제거합니다. 이후
인구 수를 기준으로 순위를 계산해 상위 구만 다른 색으로 표시할 수 있도록
색상 변수를 추가합니다.

``` r
seoul_clr <- data %>% 
  rename(시도 = `행정구역별(1)`,
         구 = `행정구역별(2)`,
         인구수 = `계 (명)`) %>% 
  # 서울 데이터 추출
  filter(시도 == "서울특별시",
         구 != "소계") %>% 
  # 색상 칼럼 생성
  mutate(순위 = rank(-인구수),
    색상 = case_when(순위 < 4 ~ "#027453",
                   T ~ "#D9D8D6"))

# 데이터 확인
head(seoul_clr)
```

    ## # A tibble: 6 × 5
    ##   시도       구       인구수  순위 색상   
    ##   <chr>      <chr>     <dbl> <dbl> <chr>  
    ## 1 서울특별시 종로구   139417    24 #D9D8D6
    ## 2 서울특별시 중구     121312    25 #D9D8D6
    ## 3 서울특별시 용산구   213151    23 #D9D8D6
    ## 4 서울특별시 성동구   277361    21 #D9D8D6
    ## 5 서울특별시 광진구   335554    17 #D9D8D6
    ## 6 서울특별시 동대문구 341149    16 #D9D8D6

## 기본 막대그래프 그리기

이제 가공한 데이터를 이용해 막대그래프를 그려봅니다. 인구 수는 값이 너무
크지 않도록 만 명 단위로 나누어 표시합니다. `reorder()`를 사용해 인구가
많은 구가 왼쪽에 오도록 정렬하고, `geom_col()`로 막대를 그립니다. 막대
위에는 `geom_text()`를 사용해 실제 값을 함께 표시합니다.

``` r
seoul_clr %>% 
  ggplot(
    aes(
      x = reorder(구, -인구수),
      y = 인구수 / 10000,
      fill = 색상
    )
  ) +
  geom_col(
    width = 0.6,
    color = "gray10",
    linewidth = 0.2
  ) +
  geom_text(
    aes(label = comma(인구수 / 10000, accuracy = 1)),
    family = "kopub",
    size = text.size,
    vjust = -1
  ) +
  scale_x_discrete(name = "") +
  scale_y_continuous(
    name = "주민등록인구 수(만 명)",
    expand = expansion(mult = c(0, 0.1))
  ) +
  scale_fill_identity() +
  theme_minimal(
    base_size = theme.size,
    base_family = "kopub"
  ) +
  theme(
    axis.line.x = element_line(linewidth = 0.8),
    panel.grid.major.x = element_blank(),
    panel.grid.minor = element_blank(),
    axis.text.x = element_text(angle = 45, hjust = 1)
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-04-adding-reference-lines-to-charts/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" alt="서울시 구별 주민등록인구 수 그래프" />

이 상태에서도 각 구의 인구 규모 차이는 충분히 확인할 수 있습니다. 다만
평균을 기준으로 보면 어떤 구가 평균보다 높은지, 낮은지는 바로 파악하기
어렵습니다.

## 평균선 추가하기

먼저 기준이 될 평균 인구 수를 계산합니다. 앞에서 그래프를 만 명 단위로
그렸기 때문에 평균값도 같은 단위로 맞춰 계산합니다.

``` r
mean_value <- mean(seoul_clr$인구수) / 10000
```

이제 `geom_hline()`을 사용해 평균선을 추가합니다. `yintercept`에는
기준이 될 평균값을 넣고, 점선으로 표시해 막대와 구분되도록 합니다.
이어서 `annotate()`를 사용해 평균선이 무엇을 의미하는지 텍스트로 함께
표시합니다.

``` r
seoul_clr %>% 
  ggplot(
    aes(
      x = reorder(구, -인구수),
      y = 인구수 / 10000,
      fill = 색상
    )
  ) +
  geom_col(width = 0.6, color = "gray10", linewidth = 0.2) +
  geom_text(
    aes(label = comma(인구수 / 10000, accuracy = 1)),
    family = "kopub",
    size = text.size,
    vjust = -1
  ) +
  geom_hline(
    yintercept = mean_value,
    linetype = "dashed",
    linewidth = 0.7
  ) +
  annotate(
    "text",
    x = length(unique(seoul_clr$구)),
    y = mean_value,
    label = paste0("평균: ", comma(mean_value, accuracy = 1)),
    hjust = 1,
    vjust = -1,
    family = "kopub",
    size = text.size
  ) +
  scale_x_discrete(name = "") +
  scale_y_continuous(
    name = "주민등록인구 수(만 명)",
    expand = expansion(mult = c(0, 0.1))
  ) +
  scale_fill_identity() +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.line.x = element_line(linewidth = 0.8),
    panel.grid.major.x = element_blank(),
    panel.grid.minor = element_blank(),
    axis.text.x = element_text(angle = 45, hjust = 1)
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-04-adding-reference-lines-to-charts/unnamed-chunk-7-1.png" width="100%" style="display: block; margin: auto;" alt="평균선 추가한 막대그래프" />

평균선을 추가하면 각 구의 인구가 평균보다 많은지 적은지를 바로 확인할 수
있습니다. 막대를 하나씩 비교하지 않아도 전체 분포와 위치 관계가
자연스럽게 눈에 들어옵니다. 이 방법은 평균선뿐 아니라 목표값이나 정책
기준선을 표시할 때도 그대로 사용할 수 있습니다. 가로 기준선은
`geom_hline()`, 세로 기준선은 `geom_vline()`을 사용하면 됩니다. 기준선을
적절히 활용하면 막대그래프를 훨씬 직관적으로 읽을 수 있어 해석이
쉬워집니다.