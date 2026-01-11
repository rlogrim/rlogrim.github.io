---
title: 막대+선 혼합형 그래프 그리기
author: Rvinci
description: 막대그래프 위에 선그래프를 추가했는데 선이 보이지 않나요? 스케일을 조정하고 보조 y축을 활용해 혼합형 그래프를 만드는 방법을 설명합니다.
excerpt: 막대그래프 위에 선그래프를 추가했는데 선이 보이지 않나요? 스케일을 조정하고 보조 y축을 활용해 혼합형 그래프를 만드는 방법을 설명합니다.
categories: [chart]
tags: [r, ggplot2, tidyverse, geom_col, geom_line, sec.axis]
published: true
---

연도별 수치를 시각화할 때 절대값(규모)과 변화율(추세)을 함께 보고 싶은
경우가 있습니다. 예를 들어 항공 승객 수의 전체 규모와 함께 전년 대비
증감률을 동시에 표현하고 싶을 때입니다. 이럴 때 막대그래프와 선그래프를
조합한 혼합형 그래프(이중축 그래프)가 유용합니다.

이런 분들을 위한 글입니다.
- 막대와 선 그래프를 함께 그리고 싶은데 방법을 모르는 경우
- 값의 크기가 다른 두 지표를 한 그래프에 표현하고 싶은 경우
- 이중 축이 왜 필요한지 궁금한 경우

이번에는 R에 내장된 `AirPassengers` 데이터를 사용합니다. 이 데이터는
1949년부터 1960년까지 월별 항공 승객 수를 담고 있습니다(단위: 천 명).

`AirPassengers`는 시계열(ts) 형태이므로, `ggplot2`로 시각화하려면
데이터프레임으로 변환해야 합니다. 각 행은 연도, 각 열(1~12)은 월별 승객 수를 나타냅니다.

``` r
# 패키지 로드
library(tidyverse)

# 데이터 변환
data <- data.frame(matrix(AirPassengers, ncol = 12, byrow = TRUE))

colnames(data) <- seq(1, 12) # 열 이름 변경 (1~12월)

data <- data %>% mutate(년도 = seq(1949, 1960))

# 데이터 확인
head(data)
```

    ##     1   2   3   4   5   6   7   8   9  10  11  12 년도
    ## 1 112 118 132 129 121 135 148 148 136 119 104 118 1949
    ## 2 115 126 141 135 125 149 170 170 158 133 114 140 1950
    ## 3 145 150 178 163 172 178 199 199 184 162 146 166 1951
    ## 4 171 180 193 181 183 218 230 242 209 191 172 194 1952
    ## 5 196 196 236 235 229 243 264 272 237 211 180 201 1953
    ## 6 204 188 235 227 234 264 302 293 259 229 203 229 1954

이제 연도별로 승객 수를 합산하고 전년 대비 증감률을 계산합니다.
`rowSums()`로 각 행의 1~12월 값을 모두 더합니다. `lag()`로 전년도 값을
가져와서 증감률을 계산합니다.

``` r
# 데이터 합산
data2 <- data %>% 
  mutate(
    `승객 수` = rowSums(across(1:12)),
    `전년 대비 승객 수 증감률` = (`승객 수` - lag(`승객 수`)) / lag(`승객 수`)
  ) %>% 
  select(년도, `승객 수`, `전년 대비 승객 수 증감률`)

# 데이터 확인
head(data2)
```

    ##   년도 승객 수 전년 대비 승객 수 증감률
    ## 1 1949    1520                       NA
    ## 2 1950    1676               0.10263158
    ## 3 1951    2042               0.21837709
    ## 4 1952    2364               0.15768854
    ## 5 1953    2700               0.14213198
    ## 6 1954    2867               0.06185185

## 기본 막대그래프 그리기

먼저 연도별 승객 수를 막대그래프로 그려줍니다. 막대그래프는
`geom_col()`로 그릴 수 있습니다.

``` r
# 패키지 로드
library(ggplot2)

# 막대그래프 생성
ggplot(data = data2) +
  geom_col(aes(x = 년도, y = `승객 수`), width = 0.6) +
  scale_x_continuous(n.breaks = 12) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-03-creating-a-bar-and-line-combo-chart/unnamed-chunk-3-1.png" alt="" width="70%" style="display: block; margin: auto;" alt="연도별 승객 수 막대그래프" />

## 선그래프 추가하기

앞서 그려놓은 막대그래프에 전년 대비 승객 수 증감률을 선그래프로
추가합니다. 선그래프는 `geom_line()`으로 그릴 수 있습니다. 그러나 값의
차이로 인해 선그래프가 거의 보이지 않습니다. 값의 차이가 커서 선그래프가
화면 아래쪽에 붙어버립니다.

``` r
# 선그래프 추가
ggplot(data = data2) +
  geom_col(aes(x = 년도, y = `승객 수`), width = 0.6) +
  geom_line(aes(x = 년도, y = `전년 대비 승객 수 증감률`)) +
  scale_x_continuous(n.breaks = 12) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-03-creating-a-bar-and-line-combo-chart/unnamed-chunk-4-1.png" alt="" width="70%" style="display: block; margin: auto;" alt="연도별 승객 수 막대그래프와 전년 대비 승객 수 증감률 선그래프" />

## 선그래프 스케일 조정하기

두 지표를 같은 그래프에 표현하려면 증감률에 적절한 상수를 곱해 위치를
조정해야 합니다. `coeff`는 증감률을 승객 수 범위에 맞추기 위한 비율을
나타냅니다. 증감률에 `coeff`를 곱하면 선그래프가 적절한 높이에
표시됩니다.

``` r
# 상수 계산
coeff <- max(data2$`승객 수`, na.rm = TRUE) / (max(data2$`전년 대비 승객 수 증감률`, na.rm = TRUE) * 2)

# 조정된 선그래프 생성
ggplot(data = data2) +
  geom_col(aes(x = 년도, y = `승객 수`), width = 0.6) +
  geom_line(aes(x = 년도, y = `전년 대비 승객 수 증감률` * coeff)) +
  scale_x_continuous(n.breaks = 12) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-03-creating-a-bar-and-line-combo-chart/unnamed-chunk-5-1.png" alt="" width="70%" style="display: block; margin: auto;" alt="선그래프 스케일 조정한 그래프" />

## 이중축 표시하기

이제 선그래프의 값 수준을 확인할 수 있도록 오른쪽에 보조 y축을
추가합니다.`sec.axis`를 사용해 두 번째 y축을 만들고, 선그래프에 곱해
주었던 상수(`coeff`)를 `~./coeff`로 다시 나누어 원래 증감률 값이
표시되도록 합니다. `scales::percent_format()`을 적용해 증감률을
퍼센트(%) 단위로 표현합니다.

``` r
# 이중축 추가
ggplot(data = data2) +
  geom_col(aes(x = 년도, y = `승객 수`), width = 0.6) +
  geom_line(aes(x = 년도, y = `전년 대비 승객 수 증감률` * coeff)) +
  scale_x_continuous(n.breaks = 12) +
  scale_y_continuous(
    name = "승객 수(천명)",
    labels = scales::comma_format(),
    sec.axis = sec_axis(~./coeff, name = "전년 대비 승객 수 증감률(%)",
                        labels = scales::percent_format())
  ) +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-03-creating-a-bar-and-line-combo-chart/unnamed-chunk-6-1.png" alt="" width="70%" style="display: block; margin: auto;" alt="이중축 추가한 그래프" />

## 범례 추가로 완성하기

막대그래프와 선그래프가 각각 무엇을 의미하는지 분명히 보여주기 위해
범례를 추가합니다.

이를 위해 `geom_col(aes())`에는 `fill = "승객 수"`를,
`geom_line(aes())`에는 `color = "전년 대비 승객 수 증감률"`을 지정해
범례 항목을 생성합니다. 이후 `scale_fill_manual()`과
`scale_color_manual()`을 사용해 막대와 선의 색상을 각각 지정합니다.

또한 `theme()`에서 범례와 관련된 요소를 조정해 그래프를 깔끔하게
정리합니다. 이 예제에서는 범례 제목과 배경, 여백을 제거하고, 범례가
그래프 좌측 상단에 위치하도록 설정했습니다.

``` r
ggplot(data = data2) +
  geom_col(aes(x = 년도, y = `승객 수`, fill = "승객 수"), width = 0.6) +
  geom_line(aes(x = 년도, y = `전년 대비 승객 수 증감률` * coeff, color = "전년 대비 승객 수 증감률")) +
  scale_fill_manual(values = "gray40") +
  scale_color_manual(values = "gray70") +
  scale_x_continuous(n.breaks = 12) +
  scale_y_continuous(
    name = "승객 수(천명)",
    labels = scales::comma_format(),
    sec.axis = sec_axis(~./coeff, name = "전년 대비 승객 수 증감률(%)",
                        labels = scales::percent_format())
  ) +
  theme_bw() +
  theme(
    legend.title = element_blank(),
    legend.background = element_blank(),
    legend.position = "inside",
    legend.position.inside = c(0.05, 0.95),
    legend.justification = c(0, 1),
    legend.margin = margin(0, 0, 0, 0)
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-03-creating-a-bar-and-line-combo-chart/unnamed-chunk-7-1.png" alt="" width="70%" style="display: block; margin: auto;" alt="범례 추가한 혼합형 그래프" />

혼합형 그래프는 세 단계로 만들 수 있습니다. 먼저 문제를 파악합니다.
크기가 다른 두 지표를 그대로 겹치면 한쪽이 보이지 않습니다. 다음으로,
선그래프에 상수를 곱해 스케일을 맞춥니다. 이렇게 하면 막대그래프와 함께
나타낼 수 있습니다. 마지막으로 보조 y축을 추가하면, 혼합형 그래프가
완성됩니다.

이 방법은 다양한 상황에서 활용할 수 있습니다. 매출과 성장률을 함께
보여줄 때, 지역별 인구와 증가율을 비교할 때, 정책 지표의 수준과 변화를
동시에 표현할 때도 같은 방식을 쓸 수 있습니다.

다만 주의할 점이 있습니다. 이중축 그래프는 신중하게 사용해야 합니다.
관련 없는 두 지표를 함께 놓으면 독자가 혼란스러워할 수 있습니다. 의미
있는 관계가 있을 때만 사용하세요. 그리고 명확한 범례와 축 이름을
제공하는 것을 잊지마세요!