---
title: 선 그래프 끝에 라벨 달기
author: Rvinci
description: 선이 여러 개인 그래프를 볼 때, 범례와 선을 번갈아 보느라 힘들었던 적 있지 않나요? 이번 글에서는 ggplot2 선 그래프 끝에 이름을 직접 붙여, 범례 없이도 한눈에 들어오는 직관적인 차트를 만드는 방법을 소개합니다.
excerpt: 선이 여러 개인 그래프를 볼 때, 범례와 선을 번갈아 보느라 힘들었던 적 있지 않나요? 이번 글에서는 ggplot2 선 그래프 끝에 이름을 직접 붙여, 범례 없이도 한눈에 들어오는 직관적인 차트를 만드는 방법을 소개합니다.
categories: [chart]
tags: [r, tidyverse, ggrepel, labeling]
published: true
---

선 그래프에 선이 많아지면 고민이 시작됩니다. 색깔만으로는 어떤 선이 어떤
그룹인지 구분하기 어렵고, 그렇다고 범례를 넣자니 시선이 ’선 → 범례 →
다시 선’으로 계속 왔다 갔다 하며 흐름이 끊기기 때문이죠.

이럴 때 가장 좋은 해결책은 각 선의 ’끝 지점’에 그룹 이름을 직접 적어주는
것입니다. 이번 포스팅에서는 1995년부터 2023년까지의 시도별 주민등록인구
데이터를 활용해, 범례 없이도 한눈에 들어오는 직관적인 그래프를 만드는
방법을 알아보겠습니다.

## 데이터 준비하기

먼저 분석에 필요한 패키지를 불러오고 차트 글꼴을 설정합니다. 선 끝에
글자를 붙일 때 가장 큰 문제는 ’글자 끼리 겹치는 현상’인데요, 이를
똑똑하게 해결해 주는 `ggrepel` 패키지를 사용하겠습니다.

``` r
# 패키지 로드
library(readxl)
library(tidyverse)
library(showtext)
library(scales)
library(lubridate)
library(ggrepel)

# 차트 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/KoPub Dotum Medium.ttf")
showtext_auto()
showtext_opts(dpi=300)

# 그래프 글꼴 크기 설정
theme.size = 12
text.size = theme.size / .pt
```

그다음, 통계청에서 받은 [인구
데이터](https://kosis.kr/statHtml/statHtml.do?sso=ok&returnurl=https%3A%2F%2Fkosis.kr%3A443%2FstatHtml%2FstatHtml.do%3Flist_id%3D101%26obj_var_id%3D%26seqNo%3D%26tblId%3DDT_1YL20651E%26vw_cd%3DMT_GTITLE01%26orgId%3D101%26path%3D%252FstatisticsList%252FstatisticsListIndex.do%26conn_path%3DMT_GTITLE01%26itm_id%3D%26lang_mode%3Dko%26scrId%3D%26)를
불러와 R에서 다루기 좋게 가공합니다.

``` r
# 데이터 로드
data <- read_xlsx("데이터/주민등록인구_시도_1995-2023.xlsx")

# 데이터 가공
data <- data %>% 
  rename(
    시도 = 행정구역별,
    주민등록인구수 = `계 (명)`
    ) %>% 
  filter(시도 != "전국") %>% 
  mutate(
    시점 = as.character(시점),
    주민등록인구수 = as.numeric(주민등록인구수)
    )

# 데이터 확인
head(data)
```

    ## # A tibble: 6 × 3
    ##   시도       시점  주민등록인구수
    ##   <chr>      <chr>          <dbl>
    ## 1 서울특별시 1995        10550871
    ## 2 서울특별시 1996        10418076
    ## 3 서울특별시 1997        10336134
    ## 4 서울특별시 1998        10270506
    ## 5 서울특별시 1999        10264260
    ## 6 서울특별시 2000        10311314

## 기본 선 그래프 그려 문제 확인하기

가공한 데이터를 그대로 그래프로 그려보겠습니다. 이 그래프에는 두 가지
문제가 있습니다.

1.  Y축 숫자가 너무 커서 읽기 어렵습니다.
2.  선이 너무 많아 오른쪽 범례에서 색을 일일이 대조하여 이름을 찾기
    힘듭니다.

``` r
data %>% 
  ggplot(aes(x = 시점, 
             y = 주민등록인구수, 
             group = 시도, 
             color = 시도)) +
  geom_line(linewidth = 0.4) +
  geom_point(size = 3) +
  scale_x_discrete(name = "") +
  scale_y_continuous(name = "주민등록인구 수(명)",
                     expand = expansion(mult = c(0, 0.1))) +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.line = element_line(linewidth = 0.5, color = "gray10"),
    axis.ticks = element_line(linewidth = 0.1, color = "gray10"),
    
    panel.grid.minor = element_blank(),
    
    axis.text = element_text(color = "gray10")
    )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-11-labeling-line-ends/unnamed-chunk-3-1.png" alt="" width="100%" style="display: block; margin: auto;" />

## 그래프 다듬기: 단위 변환과 데이터 필터링

가독성을 높이기 위해 두 가지 처리를 먼저 하겠습니다. ‘명’ 단위를 ‘만 명’
단위로 바꿔 숫자를 간결하게 만듭니다. 인구 수가 압도적으로 많은 서울과
경기도를 제외하여, 나머지 시도들의 변화 추이가 더 잘 보이도록 합니다.

``` r
data_filtered <- data %>% 
  filter(!시도 %in% c("서울특별시", "경기도"))

data_filtered %>% 
  ggplot(aes(x = 시점, 
             y = 주민등록인구수 / 10000, 
             group = 시도, 
             color = 시도)) +
  geom_line(linewidth = 0.4,
            show.legend = FALSE) +
  geom_point(size = 3,
            show.legend = FALSE) +
  scale_x_discrete(name = "") +
  scale_y_continuous(name = "주민등록인구 수(만 명)",
                     labels = comma_format(accuracy = 1),
                     expand = expansion(mult = 0.1)) +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.line = element_line(linewidth = 0.5, color = "gray10"),
    axis.ticks = element_line(linewidth = 0.1, color = "gray10"),
    
    panel.grid.minor = element_blank(),
    
    axis.text = element_text(color = "gray10"),
    axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1)
    )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-11-labeling-line-ends/unnamed-chunk-4-1.png" alt="" width="100%" style="display: block; margin: auto;" />

## 선 끝에 시도명 라벨 붙이기

이제 마지막 단계입니다. 범례를 없앤 대신, 가장 최근 시점(2023년) 데이터
옆에 이름을 붙여주겠습니다. 이름이 너무 길면 겹치기 쉬우므로
“서울특별시”는 “서울”로, “충청남도”는 “충남”처럼 짧게 줄여서 표시하는
것이 핵심입니다.

``` r
sido_text <- data %>%
  mutate(시도 = case_when(grepl("(특별시)|(광역시)|(자치시)|(특별자치도)$", 시도) ~ substr(시도, 1, 2),
                        시도 == "경기도" ~ "경기",
                        TRUE ~ paste0(substr(시도, 1, 1), substr(시도, 3, 3))
                        )) %>% 
  filter(시점 == "2023",
         !시도 %in% c("서울", "경기"))

data_filtered %>% 
  ggplot(aes(x = 시점, 
             y = 주민등록인구수 / 10000, 
             group = 시도, color = 시도)) +
  geom_line(linewidth = 0.4,
            show.legend = FALSE) +
  geom_point(size = 3,
            show.legend = FALSE) +
  geom_text_repel(data = sido_text,
    aes(x = 시점, 
        y = 주민등록인구수 / 10000, 
        label = 시도),
    color = "gray10",
    hjust = 0,
    nudge_x = 3,
    direction = "y",
    segment.color = "gray80",
    family = "kopub",
    size = text.size
    ) +
  scale_x_discrete(name = "") +
  scale_y_continuous(name = "주민등록인구 수(만 명)",
                     labels = comma_format(accuracy = 1),
                     expand = expansion(mult = 0.1)) +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.line = element_line(linewidth = 0.5, color = "gray10"),
    axis.ticks = element_line(linewidth = 0.1, color = "gray10"),
    
    panel.grid.minor = element_blank(),
    
    axis.text = element_text(color = "gray10"),
    axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1)
    )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-11-labeling-line-ends/unnamed-chunk-5-1.png" alt="선 끝에 라벨 붙인 그래프" width="100%" style="display: block; margin: auto;" />

범례를 없애고 선 끝에 이름을 붙이니 그래프가 훨씬 시원하고 직관적이지
않나요?

💡 팁 하나 더! 만약 시도가 너무 많아 여전히 복잡해 보인다면, 모든 라벨을
다 붙이기보다 가장 인구가 많은 곳이나 변화율이 큰 곳만 선택해서 라벨을
붙이는 방식도 있습니다. `nudge_x`나 `size` 값을 조금씩 조절하며
여러분만의 깔끔한 차트를 만들어 보세요!