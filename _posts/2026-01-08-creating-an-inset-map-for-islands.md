---
title: 도서 지역 박스 처리하기
author: Rvinci
description: 도서 지역이 함께 포함되면서 정작 보여주고 싶은 지역이 작아져 난감했던 적이 있지 않나요? 이번 글에서는 메인 지도와 도서 지역 지도를 따로 그린 뒤, 도서 지역을 박스 형태로 메인 지도 위에 얹는 방법을 소개합니다. 이렇게 하면 지역을 누락하지 않으면서도 메인 지역이 화면에서 충분히 크게 보이도록 구성할 수 있습니다.
excerpt: 도서 지역이 함께 포함되면서 정작 보여주고 싶은 지역이 작아져 난감했던 적이 있지 않나요? 이번 글에서는 메인 지도와 도서 지역 지도를 따로 그린 뒤, 도서 지역을 박스 형태로 메인 지도 위에 얹는 방법을 소개합니다. 이렇게 하면 지역을 누락하지 않으면서도 메인 지역이 화면에서 충분히 크게 보이도록 구성할 수 있습니다.
categories: [map]
tags: [r, tidyverse, sf, cowplot, ggdraw, draw_plot]
published: true
---

지도를 그릴 때 서로 멀리 떨어진 지역이 함께 있으면, 정작 집중해서 보고 싶은 지역이 작아 보일 때가 있습니다. 수도권
지도를 예로 들면 옹진군처럼 육지에서 떨어진 섬 지역이 포함되면서, 실제로
강조하고 싶은 수도권 육지가 화면에서 차지하는 비중이 줄어들 수 있습니다. 
이럴 때는 멀리 떨어진 지역을 ’박스’로 따로 그려서 메인 지도 위에 얹으면
됩니다. 지역을 누락하지 않으면서도 중심 지역에 초점을 맞추는 지도를 만들 수 있습니다.

## 데이터 준비하기

여기서는 [여러 지역
병합하기](https://rlogrim.github.io/map/how-to-merge-regions/)에서 만든
수도권 지도를 사용하겠습니다. `st_read()`로 shp 파일을 읽어오고,
`head()`로 파일을 확인합니다.

``` r
# 패키지 로드
library(tidyverse)
library(sf)
library(cowplot)

# 데이터 불러오기
map <- st_read("아웃풋/수도권 행정구 병합 지도.shp")
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
# 데이터 확인하기
head(map)
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

## 박스에 넣을 옹진군 지도 그리기

박스에 들어갈 지역만 필터링해서 서브 지도를 만듭니다. `filter()` 함수로
`SGG_NM`이 ’인천광역시 옹진군’인 것만 선택합니다. 박스처럼 보이도록
테두리를 `panel.border`로 추가합니다.

``` r
sub_map <- map %>% 
  filter(SGG_NM == "인천광역시 옹진군") %>% 
  ggplot() +
  geom_sf(color = "gray40",
          fill = "#F7F7F7",
          linewidth = 0.4) +
  theme_void() +
  theme(
    panel.border = element_rect(color = "gray20",
                                fill = NA,
                                linewidth = 0.3),
    panel.background = element_rect(fill = "#A5D1F2")
  )

sub_map
```

<img src="{{ site.baseurl }}/assets/images/2026-01-08-creating-an-inset-map-for-islands/unnamed-chunk-2-1.png" width="100%" style="display: block; margin: auto;" alt="옹진군 지도" />

## 옹진군을 제외한 수도권 지도 그리기

이번에는 박스에 넣을 지역을 뺀 나머지 지역을 그립니다. 이렇게 하면
화면 범위가 중심 지역(여기서는 수도권 육지)에 맞춰져서 지도가 더 보기
좋아집니다. 서브 지도와 톤을 맞추기 위해 배경색과 선, 면 스타일도 같은
값을 사용합니다.

``` r
base_map <- map %>%
  filter(SGG_NM != "인천광역시 옹진군") %>% 
  ggplot() +
  geom_sf(color = "gray40",
          fill = "#F7F7F7",
          linewidth = 0.4) +
  theme_void() +
  theme(
    panel.background = element_rect(fill = "#A5D1F2")
  )

base_map
```

<img src="{{ site.baseurl }}/assets/images/2026-01-08-creating-an-inset-map-for-islands/unnamed-chunk-3-1.png" width="100%" style="display: block; margin: auto;" alt="옹진군을 제외한 수도권 지도" />

## 메인 지도 위에 박스 얹기

이제 `cowplot`으로 두 지도를 하나로 합칩니다. `ggdraw()`로 빈 캔버스를
만들고, `draw_plot()`으로 원하는 위치에 플롯을 얹습니다. `x`와 `y`는
위치, `width`는 인셋의 상대 크기입니다. 값은 데이터와 출력 비율에 따라
조금씩 달라지므로, 한두 번 조정하면서 맞추는 과정이 필요합니다.

``` r
ggdraw(base_map) +
  draw_plot(sub_map,
            x = 0.18, y = 0.37,
            width = 0.2)
```

<img src="{{ site.baseurl }}/assets/images/2026-01-08-creating-an-inset-map-for-islands/unnamed-chunk-4-1.png" width="100%" style="display: block; margin: auto;" alt="옹진군을 박스 처리한 수도권 지도" />

중심이 될 지도와 박스에 넣을 지도를 분리해서 그리고, 마지막에 한
화면으로 합치면 간단하게 완성됩니다. 옹진군처럼 도서 지역을 따로 박스로
표시해야 하는 경우에 이 방식을 적용해 보세요!