---
title: 지도에 지역명 표기하기
author: Rvinci
description: 지도 위에 지역 이름을 표시하고 싶었던 적 있으신가요? 이번 글에서는 ggplot2와 sf로 지역명을 자동으로 표시하는 방법부터, 중심점과 폴리곤 내부 점을 계산해 원하는 위치에 배치하는 방법까지 살펴봅니다.
excerpt: 지도 위에 지역 이름을 표시하고 싶었던 적 있으신가요? 이번 글에서는 ggplot2와 sf로 지역명을 자동으로 표시하는 방법부터, 중심점과 폴리곤 내부 점을 계산해 원하는 위치에 배치하는 방법까지 살펴봅니다.
categories: [map]
tags: [r, tidyverse, sf, geom_sf_text, st_centroid, st_point_on_surface]
published: true
---

이번 글에서는 `ggplot2`와 `sf`로 만든 지도에 ’시군구명’을 표시하는
방법을 정리합니다. 자동 배치로 지역명을 표시하는 법과 지역명을 놓을
좌표를 직접 만든 뒤 그 좌표에 텍스트를 배치하는 법을 알려드리겠습니다.

## 수도권 지도 준비하기

[여러 지역
병합하기](https://rlogrim.github.io/map/how-to-merge-regions/) 글에서
만든 수도권 shp 파일을 읽어 오고, 데이터에 어떤 열이 있는지 확인합니다.
아래 데이터를 보면, `SGG_NM` 열에 ’시도 시군구’가 합쳐진 문자열이 들어있습니다.

``` r
# 패키지 로드
library(tidyverse)
library(sf)

# 데이터 불러오기
sdg <- st_read("아웃풋/수도권 행정구 병합 지도.shp",
               quiet = TRUE)

# 데이터 확인
head(sdg)
```

    ## Simple feature collection with 6 features and 2 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 170923.2 ymin: 518463.7 xmax: 254296.8 ymax: 605780.4
    ## Projected CRS: KGD2002 / Central Belt 2010
    ##          SGG_NM COL_ADM_SE                       geometry
    ## 1 경기도 가평군      41820 MULTIPOLYGON (((239726.3 60...
    ## 2 경기도 고양시      41280 MULTIPOLYGON (((193799.4 57...
    ## 3 경기도 과천시      41290 MULTIPOLYGON (((200375.6 54...
    ## 4 경기도 광명시      41210 MULTIPOLYGON (((188355.1 54...
    ## 5 경기도 광주시      41610 MULTIPOLYGON (((230605 5485...
    ## 6 경기도 구리시      41310 MULTIPOLYGON (((213032.4 56...

지도에서 사용할 한글 폰트와 폰트 크기도 미리 설정해줍니다.

``` r
# 패키지 로드
library(showtext)

# 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/KoPub Dotum Medium.ttf")
showtext_auto()
showtext_opts(dpi=300)

theme.size = 12
text.size = theme.size / .pt
```

## 시도명과 시군구명 분리하기

공백으로 구분된 값이 들어 있는 `SGG_NM`을 `separate()`를 이용해 공백을
기준으로 두 열로 나눕니다. 원본 열 이름을 그대로 쓰면 나뉜 결과가 다시
`SGG_NM`에 들어가므로, 헷갈리지 않게 원본 열을 `SGG_FULL`로 한 번 바꾼 뒤
분리합니다.

``` r
sdg <- sdg %>% 
  rename(SGG_FULL = SGG_NM) %>%
  separate(col = SGG_FULL,
           into = c("SD_NM", "SGG_NM"),
           sep = " ",
           fill = "right",
           remove = TRUE)

head(sdg)
```

    ## Simple feature collection with 6 features and 3 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 170923.2 ymin: 518463.7 xmax: 254296.8 ymax: 605780.4
    ## Projected CRS: KGD2002 / Central Belt 2010
    ##    SD_NM SGG_NM COL_ADM_SE                       geometry
    ## 1 경기도 가평군      41820 MULTIPOLYGON (((239726.3 60...
    ## 2 경기도 고양시      41280 MULTIPOLYGON (((193799.4 57...
    ## 3 경기도 과천시      41290 MULTIPOLYGON (((200375.6 54...
    ## 4 경기도 광명시      41210 MULTIPOLYGON (((188355.1 54...
    ## 5 경기도 광주시      41610 MULTIPOLYGON (((230605 5485...
    ## 6 경기도 구리시      41310 MULTIPOLYGON (((213032.4 56...

## 방법 1: `geom_sf_text`로 자동 배치하기

가장 간단한 방법은 `geom_sf_text()`를 쓰는 것입니다. 폴리곤 지오메트리를
기준으로 텍스트를 적당한 위치에 배치합니다. 지역이 길쭉하거나 도넛
모양(구멍이 있는 폴리곤)인 경우에는 텍스트가 폴리곤 바깥에 놓이거나 서로
겹칠 수 있습니다.

지도 폴리곤을 먼저 그리고, 같은 데이터에서 `SGG_NM`을 라벨로 사용해
시군구명을 표시합니다.

``` r
ggplot(data = sdg) +
  geom_sf(colour = "gray40", 
          fill = "#eaeaea", 
          linewidth = 0.5) +
  geom_sf_text(aes(label = SGG_NM),
               family = "kopub",
               size = text.size) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/지역명-표기하기/unnamed-chunk-4-1.png" width="100%" style="display: block; margin: auto;" alt="자동으로 지역명을 배치한 수도권 지도" />

## 방법 2: 좌표 만들어 표시하기

### 폴리곤 중심점(`st_centroid`)에 표시하기

자동 배치 대신, 라벨을 배치할 좌표를 직접 만드는 방법도 있습니다.
`st_centroid()`로 중심점을 만든 뒤, 그 점 데이터에 `geom_sf_text()`를
적용해 텍스트를 표시할 수 있습니다.

아래 지도를 보면 옹진군의 중심점이 폴리곤 밖에 위치해, 지역명이 폴리곤 외부에 표시된 것을 확인할 수 있습니다.

``` r
# 포인트 추출
pnts1 <- st_centroid(sdg)

# 지도 작성
ggplot() +
  geom_sf(data = sdg, 
          colour = "gray40", 
          fill = "#eaeaea", 
          linewidth = 0.5) +
  geom_sf_text(data = pnts1,
            aes(label = SGG_NM),
            family = "kopub",
            size = text.size) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/지역명-표기하기/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" alt="중심점에 지역명 표기한 수도권 지도" />

### 폴리곤 내부(`st_point_on_surface`)에 표시하기

라벨이 ’반드시 폴리곤 내부’에 있어야 한다면 `st_point_on_surface()`가 더
적합합니다. 이 함수는 각 폴리곤에 대해 내부에 위치하는 대표 점을 계산해
줍니다. 중심점을 사용할 때보다 ’텍스트가 경계 밖으로 나가는 문제’를
줄이는 데 도움이 됩니다.

``` r
# 포인트 추출
pnts2 <- st_point_on_surface(sdg)

# 지도 작성
ggplot() +
  geom_sf(data = sdg, 
          colour = "gray40", 
          fill = "#eaeaea", 
          linewidth = 0.5) +
  geom_sf_text(data = pnts2,
            aes(label = SGG_NM),
            family = "kopub",
            size = text.size) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/지역명-표기하기/unnamed-chunk-6-1.png" width="100%" style="display: block; margin: auto;" alt="폴리곤 내부에 지역명 표기한 수도권 지도" />

`ggplot2`와 `sf`로 지역명을 표시하는 방법을 살펴봤습니다.
`geom_sf_text()`로 라벨을 자동으로 배치하는 방법과 `st_centroid()`,
`st_point_on_surface()`로 좌표를 계산해 원하는 위치에 텍스트를
배치하는 방법을 함께 다뤘습니다. 이 방법들을 활용해 지도에 라벨을 알맞게
배치해 보세요!

수도권 전체 시군구를 기준으로 보면 서울 자치구는 면적이 작아 라벨이 서로 겹치기 쉽습니다. 이런 겹침 문제를 줄이는 방법은 [지도 지역명이 겹칠 때 해결하기](https://rlogrim.github.io/map/handling-overlapping-labels-in-maps/)에서 이어서 다루겠습니다.