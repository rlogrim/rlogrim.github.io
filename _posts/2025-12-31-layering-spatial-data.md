---
title: 지도 위에 공간 정보 쌓아보기
author: Rvinci
description: 행정구역 지도 위에 여러 공간 데이터를 함께 그리고 싶었던 적 있나요? 행정구역 경계를 바탕으로 녹지, 수계, 도로 데이터를 불러와 정리하고, 레이어로 쌓아 하나의 지도를 완성하는 과정을 소개합니다.
excerpt: 행정구역 지도 위에 여러 공간 데이터를 함께 그리고 싶었던 적 있나요? 행정구역 경계를 바탕으로 녹지, 수계, 도로 데이터를 불러와 정리하고, 레이어로 쌓아 하나의 지도를 완성하는 과정을 소개합니다.
categories: [map]
tags: [r, ggplot2, sf, st_intersection, geom_sf]
published: true
---

행정구역 경계만 있는 지도는  
어딘가 밋밋하게 보일 때가 있습니다.

이번 글에서는 행정구역 경계를 기본으로 두고,  
산지, 녹지, 수계, 도로 같은 정보를 겹쳐  
조금 더 풍부한 정보를 담고 있는 지도를 만들어 보겠습니다.

먼저 사용할 데이터를 준비합니다.  
행정구역 경계, 산지, 녹지, 수계, 도로 데이터는 모두  
[V-World 디지털트윈국토](https://www.vworld.kr)에서 받을 수 있습니다.  

다행히 모든 데이터의 좌표계가 GRS80(EPSG: 5186)으로 동일해  
좌표계 변환 없이 바로 사용할 수 있습니다.  
그래서 좌표계를 맞추는 추가 작업은 필요 없습니다.

| 데이터        | 게시물                                                                                                                                                                                                                                                            | 좌표계           |
|------------------------|------------------------|------------------------|
| 행정구역 경계 | [행정구역시군구_경계](https://www.vworld.kr/dtmk/dtmk_ntads_s002.do?searchKeyword=%ED%96%89%EC%A0%95%EA%B5%AC%EC%97%AD&searchOrganization=&searchBrmCode=&searchTagList=&searchFrm=&pageIndex=1&gidmCd=&gidsCd=&sortType=00&svcCde=MK&dsId=30604&listPageIndex=1) | GRS80(EPSG:5186) |
| 산지          | [(연속주제)\_산지관리/보전준보전산지](https://www.vworld.kr/dtmk/dtmk_ntads_s002.do?svcCde=MK&dsId=30362)                                                                                                                                                         | GRS80(EPSG:5186) |
| 녹지          | [(연속주제)\_국토계획/공간시설](https://www.vworld.kr/dtmk/dtmk_ntads_s002.do?svcCde=MK&dsId=30289)                                                                                                                                                               | GRS80(EPSG:5186) |
| 수계          | [실폭하천](https://www.vworld.kr/dtmk/dtmk_ntads_s002.do?svcCde=MK&dsId=30207)                                                                                                                                                                                    | GRS80(EPSG:5186) |
| 도로          | [(도로명주소)실폭도로](https://www.vworld.kr/dtmk/dtmk_ntads_s002.do?svcCde=MK&dsId=30057)                                                                                                                                                                        | GRS80(EPSG:5186) |

## 여러 shp 파일, 한 번에 불러오기

다운로드한 shp 파일들을 R로 불러옵니다.  
행정구역 경계는 이전에 만들어둔  
[수도권 행정구 병합 지도.shp](https://rlogrim.github.io/map/how-to-merge-regions/) 파일을 사용합니다.

나머지 데이터는 폴더 안에 여러 shp 파일로 나뉘어 있으므로,  
`list.files()`로 한꺼번에 불러와 `bind_rows()`로 합쳐줍니다.
이렇게 하면 데이터 종류별로 하나의 sf 객체로 만들 수 있습니다.

``` r
# 패키지 로드
library(tidyverse)
library(sf)

# 행정구역 경계 지도 로드
sgg <- st_read("아웃풋/수도권 행정구 병합 지도.shp")

# 녹지 데이터 로드 및 병합
folder_path <- "데이터/수도권 지도/녹지"

shp_files <- list.files(folder_path, pattern = "*.shp$", full.names = TRUE)

green <- shp_files %>% 
  lapply(st_read) %>% 
  bind_rows()

# 도로 데이터 로드 및 병합
folder_path <- "데이터/수도권 지도/도로"

shp_files <- list.files(folder_path, pattern = "*.shp$", full.names = TRUE)

road <- shp_files %>% 
  lapply(st_read) %>% 
  bind_rows()

# 산지 데이터 로드 및 병합
folder_path <- "데이터/수도권 지도/산지"

shp_files <- list.files(folder_path, pattern = "*.shp$", full.names = TRUE)

mt <- shp_files %>% 
  lapply(st_read) %>% 
  bind_rows()

# 수계 데이터 로드 및 병합
folder_path <- "데이터/수도권 지도/수계"

shp_files <- list.files(folder_path, pattern = "*.shp$", full.names = TRUE)

water <- shp_files %>% 
  lapply(st_read) %>% 
  bind_rows()
```

## 지도에 쓰기 좋게 shp 파일 다듬기

녹지 데이터는 '국토계획/공간시설' 자료에서  
필요한 부분만 골라 사용합니다.  
용도지역지구 표준분류코드를 기준으로 보면,  
광장(UQT100)은 녹지 성격과 거리가 있어 제외했습니다.

`MNUM`에서 필요한 부분만 잘라낸 뒤,  
조건에 맞지 않는 항목을 `filter()`로 걸러냅니다.

``` r
green <- green %>% 
  mutate(no = substr(MNUM, 21, 24)) %>% 
  filter(no != "UQT1")
```

산지와 수계 데이터는 전국 단위라 그대로 쓰기엔 너무 큽니다.  
그래서 `st_intersection()`을 이용해  
수도권 행정구역(sgg)과 겹치는 부분만 남깁니다.

``` r
st_crs(water) <- 5186
st_crs(mt) <- 5186

water <- st_intersection(water, sgg)
mt <- st_intersection(mt, sgg)
```

## 레이어를 차곡차곡 쌓아서 한 장의 지도로 만들기

이제 모든 재료가 준비됐습니다.  
`ggplot2`의 `geom_sf()`를 이용해 지도 레이어를 하나씩 쌓아봅니다.  
바탕에 행정구역 면, 그 위에 녹지, 산지, 수계, 도로,  
마지막으로 행정구역 경계선을 그려줍니다.

축과 배경은 `theme_void()`로 제거해  
지도 자체에만 시선이 가도록 합니다.

``` r
ggplot() +
  geom_sf(data = sgg, colour=NA, fill="#eaeaea") + # 시군구 영역
  geom_sf(data = green, colour=NA, fill="#A0D097") + # 녹지
  geom_sf(data = mt, colour=NA, fill="#A0D097") + # 산지
  geom_sf(data = water, colour=NA, fill="#A5D1F2") + # 수계
  geom_sf(data = road, colour=NA, fill="white", linewidth=0.1) + # 도로
  geom_sf(data = sgg, colour="white", fill=NA, linewidth=1.3) + # 시군구 경계
  geom_sf(data = sgg, colour="gray40", fill=NA, linewidth=0.5) + # 시군구 경계
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2025-12-31-layering-spatial-data/unnamed-chunk-4-1.png" width="100%" style="display: block; margin: auto;" alt="녹지, 산지, 수계, 도로를 표현한 수도권 지도" />

행정구역을 뼈대로 삼고,  
그 위에 어떤 공간 정보든 자유롭게 얹을 수 있습니다.

예를 들어,
- 인구, 산업, 시설 밀도 등 점 데이터
- 도로, 철도, 하천 등 선 데이터
- 개발제한구역, 용도지역 등 면 데이터

이 방법을 활용해서 목적에 맞는 다양한 지도를 만들어 보세요.