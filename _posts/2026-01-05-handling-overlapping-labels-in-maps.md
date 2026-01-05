---
title: 지도 지역명이 겹칠 때 해결하기
author: Rvinci
description: 지도 위에 지역명을 모두 표시하려다 텍스트가 겹쳐 난감했던 적 있나요? geom_text_repel()로 라벨을 분산 배치하는 방법과, 일부 지역명을 생략해 지도를 단순화하는 방법을 소개합니다.
excerpt: 지도 위에 지역명을 모두 표시하려다 텍스트가 겹쳐 난감했던 적 있나요? geom_text_repel()로 라벨을 분산 배치하는 방법과, 일부 지역명을 생략해 지도를 단순화하는 방법을 소개합니다.
categories: [map]
tags: [r, tidyverse, ggplot2, sf, ggrepel, geom_text_repel]
published: true
---

폴리곤 크기가 작은 지역들이 인접해 있는 경우, 여러 지역명이 한곳에
몰리면서 서로 겹치는 문제가 발생했습니다. 지도에서 텍스트가 겹치면
지역을 식별하기 어려워집니다.

이번 글에서는 지도에서 텍스트가 겹치는 문제를 해결하는 두 가지 방법을
살펴봅니다. 첫 번째 방법은 `geom_text_repel()`을 사용해 텍스트를
자동으로 분산 배치하는 방식입니다. 두 번째 방법은 특정 기준을 정해 일부
지역명을 생략하고, 지도를 더 단순하게 만드는 방식입니다.

## 기본 지도 그려 문제 상황 확인하기

먼저 지도를 그리기 위해 필요한 패키지를 불러옵니다. `tidyverse`는 데이터
가공과 정리, 지도 시각화에 사용하고, `sf`는 공간 데이터를 읽고 처리하는
데 사용합니다. `showtext`는 폰트를 설정하기 위해 사용합니다.

``` r
# 패키지 로드
library(tidyverse)
library(sf)
library(showtext)

# 글꼴 설정
font_add("kopub", "C:/Users/.../appdata/local/microsoft/windows/fonts/kopub dotum medium.ttf")
showtext_auto()
showtext_opts(dpi=300)

theme.size = 12
text.size = theme.size / .pt
```

다음으로 수도권 행정구 경계가 저장된 shp 파일을 불러옵니다. `st_read()`
함수는 공간 데이터 파일을 읽어 sf 객체로 변환합니다. 이렇게 불러온
데이터는 일반 데이터프레임처럼 다룰 수 있으면서도, 공간 정보를 함께
포함합니다.

이후 `separate()`를 사용해 시군구 이름이 들어 있는 `SGG_NM` 변수를
분리합니다. 원본 데이터에서는 시도명과 시군구명이 하나의 문자열로 들어
있어, 이후 작업에 불편함이 생길 수 있습니다. `sep = " "` 옵션을 사용해
공백을 기준으로 시도명과 시군구명을 나누고, 각각 `SD_NM`과 `SGG_NM`
변수로 저장합니다. `fill = "right"`는 이름이 하나만 있는 경우에도 오류
없이 처리하도록 설정합니다. 이 과정을 거치면 시도 단위와 시군구 단위를
각각 따로 사용할 수 있어, 이후 지도 시각화와 라벨 처리 작업이 훨씬
수월해집니다.

``` r
# 데이터 불러오기
sdg <- st_read("아웃풋/수도권 행정구 병합 지도.shp") %>% 
    separate(col = SGG_NM,
           into = c("SD_NM", "SGG_NM"),
           sep = " ",
           fill = "right")
```

    ## Reading layer `수도권 행정구 병합 지도' from data source 
    ##   `...\아웃풋\수도권 행정구 병합 지도.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 66 features and 2 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: -10044.95 ymin: 477264 xmax: 274945.2 ymax: 631207.8
    ## Projected CRS: KGD2002 / Central Belt 2010

``` r
# 지도 작성
ggplot(data = sdg) +
  geom_sf(colour = "gray40", fill = "#eaeaea", linewidth = 0.5) +
  geom_sf_text(aes(label = SGG_NM),
               family = "kopub",
               size = text.size) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-05-handling-overlapping-labels-in-maps/unnamed-chunk-2-1.png" width="100%" style="display: block; margin: auto;" alt="수도권 지도" />

이 결과를 보면, 서울과 경기 일부 소규모 지역이 밀집된 곳에서 지역명이
겹치는 것을 확인할 수 있습니다.

## `geom_text_repel()`로 지역명 분산 배치하기

첫 번째 방법은 `ggrepel` 패키지의 `geom_text_repel()`을 사용하는
방식입니다. 이 함수는 텍스트 라벨들이 서로 겹치지 않도록 자동으로 위치를
조정합니다. 지도처럼 좌표가 있는 sf 객체에 사용할 때는
`stat = "sf_coordinates"`를 지정해 좌표를 추출합니다. `geometry = geometry`를 지정해 각 폴리곤의 중심 좌표를 기준으로 텍스트를
배치합니다.

`geom_text_repel()`에는 텍스트 배치를 조절하는 여러 인자가 있습니다. 이
중에서 자주 사용하는 것이 `force`와 `force_pull`입니다.
- `force`는
텍스트 라벨들끼리 서로 밀어내는 힘의 크기를 의미합니다. 값이 클수록 라벨
사이 간격이 더 넓어지고, 값이 작을수록 라벨이 가까이 모입니다.
- `force_pull`은 라벨이 원래 위치, 즉 폴리곤 중심 좌표로 얼마나 강하게
끌리는지를 설정합니다. 값이 클수록 라벨이 원래 위치에 가까이 붙고, 값이
작을수록 중심에서 멀어질 수 있습니다.

이 방법은 텍스트 수가 많지 않을 때 효과적입니다. 수도권처럼 폴리곤이
많고 밀집된 지도에서는 모든 지역명을 깔끔하게 배치하기 어렵습니다.
`force`나 `force_pull` 값을 조정해도 한계가 있습니다.

``` r
# 패키지 로드
library(ggrepel)

# 지도 작성
ggplot(data = sdg) +
  geom_sf(colour = "gray40", fill = "#eaeaea", linewidth = 0.5) +
  geom_text_repel(aes(label = SGG_NM, geometry = geometry),
                  stat = "sf_coordinates",
                  family = "kopub",
                  size = text.size) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-05-handling-overlapping-labels-in-maps/unnamed-chunk-3-1.png" width="100%" style="display: block; margin: auto;" alt="지역명 분산 배치한 수도권 지도" />

## 일부 지역명 생략하기

두 번째 방법은 모든 지역명을 표시하려고 하지 않고, 일부 지역명을 과감히
생략하는 방식입니다. 예를 들어 폴리곤 면적이 작은 지역은 이름을 표시하지
않도록 설정할 수 있습니다.

먼저 각 지역의 면적을 계산합니다.

``` r
sdg <- sdg %>% 
  mutate(면적 = as.numeric(st_area(sdg)))
```

이제 면적이 중위값보다 작은 지역은 빈 문자열로 처리해 라벨이 표시되지
않습니다.

``` r
ggplot(data = sdg) +
  geom_sf(colour = "gray40", fill = "#eaeaea", linewidth = 0.5) +
  geom_sf_text(aes(label = ifelse(sdg$면적 < median(sdg$면적), "", SGG_NM)),
                  family = "kopub",
                  size = text.size) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-05-handling-overlapping-labels-in-maps/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" alt="지역명 일부 생략한 수도권 지도" />

이번 글에서는 지도에서 지역명이 겹치는 문제를 해결하는 두 가지 방법을
살펴봤습니다. `geom_text_repel()`은 텍스트를 자동으로 분산 배치해
주지만, 복잡한 지도에서는 한계가 있습니다. 한편, 일부 지역명을 생략하는
방법은 정보를 줄이는 대신 지도를 깔끔하게 만들어줍니다.

지도에서 어떤 정보가 가장 중요한지 먼저 정하고, 그에 맞는 방법을 선택해
보세요!