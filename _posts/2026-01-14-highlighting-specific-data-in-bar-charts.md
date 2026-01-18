---
title: 특정 막대 색상 강조하는 법
author: Rvinci
description: 막대그래프에서 상하위권 혹은 특정 조건에 맞는 값을 강조하고 싶었던 적이 있나요? R을 이용해 특정 조건을 만족하는 막대그래프 색상을 바꾸는 법을 소개합니다.
excerpt: 막대그래프에서 상하위권 혹은 특정 조건에 맞는 값을 강조하고 싶었던 적이 있나요? R을 이용해 특정 조건을 만족하는 막대그래프 색상을 바꾸는 법을 소개합니다.
categories: [chart]
tags: [r, tidyverse, case_when, scale_fill_identity]
published: true
---

시각화의 핵심은 단순히 데이터를 보여주는 것이 아니라, 데이터가 담고 있는
메시지를 효과적으로 전달하는 데 있습니다. 수많은 막대가 나열된
그래프에서 우리가 주목해야 할 지점을 단번에 보여주기 위해 가장 효과적인
방법은 바로 ’색상의 대비’를 활용하는 것입니다. 오늘은 2023년 서울특별시
25개 구의 주민등록인구 데이터를 활용해, 인구가 가장 많은 상위 3개 구를
자동으로 강조하는 차트를 만들어 보겠습니다.

## 데이터 준비하기

먼저 분석에 필요한 도구들을 챙겨야 합니다. R의 강력한 데이터 가공
도구이자 시각화 도구인 `tidyverse`를 로드합니다. 또한 KoPub 돋움 같은
깔끔한 폰트를 사용하기 위해 `showtext` 패키지를 로드합니다.

``` r
# 패키지 로드
library(readxl)
library(tidyverse)
library(showtext)
library(scales)

# 차트 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/KoPub Dotum Medium.ttf")
showtext_auto()
showtext_opts(dpi=300)

theme.size = 12
text.size = theme.size / .pt
```

데이터는 통계청에서 제공하는 [주민등록인구
자료](https://kosis.kr/statHtml/statHtml.do?orgId=101&tblId=DT_1YL20651E&vw_cd=MT_GTITLE01&list_id=101&scrId=&seqNo=&lang_mode=ko&obj_var_id=&itm_id=&conn_path=MT_GTITLE01&path=%252FstatisticsList%252FstatisticsListIndex.do)를
활용합니다. 엑셀 파일을 읽어온 뒤, 분석에 불필요한 행을 제외하고
서울특별시 소속 구 데이터만 골라내는 과정이 필요합니다. 이때 `rename()`
함수를 써서 칼럼명을 ‘구’, ’인구수’와 같이 바꿔주면 이후 활용하기
수월해집니다.

``` r
# 데이터 로드
data <- read_xlsx("데이터/주민등록인구_시도_시군구_2023.xlsx", skip=1)

# 서울 데이터만 추출
seoul <- data %>% 
  rename(시도 = `행정구역별(1)`,
         구 = `행정구역별(2)`,
         인구수 = `계 (명)`) %>% 
  filter(시도 == "서울특별시",
         구 != "소계")

# 데이터 확인
head(seoul)
```

    ## # A tibble: 6 × 3
    ##   시도       구       인구수
    ##   <chr>      <chr>     <dbl>
    ## 1 서울특별시 종로구   139417
    ## 2 서울특별시 중구     121312
    ## 3 서울특별시 용산구   213151
    ## 4 서울특별시 성동구   277361
    ## 5 서울특별시 광진구   335554
    ## 6 서울특별시 동대문구 341149

## 막대그래프 내림차순 정렬하기

데이터 준비가 끝났다면 먼저 기본 그래프를 그려봅니다. 막대그래프를 그릴
때 가장 중요한 점은 가나다순이 아닌 크기순으로 정렬하는 것입니다.
엑셀에서는 데이터 탭에서 정렬 기능을 써야 하지만, R의 `ggplot2`에서는
`reorder()` 함수를 통해 차트 생성과 동시에 정렬을 수행할 수 있습니다.
인구수 단위를 ’만 명’으로 조정하면 차트가 한결 깔끔해집니다.

``` r
seoul %>% 
  ggplot(aes(x = reorder(구, -인구수), y = 인구수 / 10000)) +
  geom_col(width = 0.6) +
  scale_x_discrete(name = "") +
  scale_y_continuous(name = "주민등록인구 수(만 명)",
                     expand = expansion(mult = c(0, 0.1))) +
  theme_minimal(base_size = theme.size, base_family = "kopub")
```

<img src="{{ site.baseurl }}/assets/images/2026-01-14-highlighting-specific-data-in-bar-charts/unnamed-chunk-3-1.png" alt="" width="100%" style="display: block; margin: auto;" />

## 상위 3개 구 강조하기

이제 이 포스팅의 핵심인 ’상위 3개 구’를 강조할 차례입니다. 엑셀에서는
막대를 직접 클릭해 색을 바꿨겠지만, R에서는 데이터 자체에 ’색상 정보’를
추가하는 방식을 사용합니다. `mutate()`와 `case_when()` 함수를 결합해
인구수 순위를 매기고, 1위부터 3위까지는 진한 녹색(#027453)을, 나머지는
차분한 회색(#D9D8D6)을 적용합니다.

여기에 `scale_fill_identity()`라는 함수를 추가하면, 우리가
데이터프레임에 입력한 색상 코드가 그래프에 그대로 반영됩니다. 이 방식의
최대 장점은 데이터가 바뀌더라도 코드를 수정할 필요 없이 상위 3개를
찾아내 강조해준다는 점입니다.

``` r
seoul_clr <- seoul %>% 
  mutate(순위 = rank(-인구수),
    색상 = case_when(순위 < 4 ~ "#027453",
                   TRUE ~ "#D9D8D6"))

seoul_clr %>% 
  ggplot(aes(x = reorder(구, -인구수), y = 인구수 / 10000,
             fill = 색상)) +
  geom_col(width = 0.6) +
  scale_x_discrete(name = "") +
  scale_y_continuous(name = "주민등록인구 수(만 명)",
                     expand = expansion(mult = c(0, 0.1))) +
  scale_fill_identity() +
  theme_minimal(base_size = theme.size, base_family = "kopub")
```

<img src="{{ site.baseurl }}/assets/images/2026-01-14-highlighting-specific-data-in-bar-charts/unnamed-chunk-4-1.png" alt="" width="100%" style="display: block; margin: auto;" />

## 차트 다듬기

마지막으로 차트를 좀 더 다듬어 보겠습니다. 각 막대 위에 정확한
숫자(데이터 레이블)를 표시해 정보를 추가해줍니다. 또한 불필요한
격자(Grid line)를 제거하여 시선의 분산을 막고, x축 라인을 강조해
안정감을 더해줍니다.

``` r
seoul_clr %>% 
  ggplot(aes(x = reorder(구, -인구수), y = 인구수 / 10000,
             fill = 색상)) +
  geom_col(width = 0.6,
           color = "gray10",
           linewidth = 0.2) +
  geom_text(aes(label = comma(인구수 / 10000, accuracy = 1)),
            family = "kopub",
            size = text.size,
            color = "gray10",
            vjust = -1) +
  scale_x_discrete(name = "") +
  scale_y_continuous(name = "주민등록인구 수(만 명)",
                     expand = expansion(mult = c(0, 0.1))) +
  scale_fill_identity() +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    panel.grid = element_blank(),
    panel.grid.major.y = element_line(linewidth = 0.2, color = "gray40"),
    
    axis.line.x = element_line(linewidth = 0.8, color = "gray10"),

    axis.text = element_text(color = "gray10"),
    axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1)
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-14-highlighting-specific-data-in-bar-charts/unnamed-chunk-5-1.png" alt="서울시 인구 수 상위 3개 구를 색상으로 강조한 막대그래프" width="100%" style="display: block; margin: auto;" />

이번 글에서 다룬 내용은 상위 3개 데이터를 강조하는 것에만 활용할 수 있는 것은 아닙니다. ‘목표 인구 50만을 넘긴 곳’ 등 여러분이 원하는 어떤
조건이든 `case_when` 문에 넣어 원하는 곳의 색상을 바꿀 수 있습니다. 이제
여러분의 데이터 분석 목적에 맞는 조건을 자유롭게 설정하여, 강조하고 싶은 데이터를 돋보이게 만들어 보세요!