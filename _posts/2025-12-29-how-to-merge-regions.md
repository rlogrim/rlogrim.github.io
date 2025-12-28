---
title: 여러 지역 경계 병합하기
author: Rvinci
description: 여러 지역을 하나의 단위로 묶어 지도에 표현하고 싶다면 어떻게 해야 할까요? 지역 경계를 병합하는 방법을 소개합니다.
excerpt: 여러 지역을 하나의 단위로 묶어 지도에 표현하고 싶다면 어떻게 해야 할까요? 지역 경계를 병합하는 방법을 소개합니다.
categories: [map]
tags: [r, ggplot2, sf, group_by, summarise]
published: true
---

지도를 그리다 보면, 여러 개의 지역을 하나의 단위로 묶어 표현하고 싶을 때가 있습니다.

이런 경우에는 기존의 행정 경계를 그대로 쓰기보다  
경계를 병합해 새로 정리해야 합니다.

이번 글에서는 이런 상황의 한 예로  
행정구 경계를 시 단위로 병합하는 방법을 다뤄보겠습니다.

수원시나 성남시처럼 행정구가 설치된 도시를  
하나의 시 경계로 정리하는 과정을 차근차근 살펴보겠습니다.

## 행정구역 경계 데이터 받기

먼저 지도의 기본이 되는 행정구역 경계 데이터를 준비합니다.

[V-World 디지털트윈국토](https://www.vworld.kr)에서 ‘행정구역’을 검색하면  
[행정구역시군구_경계](https://www.vworld.kr/dtmk/dtmk_ntads_s002.do?searchKeyword=%ED%96%89%EC%A0%95%EA%B5%AC%EC%97%AD&searchOrganization=&searchBrmCode=&searchTagList=&searchFrm=&pageIndex=1&gidmCd=&gidsCd=&sortType=00&svcCde=MK&dsId=30604&listPageIndex=1) 데이터를 받을 수 있습니다.

이번 시간에는 서울, 인천, 경기를 포함한 수도권 데이터를 사용합니다.

## shp 파일 불러오기

shp 파일을 처리하기 위해 `sf` 패키지를 사용합니다.

`list.files()`로 폴더 안의 shp 파일 목록을 불러오고,  
`st_read()`로 읽은 뒤  
`bind_rows()`로 하나의 데이터로 합칩니다.

이렇게 하면 수도권 전체 행정구역이  
하나의 sf 객체로 정리됩니다.

``` r
# 패키지 로드
library(tidyverse)
library(sf)

# 데이터 불러오기
folder_path <- "데이터/수도권 지도/행정구역 경계"

shp_files <- list.files(folder_path, pattern = "*.shp$", full.names = TRUE)

merged_shp <- shp_files %>% 
  lapply(st_read) %>% 
  bind_rows()

# 데이터 확인
head(merged_shp)
```

    ## Simple feature collection with 6 features and 4 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 195102.6 ymin: 545229.7 xmax: 210189.8 ymax: 559196.5
    ## Projected CRS: Korea_2000_Korea_Central_Belt_2010
    ##   ADM_SECT_C              SGG_NM SGG_OID COL_ADM_SE
    ## 1      11110   서울특별시 종로구      11      11110
    ## 2      11140     서울특별시 중구      34      11140
    ## 3      11170   서울특별시 용산구       1      11170
    ## 4      11200   서울특별시 성동구       1      11200
    ## 5      11215   서울특별시 광진구      49      11215
    ## 6      11230 서울특별시 동대문구     232      11230
    ##                         geometry
    ## 1 MULTIPOLYGON (((197800 5590...
    ## 2 MULTIPOLYGON (((202072.4 55...
    ## 3 MULTIPOLYGON (((197569.6 55...
    ## 4 MULTIPOLYGON (((203845.4 55...
    ## 5 MULTIPOLYGON (((208984.4 55...
    ## 6 MULTIPOLYGON (((206279 5563...

## 행정구를 정리한 시군구 지도 그리기

이제 행정구를 시 단위로 묶을 차례입니다.

`SGG_NM`에는 '경기도 수원시 장안구'처럼 지역명이 들어 있습니다.  
`str_replace()`를 이용해 구 이름을 제거하고  
'경기도 수원시'까지만 남깁니다.  
이때 사용하는 정규표현식 `"(\\s\\w+)\\s\\w+$"`는  
문자열의 두 번째 공백 이전 단어를 찾는 역할을 합니다.  
'경기도 수원시 장안구'는 '경기도 수원시'로 바뀝니다.  

정규표현식을 사용하면  
이처럼 규칙이 있는 문자열을 한 번에 정리할 수 있어,  
일괄 처리할 때 특히 유용합니다.

이렇게 이름을 정리한 뒤 `group_by()`와 `summarise()`를 적용하면,  
같은 시 이름을 가진 경계들이 하나의 행정구역으로 병합됩니다.

``` r
# 패키지 로드
library(stringr)
 
# 행정구역 명칭의 두 번째 공백 뒤 단어를 없애기
result <- merged_shp %>%
  mutate(SGG_NM = str_replace(SGG_NM, "(\\s\\w+)\\s\\w+$", "\\1")) %>%
  group_by(SGG_NM, COL_ADM_SE) %>%
  summarise()

# 행정구역 확인
unique(result$SGG_NM)
```

    ##  [1] "경기도 가평군"       "경기도 고양시"       "경기도 과천시"      
    ##  [4] "경기도 광명시"       "경기도 광주시"       "경기도 구리시"      
    ##  [7] "경기도 군포시"       "경기도 김포시"       "경기도 남양주시"    
    ## [10] "경기도 동두천시"     "경기도 부천시"       "경기도 성남시"      
    ## [13] "경기도 수원시"       "경기도 시흥시"       "경기도 안산시"      
    ## [16] "경기도 안성시"       "경기도 안양시"       "경기도 양주시"      
    ## [19] "경기도 양평군"       "경기도 여주시"       "경기도 연천군"      
    ## [22] "경기도 오산시"       "경기도 용인시"       "경기도 의왕시"      
    ## [25] "경기도 의정부시"     "경기도 이천시"       "경기도 파주시"      
    ## [28] "경기도 평택시"       "경기도 포천시"       "경기도 하남시"      
    ## [31] "경기도 화성시"       "서울특별시 강남구"   "서울특별시 강동구"  
    ## [34] "서울특별시 강북구"   "서울특별시 강서구"   "서울특별시 관악구"  
    ## [37] "서울특별시 광진구"   "서울특별시 구로구"   "서울특별시 금천구"  
    ## [40] "서울특별시 노원구"   "서울특별시 도봉구"   "서울특별시 동대문구"
    ## [43] "서울특별시 동작구"   "서울특별시 마포구"   "서울특별시 서대문구"
    ## [46] "서울특별시 서초구"   "서울특별시 성동구"   "서울특별시 성북구"  
    ## [49] "서울특별시 송파구"   "서울특별시 양천구"   "서울특별시 영등포구"
    ## [52] "서울특별시 용산구"   "서울특별시 은평구"   "서울특별시 종로구"  
    ## [55] "서울특별시 중구"     "서울특별시 중랑구"   "인천광역시 강화군"  
    ## [58] "인천광역시 계양구"   "인천광역시 남동구"   "인천광역시 동구"    
    ## [61] "인천광역시 미추홀구" "인천광역시 부평구"   "인천광역시 서구"    
    ## [64] "인천광역시 연수구"   "인천광역시 옹진군"   "인천광역시 중구"

이제 병합된 데이터를 지도로 그려봅니다.

시군구 경계는 얇게, 시도 경계는 조금 더 굵게  
레이어를 나눠 표현하면 좋습니다.  
지도는 `ggplot()`과 `geom_sf()`로 그립니다.  
경계 선 색상은 `colour`, 면 색상은 `fill`, 선 굵기는 `linewidth`로 조정할 수 있습니다.

``` r
# 패키지 로드
library(ggplot2)

# 지도 작성
ggplot() +
  geom_sf(data = result, colour = "gray40", fill = NA, linewidth = 0.5) +
  geom_sf(data = result_sido, colour = "gray20", fill = NA, linewidth = 1) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2025-12-29-how-to-merge-regions/unnamed-chunk-4-1.png" width="100%" style="display: block; margin: auto;" alt="행정구가 병합된 수도권 지도"/>

## 지도를 shp 파일로 저장하기

마지막으로, 병합된 지도를 shp 파일로 저장해 두면  
다른 작업에서 바로 활용할 수 있어서 편리합니다.  
`st_write()` 함수를 지도를 저장할 수 있습니다.

한글이 깨지지 않도록 UTF-8 인코딩을 함께 지정합니다.  
또한, `append = FALSE`로 지정하여 기존 파일에 덮어쓰기할  
수 있도록 설정합니다.

``` r
st_write(result, 
         "아웃풋/수도권 행정구 병합 지도.shp",
         append = FALSE,
         layer_options = "ENCODING=UTF-8")
```

    ## Deleting layer `수도권 행정구 병합 지도' using driver `ESRI Shapefile'
    ## Writing layer `수도권 행정구 병합 지도' to data source 
    ##   `아웃풋/수도권 행정구 병합 지도.shp' using driver `ESRI Shapefile'
    ## options:        ENCODING=UTF-8 
    ## Writing 66 features with 2 fields and geometry type Unknown (any).