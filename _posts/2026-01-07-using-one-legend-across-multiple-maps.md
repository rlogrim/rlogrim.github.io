---
title: 지도 범례 고정해서 시점별 변화 비교하기
author: Rvinci
description: 연도별 지도를 그렸는데 지도마다 색이 달라져서 값 비교가 어렵지 않았나요? 여러 지도가 같은 색상 범례를 공유하면 색만 봐도 값의 크기와 변화를 한 눈에 비교할 수 있습니다. 공통 범례 설정 방법을 알려드립니다.
excerpt: 연도별 지도를 그렸는데 지도마다 색이 달라져서 값 비교가 어렵지 않았나요? 여러 지도가 같은 색상 범례를 공유하면 색만 봐도 값의 크기와 변화를 한 눈에 비교할 수 있습니다. 공통 범례 설정 방법을 알려드립니다.
categories: [map]
tags: [r, tidyverse, sf, facet_wrap, scale_fill_gradient, limits]
published: true
---

## 왜 범례를 고정해야 하나요?

시점(연도)이 다른 지도를 비교할 때 자주 겪는 문제는 지도마다 색상
범례가 달라지는 것입니다. 예를 들어 1995년과 2023년 서울 자치구 인구
지도를 각각 따로 그리면, 각 그림은 각 데이터 범위에 맞춰 색을 나눕니다.
그러면 1995년 지도에서 ’진한 빨강’이더라도, 2023년 지도에서의 ’진한
빨강’과 같은 값이라고 말할 수 없습니다. 색이 비슷해 보여도 실제 값의
크기는 다를 수 있어서, 연도 간 비교하기 어려워집니다. 따라서, 연도 간
변화를 비교하고 싶다면, 색이 의미하는 값의 범위를 동일하게 맞춰야
합니다.

이 글에서는 두 연도의 지도를 한 화면에 배치하면서 하나의 범례를 제시하는
방법과, 지도를 따로 저장하더라도 범례를 고정하는 방법을 소개합니다.

## 데이터 준비

먼저 서울시 자치구 경계 shp 파일을 `st_read()`로 읽고, 1995년과 2023년
주민등록인구 데이터는 `read_xlsx()`로 불러옵니다.

서울시 자치구 경계 데이터는 V-World 디지털트윈국토의
[행정구역시군구_경계](https://www.vworld.kr/dtmk/dtmk_ntads_s002.do?svcCde=MK&dsId=30604)에서
다운받고, 주민등록인구 데이터는 통계청의
[주민등록인구(시도/시/군/구)](https://kosis.kr/statHtml/statHtml.do?orgId=101&tblId=DT_1YL20651E&vw_cd=MT_GTITLE01&list_id=101&scrId=&seqNo=&lang_mode=ko&obj_var_id=&itm_id=&conn_path=MT_GTITLE01&path=%252FstatisticsList%252FstatisticsListIndex.do)에서
다운받을 수 있습니다.

``` r
# 패키지 로드
library(tidyverse)
library(sf)
library(readxl)
library(scales)

# 서울시 자치구 shp 파일 로드
sgg <- st_read("데이터/수도권 지도/행정구역 경계/LARD_ADM_SECT_SGG_11_202405.shp")
```

    ## Reading layer `LARD_ADM_SECT_SGG_11_202405' from data source 
    ##   `...\데이터\수도권 지도\행정구역 경계\LARD_ADM_SECT_SGG_11_202405.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 25 features and 4 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 179191.4 ymin: 536562.8 xmax: 216242.3 ymax: 566863.5
    ## Projected CRS: KGD2002_Central_Belt_2010

``` r
# 서울시 자치구 주민등록인구수
pop_raw <- read_xlsx("데이터/주민등록인구_서울.xlsx")

# 데이터 확인
head(pop_raw)
```

    ## # A tibble: 6 × 3
    ##   행정구역별   시점  `계 (명)`
    ##   <chr>        <chr>     <dbl>
    ## 1 서울특별시   1995   10550871
    ## 2 서울특별시   2023    9386034
    ## 3 　　　종로구 1995     203086
    ## 4 　　　종로구 2023     139417
    ## 5 　　　중구   1995     143138
    ## 6 　　　중구   2023     121312

## 공간 데이터와 인구 데이터 조인하기

주민등록인구 데이터와 shp 파일을 조인하기 위해 먼저 `str_trim()`으로
자치구 이름의 불필요한 공백을 제거합니다. 이후 `str_replace()`로 shp
파일의 자치구 이름에서 ’서울특별시’라는 접두어를 제거하여 데이터의 이름
형식을 통일합니다. 마지막으로, `left_join()`을 통해 두 데이터를
결합합니다. 이렇게 하면 지도 데이터에 인구수가 결합되어 시각화에 사용할
수 있는 데이터가 완성됩니다.

``` r
# 데이터 가공
pop <- pop_raw %>% 
  mutate(행정구역별 = str_trim(행정구역별, "both")) %>% 
  rename(`인구 수(명)` = `계 (명)`)

# 데이터 조인
joined_data <- sgg %>% 
  mutate(SGG_NM = str_replace(SGG_NM, "서울특별시 ",  "")) %>% 
  left_join(pop, by=c("SGG_NM" = "행정구역별"))

# 데이터 확인
head(joined_data)
```

    ## Simple feature collection with 6 features and 6 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 195102.6 ymin: 545229.7 xmax: 202367.6 ymax: 559196.5
    ## Projected CRS: KGD2002_Central_Belt_2010
    ##   ADM_SECT_C SGG_NM SGG_OID COL_ADM_SE 시점 인구 수(명)
    ## 1      11110 종로구      11      11110 1995      203086
    ## 2      11110 종로구      11      11110 2023      139417
    ## 3      11140   중구      34      11140 1995      143138
    ## 4      11140   중구      34      11140 2023      121312
    ## 5      11170 용산구       1      11170 1995      254579
    ## 6      11170 용산구       1      11170 2023      213151
    ##                         geometry
    ## 1 MULTIPOLYGON (((197800 5590...
    ## 2 MULTIPOLYGON (((197800 5590...
    ## 3 MULTIPOLYGON (((202072.4 55...
    ## 4 MULTIPOLYGON (((202072.4 55...
    ## 5 MULTIPOLYGON (((197569.6 55...
    ## 6 MULTIPOLYGON (((197569.6 55...

## 방법 1: 한 번에 그리고 패싯으로 나누기

가장 간단한 방법은 두 시점을 한 번에 그린 뒤, `facet_wrap(~시점)`으로
패널만 나누는 것입니다. 같은 `scale_fill_gradient`를 공유하기 때문에
범례가 자동으로 통일됩니다.

이 방법은 코드가 단순하다는 장점이 있습니다. 지도를 두 장 그리는 것이
아니라 한 장의 그래프 안에서 패널만 나누는 방식이라, 범례가 자연스럽게
하나로 맞춰집니다.

``` r
ggplot(data = joined_data, aes(fill = `인구 수(명)`)) +
  geom_sf(color="gray20") +
  scale_fill_gradient(low = '#ffffff', 
                      high = '#ff6666',
                      labels = label_comma()) +
  facet_wrap(~시점) +
  geom_sf_text(aes(label = SGG_NM), 
               vjust = -0.7) +
  geom_sf_text(aes(label = comma(`인구 수(명)`)),
               vjust = 0.7) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-07-using-one-legend-across-multiple-maps/unnamed-chunk-3-1.png" width="100%" style="display: block; margin: auto;" alt="1995년, 2023년 서울 자치구별 인구 수 지도" />

## 방법 2: 색상 범위를 직접 고정하기

패싯을 나누지 않고 지도를 따로 저장해야 하는 경우도 있습니다. 이럴 때는
색상 범례의 범위를 직접 고정하면 됩니다. 핵심은 두 시점 전체 데이터를
기준으로 최소/최대값을 계산한 뒤, 그 값을
`scale_fill_gradient(limits = ...)`에 넣는 것입니다.

``` r
fill_limits <- range(joined_data$`인구 수(명)`, na.rm = TRUE)

fill_limits
```

    ## [1] 121312 666806

이제 어떤 지도에서도 같은 `fill_limits`를 사용하면, 지도마다 색이
의미하는 값 범위가 고정됩니다. 이를 이용해, 1995년 지도와 2023년 지도를 따로
그려보겠습니다.

``` r
# 1995년 지도 그리기
ggplot(data = joined_data %>% 
         filter(시점 == "1995"), 
       aes(fill = `인구 수(명)`)) +
  geom_sf(color="gray20") +
  scale_fill_gradient(low = '#ffffff', 
                      high = '#ff6666',
                      limits = fill_limits,
                      labels = label_comma()) +
  geom_sf_text(aes(label = SGG_NM), 
               vjust = -0.7) +
  geom_sf_text(aes(label = comma(`인구 수(명)`)),
               vjust = 0.7) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-07-using-one-legend-across-multiple-maps/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" alt="1995년 서울 자치구별 인구 수 지도" />

``` r
# 2023년 지도 그리기
ggplot(data = joined_data %>% 
         filter(시점 == "2023"), 
       aes(fill = `인구 수(명)`)) +
  geom_sf(color="gray20") +
  scale_fill_gradient(low = '#ffffff', 
                      high = '#ff6666',
                      limits = fill_limits,
                      labels = label_comma()) +
  geom_sf_text(aes(label = SGG_NM), 
               vjust = -0.7) +
  geom_sf_text(aes(label = comma(`인구 수(명)`)),
               vjust = 0.7) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-07-using-one-legend-across-multiple-maps/unnamed-chunk-6-1.png" width="100%" style="display: block; margin: auto;" alt="2023년 서울 자치구별 인구 수 지도" />

## 지도 다듬기

글꼴을 설정해 지도의 가독성을 높여줍니다. 먼저, `font_add`를 사용해
글꼴을 추가하고 `showtext_auto`로 활성화합니다. 마지막으로, `theme`으로
범례와 스트립 텍스트(연도)의 글꼴, 크기, 위치 등을 조정해 줍니다.

``` r
# 패키지 로드
library(showtext)

# 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/KoPub Dotum Medium.ttf")
font_add("kopub_b", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/KoPub Dotum Bold.ttf")

showtext_auto()
showtext_opts(dpi=300)

theme.size = 10
geom.text.size = theme.size / .pt

# 지도 그리기
ggplot(data = joined_data, aes(fill = `인구 수(명)`)) +
  geom_sf(color="gray20") +
  scale_fill_gradient(low = '#ffffff', 
                      high = '#ff6666',
                      labels = label_comma()) +
  facet_wrap(~시점) +
  geom_sf_text(aes(label = SGG_NM), 
               vjust = -0.7,
               family = "kopub_b",
               size = geom.text.size) +
  geom_sf_text(aes(label = comma(`인구 수(명)`)),
               vjust = 0.7,
               family = "kopub_b",
               size = geom.text.size) +
  theme_void(base_family = "kopub", base_size = theme.size) +
  theme(
    strip.text = element_text(family = "kopub_b", size = theme.size * 1.5),
    legend.position = "bottom",
    legend.title = element_text(family = "kopub_b", vjust=1),
    legend.text = element_text(size=theme.size),
    legend.key.height = unit(theme.size, "pt"),
    legend.key.width = unit(theme.size * 5, "pt")
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-07-using-one-legend-across-multiple-maps/unnamed-chunk-7-1.png" width="100%" style="display: block; margin: auto;" alt="1995년, 2023년 서울 자치구별 인구 수 지도" />

여러 지도를 비교할 때는 ’색이 의미하는 값의 범위를 지도마다 같게
맞춘다’는 원칙을 지키는 것이 중요합니다. 시점이 다른 지도뿐 아니라, 서로
다른 지역(예: 서울 vs 경기)을 나란히 놓고 비교할 때도 같은 방식으로 공통
범례를 적용하면 색만으로 값의 크기를 직관적으로 비교할 수 있습니다.