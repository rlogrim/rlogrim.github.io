---
title: 주소를 좌표로 변환해 지도에 표시하기
author: Rvinci
description: 주소 목록은 있는데 지도에 어떻게 표시해야 할지 막막하신가요? 무료 지오코딩 서비스를 이용해 주소를 위경도 좌표로 바꾼 뒤, 지도 위에 위치와 이름을 함께 표시하는 과정을 알려드립니다.
excerpt: 주소 목록은 있는데 지도에 어떻게 표시해야 할지 막막하신가요? 무료 지오코딩 서비스를 이용해 주소를 위경도 좌표로 바꾼 뒤, 지도 위에 위치와 이름을 함께 표시하는 과정을 알려드립니다.
categories: [map]
tags: [r, tidyverse, sf, geocoding]
published: true
---

주소 데이터만 있어도 ’좌표(위도, 경도)’로 바꾸면 지도 위에 위치를 표시할
수 있습니다. 여기서는 서울에 있는 고등교육기관 주소 데이터를
지오코딩으로 변환한 뒤, 서울 지도에 점과 라벨로 표시하는 과정을
보여드리겠습니다.

## 데이터 준비하기

이번 예시는 [교육통계서비스](https://kess.kedi.re.kr/)의 알림·서비스 \>
자료실에서 내려받을 수 있는 고등교육기관 주소록 데이터를 사용합니다. 이
데이터에는 학교명과 주소가 포함되어 있습니다. 또한 [이전
글](https://rlogrim.github.io/map/how-to-merge-regions/)에서 만든 수도권
지도를 불러와, 그중 서울만 골라 배경 지도로 사용하겠습니다.

먼저, 필요한 패키지를 불러오고 지도에 사용할 한글 폰트와 폰트 크기를
설정해줍니다.

``` r
# 패키지 로드
library(tidyverse)
library(sf)
library(showtext)
library(ggrepel)
library(readxl)
library(writexl)

# 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/KoPub Dotum Medium.ttf")
showtext_auto()
showtext_opts(dpi=300)

theme.size = 8
text.size = theme.size / .pt
```

수도권 지도와 고등교육기관 주소록 데이터를 불러옵니다.

``` r
# 데이터 불러오기
sdg <- st_read("아웃풋/수도권 행정구 병합 지도.shp",
               quiet = TRUE) %>% 
  rename(SGG_FULL = SGG_NM) %>% 
  separate(col = SGG_FULL,
           into = c("SD_NM", "SGG_NM"),
           sep = " ",
           fill = "right",
           remove = TRUE)

# 고등교육기관 주소록 데이터 불러오기
univ <- read_xlsx("데이터/고등교육기관 주소록(2024.4.1.).xlsx", 
                  skip = 5)

# 데이터 확인하기
head(univ)
```

    ## # A tibble: 6 × 15
    ##   연도  학교종류 시도  행정구 학교명 `학교명(영문)` KEDI학교코드 본분교 학교상태
    ##   <chr> <chr>    <chr> <chr>  <chr>  <chr>          <chr>        <chr>  <chr>   
    ## 1 2024  대학교   강원  강원 원주… 국립강릉원… Gangneung-Won… 51001000     본교(제2… 학교명변경……
    ## 2 2024  대학교   강원  강원 강릉… 국립강릉원… Gangneung-Won… 51001000     본교(제1… 학교명변경……
    ## 3 2024  일반대학원…… 강원  강원 원주… 국립강릉원… Gangneung-Won… 51001600     본교(제2… 학교명변경……
    ## 4 2024  일반대학원…… 강원  강원 강릉… 국립강릉원… Gangneung-Won… 51001600     본교(제1… 학교명변경……
    ## 5 2024  특수대학원…… 강원  강원 원주… 국립강릉원… Gangneung Won… 51001607     본교(제2… 학교명변경……
    ## 6 2024  특수대학원…… 강원  강원 강릉… 국립강릉원… Gangneung Won… 51001607     본교(제1… 학교명변경……
    ## # ℹ 6 more variables: 설립 <chr>, 우편번호 <chr>, 주소 <chr>, 전화번호 <chr>,
    ## #   팩스번호 <chr>, 홈페이지 <chr>

이 글에서는 서울만 필요하므로, `filter()`를 이용해 서울에 해당하는
데이터만 추립니다.

``` r
# 서울 지도만 추출하기
sdg_seoul <- sdg %>% 
    filter(SD_NM == "서울특별시")

# 서울 고등교육기관 데이터 추출하기
univ_seoul <- univ %>% 
  filter(시도 == "서울")

# 서울 고등교육기관 데이터 저장하기
write_xlsx(univ_seoul, 
           "아웃풋/서울 고등교육기관 주소록.xlsx")

# 데이터 확인하기
head(univ_seoul)
```

    ## # A tibble: 6 × 15
    ##   연도  학교종류 시도  행정구 학교명 `학교명(영문)` KEDI학교코드 본분교 학교상태
    ##   <chr> <chr>    <chr> <chr>  <chr>  <chr>          <chr>        <chr>  <chr>   
    ## 1 2024  대학교   서울  서울 종로… 서울대학교… Seoul Nationa… 51012000     본교(제2… 기존    
    ## 2 2024  대학교   서울  서울 관악… 서울대학교… Seoul Nationa… 51012000     본교(제1… 기존    
    ## 3 2024  일반대학원…… 서울  서울 종로… 서울대학교… Graduate Scho… 51012600     본교(제2… 기존    
    ## 4 2024  일반대학원…… 서울  서울 관악… 서울대학교… Graduate Scho… 51012600     본교(제1… 기존    
    ## 5 2024  전문대학원…… 서울  서울 관악… 서울대학교… Seoul Nationa… 51012628     본교(제1… 기존    
    ## 6 2024  전문대학원…… 서울  서울 관악… 서울대학교… Seoul Nationa… 51012661     본교(제1… 기존    
    ## # ℹ 6 more variables: 설립 <chr>, 우편번호 <chr>, 주소 <chr>, 전화번호 <chr>,
    ## #   팩스번호 <chr>, 홈페이지 <chr>

## 지오코딩으로 주소를 좌표로 변환하기

주소를 위도(latitude)와 경도(longitude)로 바꾸는 과정을
지오코딩(geocoding)이라고 합니다. R에서도 지오코딩을 할 수 있지만,
서비스에 따라 API 키가 필요하거나 유료 과금이 붙는 경우가 있어 처음
시도할 때 부담이 될 수 있습니다. 무료로 사용할 수 있는 [지오서비스
웹](https://www.geoservice.co.kr/)을 이용해 좌표로 변환하겠습니다.

지오서비스 웹은 아래 순서로 사용할 수 있습니다.

1.  로그인 후 아카이브에 ’서울 고등교육기관 주소록.xlsx\` 파일을
    업로드합니다.
2.  상단 메뉴에서 변환 \> 지오코딩 \> 주소를 좌표로 변환을 클릭합니다.
3.  업로드한 파일을 선택하고, 주소 필드에 주소 열을 지정합니다.
4.  변환이 완료된 파일을 다운로드합니다.

이렇게 다운로드한 파일은 shp 파일이므로, `st_read()`로 불러옵니다.

``` r
# 지오코딩된 데이터 불러오기
univ <- st_read("데이터/서울 고등교육기관 지오코딩/a.shp",
                quiet = TRUE)

# 데이터 확인하기
head(univ)
```

    ## Simple feature collection with 6 features and 17 fields
    ## Geometry type: POINT
    ## Dimension:     XY
    ## Bounding box:  xmin: 126.9524 ymin: 37.4674 xmax: 127.0781 ymax: 37.63234
    ## Geodetic CRS:  WGS 84
    ##   field1     field2 field3      field4                        field5
    ## 1   2024     대학교   서울 서울 종로구                    서울대학교
    ## 2   2024 일반대학원   서울 서울 종로구         서울대학교 일반대학원
    ## 3   2024     대학교   서울 서울 관악구                    서울대학교
    ## 4   2024 전문대학원   서울 서울 종로구       서울대학교 치의학대학원
    ## 5   2024 일반대학원   서울 서울 관악구             서울대학교 대학원
    ## 6   2024 일반대학원   서울 서울 노원구 서울과학기술대학교 일반대학원
    ##                                          field6   field7          field8 field9
    ## 1                     Seoul National University 51012000 본교(제2캠퍼스)   기존
    ## 2  Graduate School of Seoul National University 51012600 본교(제2캠퍼스)   기존
    ## 3                     Seoul National University 51012000 본교(제1캠퍼스)   기존
    ## 4 Seoul National University School of Dentistry 51012B54 본교(제2캠퍼스)   기존
    ## 5  Graduate School of Seoul National University 51012600 본교(제1캠퍼스)   기존
    ## 6                               Graduate School 51027600 본교(제1캠퍼스)   기존
    ##      field10 field11                                                   field12
    ## 1 국립대법인    3080                                    서울 종로구 대학로 103
    ## 2 국립대법인    3080                                    서울 종로구 대학로 103
    ## 3 국립대법인    8826           서울특별시 관악구 관악로 1 (신림동, 서울대학교)
    ## 4 국립대법인    3080                                    서울 종로구 대학로 103
    ## 5 국립대법인    8826           서울특별시 관악구 관악로 1 (신림동, 서울대학교)
    ## 6       국립    1811 서울특별시 노원구 공릉로 232 (공릉동, 서울과학기술대학교)
    ##       field13     field14
    ## 1 02-880-5114 02-885-5272
    ## 2 02-880-5114 02-885-5272
    ## 3 02-880-5114 02-885-5272
    ## 4 02-740-8611 02-745-1906
    ## 5 02-880-5114 02-885-5272
    ## 6 02-970-6114 02-970-6800
    ##                                                     field15 X_GC_TYPE
    ## 1                                             www.snu.ac.kr        정
    ## 2 https://www.snu.ac.kr/academics/graduate/graduate_schools        정
    ## 3                                             www.snu.ac.kr        정
    ## 4                                       dentistry.snu.ac.kr        정
    ## 5 https://www.snu.ac.kr/academics/graduate/graduate_schools        정
    ## 6                                       www.seoultech.ac.kr        정
    ##                             X_CLEANADDR                  geometry
    ## 1 서울특별시 종로구 대학로 103 (연건동) POINT (126.9996 37.58097)
    ## 2 서울특별시 종로구 대학로 103 (연건동) POINT (126.9996 37.58097)
    ## 3   서울특별시 관악구 관악로 1 (신림동)  POINT (126.9524 37.4674)
    ## 4 서울특별시 종로구 대학로 103 (연건동) POINT (126.9996 37.58097)
    ## 5   서울특별시 관악구 관악로 1 (신림동)  POINT (126.9524 37.4674)
    ## 6 서울특별시 노원구 공릉로 232 (공릉동) POINT (127.0781 37.63234)

500개 이상의 데이터를 지도에 한 번에 보여주기엔 정보가 너무 많아,
`filter()`로 대학교만 남기겠습니다. 이렇게 정리하면, 총 45개 학교가 남습니다.

``` r
# 대학교 데이터 추출 및 정리하기
univ_tmp <- univ %>% 
  filter(field2 == "대학교") %>% 
  select(field5, geometry) %>% 
  rename(학교명 = field5) %>% 
  group_by(학교명) %>% 
  filter(row_number() == 1)

# 데이터 확인하기
head(univ_tmp)
```

    ## Simple feature collection with 6 features and 1 field
    ## Geometry type: POINT
    ## Dimension:     XY
    ## Bounding box:  xmin: 126.9996 ymin: 37.50174 xmax: 127.1327 ymax: 37.63234
    ## Geodetic CRS:  WGS 84
    ## # A tibble: 6 × 2
    ## # Groups:   학교명 [6]
    ##   학교명                        geometry
    ##   <chr>                      <POINT [°]>
    ## 1 서울대학교         (126.9996 37.58097)
    ## 2 한국체육대학교     (127.1327 37.52127)
    ## 3 가톨릭대학교       (127.0047 37.50174)
    ## 4 건국대학교         (127.0788 37.54164)
    ## 5 서울과학기술대학교 (127.0781 37.63234)
    ## 6 서울시립대학교     (127.0556 37.58431)

## 지도에 위치와 학교명 표시하기

sf 객체끼리 겹쳐 그릴 때는 좌표계(CRS)가 같아야 위치가 정확히 맞습니다.
서울 지도(`sdg_seoul`)의 좌표계를 기준으로, 지오코딩 결과(`univ_tmp`)를
같은 좌표계로 변환한 뒤 시각화하겠습니다. `st_transform()`을 이용하면
좌표계를 변환할 수 있습니다.

``` r
# 좌표계 변환하기
univ_final <- univ_tmp %>% 
  st_transform(crs = st_crs(sdg_seoul))
```

이제 `geom_sf()`로 서울 폴리곤을 먼저 그리고, 같은 좌표계로 맞춘 학교
데이터를 점으로 표시합니다. 점을 `shape = 21`로 두면 외곽선(`color`)과
내부 색(`fill`)을 나눠서 지정할 수 있습니다. 학교명은 겹칠 수 있으므로,
라벨은 `geom_text_repel()`로 서로 밀어내며 배치하도록 설정합니다.

``` r
# 지도 작성하기
ggplot() +
  geom_sf(data = sdg_seoul, 
          color = "gray40", 
          fill = "#eaeaea", 
          linewidth = 0.5) +
  geom_sf(data = univ_final, 
          shape = 21,
          size = 4,
          stroke = 0.1,
          color = "white", 
          fill = "#00D353") +
  geom_text_repel(data = univ_final, 
                  aes(label = 학교명, geometry = geometry),
                  stat = "sf_coordinates",
                  family = "kopub",
                  size = text.size,
                  force = 0.001,
                  force_pull = 1000,
                  color = "white",
                  bg.color = "black",
                  bg.r = 0.1,
                  segment.color = "gray") +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-09-geocoding-addresses-and-mapping-them/unnamed-chunk-7-1.png" width="100%" style="display: block; margin: auto;" alt="서울 소재 대학교 지도" />

이렇게 했을 때도 라벨이 많이 겹친다면, 텍스트 자체를 짧게 만들어서 충돌
가능성을 줄일 수 있습니다. 학교명에서 ’대학교’를 제거해 라벨 길이를
줄이고, `max.overlaps = Inf`로 설정해 겹치더라도 라벨이 생략되지 않도록
합니다.

``` r
# 학교명에서 대학교 제거하기
univ_cnt <- univ_final %>% 
  mutate(학교명 = sub("대학교", "", 학교명))

# 지도 작성하기
ggplot() +
  geom_sf(data = sdg_seoul, 
          color = "gray40", 
          fill = "#eaeaea", 
          linewidth = 0.5) +
  geom_sf(data = univ_final, 
          shape = 21,
          size = 4,
          stroke = 0.1,
          color = "white", 
          fill = "#00D353") +
  geom_text_repel(data = univ_cnt, 
                  aes(label = 학교명, geometry = geometry),
                  stat = "sf_coordinates",
                  family = "kopub",
                  size = text.size,
                  force = 0.001,
                  force_pull = 1000,
                  color = "white",
                  bg.color = "black",
                  bg.r = 0.1,
                  segment.color = "gray",
                  max.overlaps = Inf) +
  theme_void()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-09-geocoding-addresses-and-mapping-them/unnamed-chunk-8-1.png" width="100%" style="display: block; margin: auto;" alt="학교명을 간추린 서울 소재 대학교 지도" />

이번 글에서는 주소 데이터를 지오코딩으로 좌표로 바꾼 뒤, 서울 지도 위에
위치와 학교명을 표시하는 과정을 살펴봤습니다. 같은 방식으로 병원,
공공기관, 매장 주소처럼 ’주소만 있는 데이터’를 좌표로 바꿔 지도에 표시해
보세요!