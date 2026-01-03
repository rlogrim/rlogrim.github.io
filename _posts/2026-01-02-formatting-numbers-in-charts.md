---
title: 콤마, 백분율, 소수점 표기하기
author: Rvinci
description: 차트에 표시된 숫자가 일관성 없이 뒤섞여 있어 한눈에 파악하기 어려웠던 적이 있지 않나요? 이 글에서는 scales 패키지로 천 단위마다 콤마를 추가하고, 비율을 백분율로 전환하며, 소수점 자릿수를 조정하는 방법을 알려드립니다. 이중 축 그래프까지 다뤄 서로 다른 단위의 데이터를 하나의 차트에 표현하는 실전 기법도 소개합니다.
excerpt: 차트에 표시된 숫자가 일관성 없이 뒤섞여 있어 한눈에 파악하기 어려웠던 적이 있지 않나요? 이 글에서는 scales 패키지로 천 단위마다 콤마를 추가하고, 비율을 백분율로 전환하며, 소수점 자릿수를 조정하는 방법을 알려드립니다. 이중 축 그래프까지 다뤄 서로 다른 단위의 데이터를 하나의 차트에 표현하는 실전 기법도 소개합니다.
categories: [chart]
tags: [r, tidyverse, ggplot2, scales, comma, percent, sec.axis]
published: true
---

데이터 시각화에서 숫자를 어떻게 표현하느냐에 따라 그래프의 가독성이 달라집니다. '1234567'보다는 '1,234,567'이, '0.123'보다는 '12.3%'가 훨씬 읽기 쉽죠. 마치 책을 읽을 때 적절한 띄어쓰기와 문장부호가 있어야 편하게 읽히는 것처럼, 숫자에도 적절한 포맷이 필요합니다.

이번 글에서는 `scales` 패키지를 활용해 그래프 상의 숫자를 전문적으로 표기하는 방법을 알아보겠습니다. 항공 승객 데이터(`AirPassengers`)를 사용해 콤마 표기, 백분율 변환, 소수점 조정부터 복합 그래프 그리기까지 단계별로 살펴볼게요.

## 데이터 준비: 연도별 항공 승객 수 집계

먼저 `AirPassengers` 데이터를 활용해 연도별 총 승객 수와 전년 대비 증감률을 계산하겠습니다. 월별 데이터를 연도별로 합산하고, 연간 증감률을 계산합니다.

`matrix` 함수로 시계열 데이터를 12개월씩 행으로 나눈 뒤, `rowSums`로 연도별 합계를 구합니다. `lag` 함수는 전년도 값을 가져와 증감률을 계산하는 데 사용되며, 첫 해(1949년)는 비교 대상이 없어 `NA` 값이 생성됩니다.

``` r
# 패키지 로드
library(tidyverse)

# 데이터 준비
data <- data.frame(matrix(AirPassengers, ncol = 12, byrow = TRUE))

colnames(data) <- seq(1, 12)

data <- data %>% mutate(년도 = seq(1949, 1960))

data2 <- data %>% 
  mutate(
    `승객 수` = rowSums(across(1:12)),
    `전년 대비 승객 수 증감률` = (`승객 수` - lag(`승객 수`)) / lag(`승객 수`)
  ) %>% 
  select(년도, `승객 수`, `전년 대비 승객 수 증감률`)
```

## 콤마 표기로 큰 숫자 읽기 쉽게 만들기

큰 숫자를 표시할 때는 세 자리마다 콤마를 추가하면 가독성이 향상됩니다. '129000'보다 '129,000'이 훨씬 직관적이죠. `scales` 패키지의 `comma` 함수가 이를 자동으로 처리해줍니다. `comma` 함수를 `geom_text`의 `label`에 적용하면 숫자가 자동으로 천 단위마다 콤마로 구분됩니다.

``` r
# 패키지 로드
library(scales)

# 그래프 작성
ggplot(data = data2,
       aes(x = 년도, y = `승객 수`, 
           label = comma(`승객 수`))
       ) +
  geom_col(width = 0.6) +
  geom_text(vjust = -1) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-formatting-numbers-in-charts/unnamed-chunk-2-1.png" width="70%" style="display: block; margin: auto;" alt="콤마 포맷팅한 그래프" />

## 백분율 표기하고 소수점 자릿수 조정하기

비율이나 증감률 데이터는 백분율로 표현하는 것이 일반적입니다. 0.15보다는 15%가 훨씬 이해하기 쉽죠. `percent` 함수를 사용하면 소수를 백분율로 변환하고, `accuracy` 옵션으로 소수점 자릿수를 조정할 수 있습니다. `percent` 함수는 소수 값을 자동으로 100을 곱해 백분율로 변환하고 '%' 기호를 붙여줍니다. `accuracy = 0.1`은 소수점 첫째 자리까지 표시하라는 의미이며, `accuracy = 1`로 설정하면 정수로 반올림됩니다.

``` r
ggplot(data = data2,
       aes(x = 년도, y = `전년 대비 승객 수 증감률`,
           label = percent(`전년 대비 승객 수 증감률`, accuracy = 0.1))
       ) +
  geom_line(size = 1) +
  geom_text(vjust = -1) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-formatting-numbers-in-charts/unnamed-chunk-3-1.png" width="70%" style="display: block; margin: auto;" alt="백분율 포맷팅한 그래프" />

## 이중 축을 그려 완성도 높이기

실무에서는 하나의 그래프에 서로 다른 단위의 데이터를 함께 표시해야 할 때가 많습니다. 예를 들어 절대값(승객 수, 단위: 천 명)과 상대값(증감률, 단위: %)을 동시에 보여주면 전체적인 추세와 변화율을 한눈에 파악할 수 있죠.

문제는 이 두 데이터의 스케일이 완전히 다르다는 점입니다. 승객 수는 수백만 단위인 반면, 증감률은 3~22% 범위에 있습니다. 같은 y축을 사용하면 증감률 선이 바닥에 붙어버려 변화를 전혀 볼 수 없게 됩니다.

이중 축 그래프를 이용하면 이러한 문제를 해결할 수 있습니다. 왼쪽에는 승객 수 축(천 명 단위), 오른쪽에는 증감률 축(% 단위)을 별도로 배치해서 각 데이터가 최적의 스케일로 표시되도록 만듭니다. 이렇게 하면 막대그래프로 절대적인 규모를 보여주면서, 동시에 선 그래프로 성장률의 변화 패턴을 명확히 드러낼 수 있습니다.

아래 코드에서 `coeff` 변수는 두 데이터의 스케일을 맞추기 위한 변환 계수로, 증감률 값에 곱해져 막대그래프와 비슷한 높이로 조정됩니다. `sec.axis`는 오른쪽에 두 번째 y축을 추가하는 옵션이며, `~./coeff`로 원래 값으로 되돌려 표시합니다. `comma_format()`과 `percent_format(suffix = "")`은  
각 축의 레이블에 포맷을 적용하기 위한 함수입니다. `suffix = ""`는 % 기호를 축 제목에서 이미 표시했기 때문에, 중복을 피하기 위해서 숫자 뒤에 단위가 붙지 않도록 합니다.

``` r
# 패키지 로드
library(showtext)

# 글꼴 설정
font_add_google("Noto Sans KR", "noto")
showtext_auto()
showtext_opts(dpi = 300)

theme.size = 10
geom.text.size = theme.size/.pt 

# 혼합형 그래프 작성을 위한 상수 계산
coeff <- data2[2, "승객 수"] / data2[2, "전년 대비 승객 수 증감률"] * 0.8

# 그래프 작성
ggplot(data = data2, aes(x = 년도)) +
  geom_col(aes(y = `승객 수`, fill = "승객 수"), width = 0.6) +
  geom_text(aes(y = `승객 수`, label = comma(`승객 수`)), 
            vjust = -1, family = "noto", size = 3) +
  geom_line(aes(y = `전년 대비 승객 수 증감률` * coeff, color = "전년 대비 승객 수 증감률"), size = 1) +
  geom_point(aes(y = `전년 대비 승객 수 증감률` * coeff, color = "전년 대비 승객 수 증감률"), size = 2) +
  geom_text(aes(y = `전년 대비 승객 수 증감률` * coeff, 
                label = percent(`전년 대비 승객 수 증감률`, accuracy = 0.1, suffix = "")), 
            vjust = -1, family = "noto", size = 3) +
  scale_fill_manual(values = "#9AD1F5") +
  scale_color_manual(values = "#e25a6c") +
  scale_x_continuous(n.breaks = 12) +
  scale_y_continuous(
    name = "승객 수(천 명)",
    labels = comma_format(),
    expand = expansion(mult = c(0, 0.1)),
    sec.axis = sec_axis(~./coeff, name = "전년 대비 승객 수 증감률(%)", labels = percent_format(suffix = ""))
  ) +
  theme_bw(base_family = "noto") +
  theme(
    panel.grid = element_blank(),
    legend.title = element_blank(),
    legend.background=element_blank(),
    legend.key.size=unit(geom.text.size, "mm"),
    legend.text=element_text(size=rel(1)),
    legend.position="inside",
    legend.position.inside=c(0.20, 0.90),
    legend.margin=margin(0, 0, 0, 0)
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-02-formatting-numbers-in-charts/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" alt="이중 축을 그린 그래프" />

보고서에서는 숫자 표기 방법을 통일하고 소수점 자릿수를 일관되게 유지하는 것이 중요합니다. 모든 값을 소수점 첫째 자리까지 통일하면 시각적으로 깔끔하고, 값 간 비교도 훨씬 용이해집니다. 적절한 콤마, 백분율, 소수점 표기는 독자가 데이터를 빠르게 이해하도록 도와줍니다.

작은 디테일이 큰 차이를 만듭니다!