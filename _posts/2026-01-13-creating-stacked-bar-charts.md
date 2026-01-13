---
title: 누적막대그래프 그리기
author: Rvinci
description: 누적막대그래프는 여러 범주의 ‘구성 비율’을 한눈에 보여줍니다. 누적막대그래프를 어떻게 그려야 할지 막막하신가요? 시도별 시·군·구 주민등록인구 비중 데이터를 예로, ggplot2에서 누적막대그래프를 만드는 방법을 알려드립니다.
excerpt: 누적막대그래프는 여러 범주의 ‘구성 비율’을 한눈에 보여줍니다. 누적막대그래프를 어떻게 그려야 할지 막막하신가요? 시도별 시·군·구 주민등록인구 비중 데이터를 예로, ggplot2에서 누적막대그래프를 만드는 방법을 알려드립니다.
categories: [chart]
tags: [r, tidyverse, ggplot2, geom_col, position_stack]
published: true
---

누적막대그래프는 전체를 구성하는 부분들의 비율을 보여주는 가장 기본적인
시각화 도구입니다.

예를 들어 어떤 지역의 인구가 시, 군, 구로 어떻게 분포되어 있는지
비교해야 할 때, 누적막대그래프를 사용할 수 있습니다. 이 그래프는 계산된
비율을 하나의 막대에 차례로 쌓아 올려서 표현합니다. 막대의 전체 길이는
100%를 나타내고, 서로 다른 색으로 구분된 구간이 각 범주가 차지하는
비율을 의미합니다. 따라서 지역별 구성이 어떻게 다른지 직관적으로 파악할
수 있습니다.

이 글에서는 2023년 시도별 시, 군, 구 주민등록인구 비중 데이터를 활용하여
누적막대그래프를 그리는 기본 과정을 단계별로 설명합니다. 이어서 그래프의
완성도를 높이는 과정에서 흔히 발생하는 라벨 겹침 문제와 x축 이름 축약
방법도 함께 다룹니다.

## 데이터 준비하기

먼저 [이전
글](https://rlogrim.github.io/data/from-counts-to-shares/)에서 만든
데이터를 사용하겠습니다. 여기서 중요한 점은 두 가지입니다.

첫째, 시도와 구분(시/군/구) 단위로 인구를 집계해야 합니다.
누적막대그래프는 큰 범주(시도) 안에 작은 범주(시/군/구)를 쌓아 올리는
구조로 표현되기 때문입니다.

둘째, 집계가 완료되면 시도별 총인구로 나누어 비율을 계산해야 합니다.
누적막대그래프에서 각 막대의 전체 길이를 1로 맞추기 위한 과정입니다.

엑셀파일을 불러온 후 필요한 열 이름을 정리하고, 시도 및 구분(시/군/구)별
인구 합계와 비율을 계산합니다.

``` r
# 패키지 로드
library(tidyverse)
library(readxl)
library(scales)

# 데이터 불러오기
raw <- read_xlsx("데이터/주민등록인구_시도_시군구_2023.xlsx", 
                  skip=1)

# 데이터 정리하기
data <- raw %>% 
  rename(
    시도 = `행정구역별(1)`,
    시군구 = `행정구역별(2)`,
    주민등록인구수 = `계 (명)`
    ) %>% 
  filter(시도 != "전국",
         (시도 == "세종특별자치시")|(시군구 != "소계")) %>% 
  mutate(시군구 = if_else(시도 == "세종특별자치시", "세종특별자치시", 시군구),
         시도 = factor(시도, levels = unique(raw$`행정구역별(1)`)[-1]),
         구분 = case_when(
           str_ends(시군구, "시") ~ "시",
           str_ends(시군구, "군") ~ "군",
           str_ends(시군구, "구") ~ "구",
           TRUE ~ NA_character_
         ),
         구분 = factor(구분, levels = c("시", "군", "구"))
         )

# 비율 계산하기
share <- data %>% 
  group_by(시도, 구분) %>% 
  summarise(주민등록인구수 = sum(주민등록인구수, na.rm = TRUE)) %>% 
  group_by(시도) %>% 
  mutate(비율 = 주민등록인구수 / sum(주민등록인구수, na.rm = TRUE))

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

## 누적막대그래프 그리기

이제 본격적으로 그래프를 작성합니다. 누적막대그래프는 생각보다
간단합니다. `ggplot()`에서 x축에는 `시도`를, y축에는 `비율`을 지정하고,
`fill`에 `구분`(시/군/구)을 매핑한 뒤 `geom_col()`을 사용하면 됩니다.
`geom_col()`은 입력된 y값을 그대로 막대의 높이로 표현하는 함수이며,
막대를 누적하는 방식이 기본 설정이므로 별도의 옵션 없이도 자동으로
쌓입니다.

막대 안에 비율을 함께 표시하면 좋습니다. 이때 `geom_text()`를
사용하는데, 누적막대의 각 구간 중앙에 텍스트를 배치하려면 반드시
`position_stack(vjust = 0.5)`를 지정해야 합니다.

``` r
ggplot(
  data = share,
  aes(
    x = 시도, y = 비율, fill = 구분,
    label = percent(비율, accuracy = 0.1, suffix = "")
    )
  ) +
  geom_col(color = "white", linewidth = 0.4, width = 0.6) +
  geom_text(
    family = "kopub",
    position = position_stack(vjust = 0.5),
    size = text.size
    ) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  scale_fill_brewer(palette = "Pastel2") +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.title = element_blank(),
    axis.text.y = element_blank(),
    
    axis.line.y = element_blank(),
    axis.line.x = element_line(color = "#959595", linewidth = 1.5),
    
    legend.title = element_blank(),
    legend.position = "bottom",
    legend.box.background = element_rect(color = "#959595", linewidth = 1),
    legend.key.size = unit(theme.size, "pt"),
    
    panel.grid = element_blank()
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-13-creating-stacked-bar-charts/unnamed-chunk-2-1.png" width="100%" style="display: block; margin: auto;" />

여기까지가 이 글의 핵심 내용입니다. 누적막대그래프 작성은 ‘비율을
계산하고’, ‘`fill`로 범주를 구분하여 쌓아 올리며’, ’필요시 라벨을
추가한다’는 세 단계로 요약됩니다.

## x축 시도명 줄이고 라벨 겹침 문제 해결하기

실제 작업에서는 그래프를 그린 후에도 추가적인 수정이 필요한 경우가
많습니다. 시도 이름이 길면 x축 라벨이 서로 겹치고, 비율이 작은 구간에는
라벨이 중복되어 표시되기도 합니다.

첫째, 시도명을 축약하여 x축을 정리합니다.

둘째, 라벨이 겹칠 때는 `ggrepel` 패키지의 `geom_text_repel()` 함수를
사용하여 텍스트가 서로 밀어내도록 조정함으로써 가독성을 높입니다.

시도명을 축약하는 과정에서 `case_when()`을 사용하면 결과가 문자형으로
변환되므로, 원래의 표시 순서를 유지하려면 `factor()`로 수준(`level`)을
다시 지정해줘야 합니다.

``` r
# 패키지 로드
library(ggrepel)

# x축 시도명 축약하여 표현하기
share2 <- share %>% 
  mutate(시도 = case_when(
    str_detect(시도, "(특별시)|(광역시)|(자치시)|(특별자치도)$") ~ str_sub(시도, 1, 2),
    시도 == "경기도" ~ "경기",
    TRUE ~ paste0(str_sub(시도, 1, 1), str_sub(시도, 3, 3))
    ),
    시도 = factor(시도, 
                levels = c("서울", "부산", "대구", "인천", "광주", "대전", "울산", "세종", "경기", "강원", "충북", "충남", "전북", "전남", "경북", "경남", "제주")
                )
    )

# 누적막대그래프 그리기
ggplot(
  data = share2,
  aes(
    x = 시도, y = 비율, fill = 구분,
    label = percent(비율, accuracy = 0.1, suffix = "")
    )
  ) +
  geom_col(color = "white", linewidth = 0.4, width = 0.6) +
  geom_text_repel(
    family = "kopub",
    position = position_stack(vjust = 0.5),
    size = text.size,
    force = 0.001,
    force_pull = 1000
  ) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  scale_fill_brewer(palette = "Pastel2") +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.title = element_blank(),
    axis.text.y = element_blank(),
    
    axis.line.y = element_blank(),
    axis.line.x = element_line(color = "#959595", linewidth = 1.5),
    
    legend.title = element_blank(),
    legend.position = "bottom",
    legend.box.background = element_rect(color = "#959595", linewidth = 1),
    legend.key.size = unit(theme.size, "pt"),
    
    panel.grid = element_blank()
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-13-creating-stacked-bar-charts/unnamed-chunk-3-1.png" width="100%" style="display: block; margin: auto;" alt="누적막대그래프" />

라벨이 여전히 빽빽하게 느껴진다면, 한 가지 원칙을 기억하시면 좋습니다.
’비율이 작은 구간에는 라벨을 과감히 생략한다’는 것입니다.
누적막대그래프는 전체 구성비를 보여주는 것이 목적이므로, 모든 구간에
숫자를 표시하는 것이 항상 최선은 아닙니다.
