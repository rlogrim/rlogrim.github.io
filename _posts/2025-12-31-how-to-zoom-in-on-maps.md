---
title: 지도 확대하기
author: Rvinci
description: 전체 지도가 아니라 원하는 지역만 확대하고 싶은 적이 있었나요? 지도를 확대하는 두 가지 방법을 알려드립니다.
excerpt: 전체 지도가 아니라 원하는 지역만 확대하고 싶은 적이 있었나요? 지도를 확대하는 두 가지 방법을 알려드립니다.
categories: [map]
tags: [r, ggplot2, sf, st_bbox, st_buffer, coord_sf]
published: true
---

지도를 그리다 보면 특정 지역만  
더 자세히 보고 싶을 때가 있습니다.

예를 들어,
- 수도권 전체 지도에서 서울만 보여주고 싶을 때
- 서울 안에서도 광화문처럼 한 지점에 초점을 맞추고 싶을 때처럼요

이럴 때 범위를 지정해서 지도를 확대하는 2가지 방법이 있습니다.

## 폴리곤 경계를 기준으로 지도 확대하기

이전 [지도 위에 공간 정보 쌓아보기](https://rlogrim.github.io/map/layering-spatial-data/) 포스팅에서 만든  
수도권 지도를 활용하겠습니다.

먼저 수도권 지도에서 서울의 경계 좌표를 가져옵니다.  
`st_bbox()`를 쓰면 폴리곤의 최소, 최대 좌표(xmin, xmax, ymin, ymax)를  
한 번에 얻을 수 있습니다.

``` r
# 서울 경계 좌표 구하기
coords_seoul <- sgg %>% 
  filter(grepl("서울특별시", SGG_NM)) %>% 
  st_bbox()

coords_seoul
```

    ##     xmin     ymin     xmax     ymax 
    ## 179191.4 536562.8 216242.3 566863.5

이제 이 좌표를 활용할 차례입니다.

앞으로 지도를 여러 번 그릴 예정이니,  
기본이 되는 지도를 `base_map`이라는 이름으로 한 번 만들어두겠습니다.  
그리고 `coord_sf()`에 방금 구한 서울 경계 좌표를 넣으면,  
서울을 중심으로 한 확대 지도가 완성됩니다.

``` r
# 기본 지도 정의
base_map <- ggplot() +
  geom_sf(data = sgg, colour=NA, fill="#eaeaea") + # 시군구 영역
  geom_sf(data = green, colour=NA, fill="#A0D097") + # 녹지
  geom_sf(data = mt, colour=NA, fill="#A0D097") + # 산지
  geom_sf(data = water, colour=NA, fill="#A5D1F2") + # 수계
  geom_sf(data = road, colour=NA, fill="white", linewidth=0.1) + # 도로
  geom_sf(data = sgg, colour="white", fill=NA, linewidth=1.3) + # 시군구 경계
  geom_sf(data = sgg, colour="gray40", fill=NA, linewidth=0.5) # 시군구 경계

# 서울 중심으로 확대된 지도 그리기
base_map +
  coord_sf(xlim = coords_seoul[c("xmin", "xmax")],
           ylim = coords_seoul[c("ymin", "ymax")]) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2025-12-31-how-to-zoom-in-on-maps/unnamed-chunk-2-1.png" width="100%" style="display: block; margin: auto;" alt="수도권에서 서울을 확대한 지도" />

## 경계에 여유를 두고 싶다면?

서울 경계에 정확히 맞춘 지도는  
조금 답답해 보일 수 있습니다.  
이럴 땐 경계에 여유를 조금 주는 게 좋습니다.

`st_buffer()`를 이용해 경계를 기준으로  
바깥쪽 여백을 만들 수 있습니다.  
여기서는 서울 경계에 3km 정도 여유를 줘보겠습니다.

``` r
# 3km 버퍼를 둔 경계 좌표 구하기
buffered_coords_seoul <- sgg %>% 
  filter(grepl("서울특별시", SGG_NM)) %>% 
  st_buffer(dist = 3000) %>% # 3km 버퍼
  st_bbox()

# 버퍼를 적용한 확대 지도 그리기
base_map +
  coord_sf(xlim = buffered_coords_seoul[c("xmin", "xmax")],
           ylim = buffered_coords_seoul[c("ymin", "ymax")]) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2025-12-31-how-to-zoom-in-on-maps/unnamed-chunk-3-1.png" width="100%" style="display: block; margin: auto;" alt = "경계에 버퍼를 준 서울 지도" />

## 좌표를 직접 지정해서 확대하기

이번에는 수동으로 범위를 지정하는 방법을 살펴볼까요?  

Google 지도에서 광화문을 검색한 후  
확대해서 보고 영역의 꼭지점을 두 번 클릭하면  
경도, 위도 좌표를 확인할 수 있습니다.

한 가지 주의할 점은,  
Google 지도 좌표는 EPSG:4326(WGS84)를 쓰기 때문에  
우리가 쓰는 지도 좌표계인 EPSG:5186(GRS80)으로 변환이 필요하다는 점입니다.

네 꼭짓점 좌표를 만들고,  
좌표계를 변환한 뒤 `coord_sf()`에 넣어주면 끝입니다.

``` r
# 광화문 영역 좌표 정의 및 변환
x <- c(126.951940, 127.000950)
y <- c(37.566511, 37.586238)

pts <- st_multipoint(matrix(c(x[c(1, 2, 2, 1)],
                              y[c(1, 1, 2, 2)]), ncol = 2))

coords_ghm <- st_sfc(pts, crs = 4326) %>% 
  st_transform(crs = 5186) %>% 
  st_bbox()

# 광화문 인근 지역 지도 그리기
base_map +
  coord_sf(xlim = coords_ghm[c("xmin", "xmax")],
           ylim = coords_ghm[c("ymin", "ymax")]) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2025-12-31-how-to-zoom-in-on-maps/unnamed-chunk-4-1.png" width="100%" style="display: block; margin: auto;" alt="광화문 인근 지역 지도"/>

특정 지역만 확대하고 싶었다면,  
이 두 가지 방법이 충분히 도움이 될 거라고 생각합니다.

폴리곤 경계를 기준으로 범위를 잡는 방법은  
폴리곤이 있는 도시나 권역에 초점을 맞출 때 유용하고,  
좌표를 직접 지정하는 방법은  
폴리곤이 없는 지역을 대상으로 지도를 그릴 때 활용할 수 있습니다.