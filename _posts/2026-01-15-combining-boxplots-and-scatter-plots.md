---
title: 박스플롯과 산점도 겹쳐 그리는 법
author: Rvinci
description: 박스플롯을 그리고 난 후 개별 데이터 값도 확인하고 싶었던 적이 있나요? 박스플롯으로 데이터 경향성을 파악하고, 개별 데이터 산점도를 추가하는 방법을 소개합니다.
excerpt: 박스플롯을 그리고 난 후 개별 데이터 값도 확인하고 싶었던 적이 있나요? 박스플롯으로 데이터 경향성을 파악하고, 개별 데이터 산점도를 추가하는 방법을 소개합니다.
categories: [chart]
tags: [r, tidyverse, geom_boxplot, geom_jitter]
published: true
---

## 박스플롯과 산점도, 왜 같이 그려야 할까요?

박스플롯(Boxplot)은 데이터의 중앙값, 사분위수, 이상치 등을 한눈에
보여주는 도구입니다. 여러 그룹의 분포를 비교할 때 이보다 효율적인 차트는
드물죠. 하지만 박스플롯에는 결정적인 약점이 있습니다. 바로 개별 데이터의
값을 보여주지 않는다는 점입니다.

반면 산점도는 모든 데이터 포인트를 점으로 표시하여 데이터가 어디에
얼마나 밀집해 있는지 보여줍니다. 이 두 차트를 결합하면 전체적인 통계적
요약 정보와 세부적인 데이터의 분포를 동시에 확인할 수 있어, 데이터를
파악할 때 좋습니다. 오늘은 2023년 시도별 청년인구 비율 데이터를 활용해
이 두 차트를 겹쳐 그려보겠습니다.

## 데이터 준비하기

가장 먼저 분석에 필요한 패키지를 불러오고 시각화의 완성도를 높여줄
폰트를 설정합니다. R의 `tidyverse` 생태계는 데이터 가공부터 시각화까지
모든 과정에 사용할 수 있는 다양한 패키지를 담고 있습니다.

``` r
# 패키지 로드
library(tidyverse)
library(readxl)
library(showtext)
library(scales)

# 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/KoPub Dotum Medium.ttf")
showtext_auto()
showtext_opts(dpi=300)

theme.size = 12
text.size = theme.size / .pt
```

이번 예제에서는 통계청의 [주민등록인구
데이터](https://kosis.kr/statHtml/statHtml.do?orgId=101&tblId=DT_1B04005N&conn_path=I3)를
사용합니다. 만 20~34세를 청년층으로 정의하고, 각 시도별 전체 인구 대비
청년층의 비율을 계산하는 전처리 과정을 거칩니다. 엑셀에서 피벗 테이블을
돌리고 수식을 복사하던 과정을 R에서는 `group_by()`와 `summarise()`를
통해 코드 두 줄로 손쉽게 처리할 수 있습니다.

``` r
# 데이터 불러오기
pop <- read_xlsx("데이터/시도 인구 특성/5세별주민등록인구_시도_2023.xlsx")

# 청년 정의
age <- unique(pop$`5세별`)
youth <- age[5:6]

# 5세별 주민등록인구 데이터 가공하기
pop_sum <- pop %>% 
  rename(시도 = `행정구역(동읍면)별`) %>% 
  mutate(구분 = case_when(`5세별` %in% youth ~ "청년",
                        TRUE ~ "기타")) %>% 
  group_by(시도, 구분) %>% 
  summarise(인구 = sum(`2023`, na.rm = T)) %>% 
  pivot_wider(names_from = 구분, values_from = 인구) %>% 
  mutate(전체인구 = 청년 + 기타,
         청년비율 = 청년 / 전체인구) %>% 
  filter(시도 != "전국")

# 데이터 확인하기
head(pop_sum)
```

    ## # A tibble: 6 × 5
    ## # Groups:   시도 [6]
    ##   시도               기타    청년 전체인구 청년비율
    ##   <chr>             <dbl>   <dbl>    <dbl>    <dbl>
    ## 1 강원특별자치도  1364197  163610  1527807   0.107 
    ## 2 경기도         11948683 1682138 13630821   0.123 
    ## 3 경상남도        2928447  322711  3251158   0.0993
    ## 4 경상북도        2306069  248255  2554324   0.0972
    ## 5 광주광역시      1228332  190905  1419237   0.135 
    ## 6 대구광역시      2092490  282470  2374960   0.119

## 박스플롯 구성 요소 이해하기

차트를 그리기 전, 박스플롯이 우리에게 어떤 정보를 주는지 먼저
살펴봅시다. 박스플롯은 데이터의 분포를 1사분위수, 중앙값, 3사분위수,
이상치 등으로 요약해 보여줍니다.

- 박스: 중앙값을 기준으로 1사분위수(Q1)와 3사분위수(Q3) 사이의 범위를
  보여줍니다. 이 영역은 데이터의 중간 50%를 나타내며, 사분위
  범위(IQR)라고도 합니다.
- 수염: 데이터의 주요 범위를 나타냅니다. 수염의 끝은 Q1과 Q3에서 IQR의
  1.5배 내외에 위치하는 데이터를 가리킵니다. 기본값은 1.5배지만, `coef`
  파라미터로 조정할 수 있습니다.
- 끝선: 수염의 끝에 짧은 선으로 표시되며, 수염의 끝이 어디인지
  보여줍니다.
- 이상치: 수염 바깥에 위치한 데이터 포인트를 점으로 표시합니다.
- 중앙값: 박스 안의 굵은 선으로 표시되며, 데이터의 중앙값을 나타냅니다.

<img src="{{ site.baseurl }}/assets/images/2026-01-15-combining-boxplots-and-scatter-plots/박스플롯 구성요소.jpg" alt="박스플롯 구성 요소" width="70%" style="display: block; margin: auto;" />

## 박스플롯과 산점도 겹쳐 그리기

이제 본격적으로 차트를 그려보겠습니다. R 시각화의 장점은 도화지 위에
물감을 덧칠하듯 차트를 하나씩 쌓아 올릴 수 있다는 점입니다.

### 단계 1: 기본 박스플롯 그리기

가장 먼저 `geom_boxplot()`을 사용해 요약된 분포를 그려봅니다. 여기서
`staplewidth`를 조절하면 수염 끝에 가로선을 추가해 경계를 더 명확하게
만들 수 있습니다.

``` r
ggplot(pop_sum, aes(x = "", y = 청년비율)) +
  geom_boxplot(staplewidth = 0.2) +
  scale_y_continuous(labels = percent_format(accuracy=1)) +
  theme_minimal(base_size = theme.size, base_family = "kopub")
```

<img src="{{ site.baseurl }}/assets/images/2026-01-15-combining-boxplots-and-scatter-plots/unnamed-chunk-3-1.png" alt="" width="70%" style="display: block; margin: auto;" />

### 단계 2: 산점도 레이어 추가하기

여기에 개별 데이터 포인트를 얹어보겠습니다. 점을 그냥 찍으면 데이터가
겹쳐 보이기 때문에, `geom_jitter()`를 사용해 점들이 퍼지도록 합니다.
`width`와 `height`를 기본값으로 두면, 포인트가 80% 범위 내에서 무작위로
분산됩니다. 이번에는 좀 더 좁은 범위에서 데이터가 표시되도록
`width = 0.2`로 지정했습니다. 또한, 포인트 사이즈를 키우고 색상과
투명도를 조정했습니다.

``` r
ggplot(pop_sum, aes(x = "", y = 청년비율)) +
  geom_boxplot(staplewidth = 0.2, outlier.shape = NA) + # 중복 표시 방지를 위해 이상치 숨김
  geom_jitter(width = 0.2, size = 3, color = "royalblue", alpha = 0.5) +
  scale_y_continuous(labels = percent_format(accuracy=1), name = "청년인구 비율") +
  theme_minimal(base_size = theme.size, base_family = "kopub")
```

<img src="{{ site.baseurl }}/assets/images/2026-01-15-combining-boxplots-and-scatter-plots/unnamed-chunk-4-1.png" alt="" width="70%" style="display: block; margin: auto;" />

### 단계 3: 차트 다듬기

x축 제목을 없애고, 패널 그리드를 없앱니다. 
축의 선 굵기를 조절해주면 완성입니다!

``` r
ggplot(pop_sum, aes(x = "", y = 청년비율)) +
  geom_boxplot(staplewidth = 0.2, outlier.shape = NA) + # 중복 표시 방지를 위해 이상치 숨김
  geom_jitter(
    width = 0.2, 
    size = 3, 
    color = "royalblue", 
    alpha = 0.5) +
  scale_y_continuous(labels = percent_format(accuracy=1), name = "청년인구 비율") +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.title.x = element_blank(),
    axis.line = element_line(linewidth = 0.4),
    axis.ticks.y = element_line(linewidth = 0.1),
    axis.ticks.x = element_blank(),
    panel.grid = element_blank(),
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-15-combining-boxplots-and-scatter-plots/unnamed-chunk-5-1.png" alt="박스플롯과 산점도 같이 그린 그래프" width="70%" style="display: block; margin: auto;" />

박스플롯과 산점도의 결합은 데이터의 전체적인 경향성을 파악하면서도, 개별
데이터 값도 파악할 수 있게 해줍니다. 엑셀에서는 도표를 겹쳐
그리기 어렵지만, R에서는 코드 추가만으로 도표를 손쉽게 겹쳐 그릴 수
있습니다. 박스플롯과 산점도를 같이 그려 데이터 특성을 파악해 보세요!