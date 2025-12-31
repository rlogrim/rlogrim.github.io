---
title: 시계열 차트에서 날짜 다루기
author: Rvinci
description: 시계열 차트를 그리다 날짜 때문에 막힌 적 있나요? 문자형 날짜를 날짜 객체로 변환하는 방법부터, 날짜 축 설정, 라벨 정리, 일 단위 데이터를 월 단위로 요약하는 과정까지 차근차근 알아봅시다.
excerpt: 시계열 차트를 그리다 날짜 때문에 막힌 적 있나요? 문자형 날짜를 날짜 객체로 변환하는 방법부터, 날짜 축 설정, 라벨 정리, 일 단위 데이터를 월 단위로 요약하는 과정까지 차근차근 알아봅시다.
categories: [chart]
tags: [r, ggplot2, lubridate, ymd, scale_x_date]
published: true
---

R로 시계열 데이터를 그리다 보면,  
생각보다 자주 막히는 지점이 바로 날짜 처리입니다.

이번 글에서는 국가별 코로나19 확진자 수 데이터를 예제로 삼아,  
- 문자형 날짜를 날짜(Date) 객체로 변환하는 방법
- ggplot2에서 날짜를 x축으로 사용하는 방법
- 날짜 라벨을 읽기 좋게 다듬는 방법
- 일 단위 데이터를 월 단위로 정리하는 방법까지 한 번에 정리해봅니다.

데이터는 [Our World in Data](https://ourworldindata.org/covid-vaccinations?country=OWID_WRL)에서 제공하는  
코로나19 공개 데이터를 사용합니다.

## 날짜, 왜 이렇게 다루기 어려울까

날짜 데이터는 막상 다뤄보면  
생각보다 까다로운 점이 많습니다.

먼저, 형식이 제각각입니다.  
"2020-08-08", "2020/08/08", "20-08-08", "2020년 8월 8일"처럼 표현 방식이 제각기 다릅니다.

문자형으로 불러오면 순서가 깨집니다.  
날짜가 시간 순서가 아니라 문자열 순서로 정렬됩니다.

숫자처럼 계산할 수 없습니다.  
날짜는 단순한 숫자가 아니어서 `+`, `-` 연산이 바로 되지 않습니다.

이런 문제를 해결해 주는 패키지가 바로 `lubridate`입니다.  
자세한 내용은 [공식 문서](https://lubridate.tidyverse.org/reference/lubridate-package.html)를 참고해도 좋습니다.

## 데이터부터 준비합시다

이번에 사용할 데이터에는 다음과 같은 열이 들어 있습니다.
- `date`: 날짜(문자형)
- `locate`: 국가명
- `new_cases`: 해당 날짜의 신규 확진자 수

한국 데이터만 골라서 분석에 필요한 행만 남기겠습니다.

``` r
# 패키지 로드
library(tidyverse)
library(lubridate)

# 데이터 불러오기
data <- read.csv("데이터/owid-covid-data.csv")

# 한국 데이터 필터링 및 열 선택
data_korea <- data %>% 
  filter(location=="South Korea") %>% 
  select(date, new_cases)

# 데이터 구조 확인
str(data_korea)
```

    ## 'data.frame':    1034 obs. of  2 variables:
    ##  $ date     : chr  "2020-01-22" "2020-01-23" "2020-01-24" "2020-01-25" ...
    ##  $ new_cases: num  NA 0 1 0 1 1 0 0 0 7 ...

이 상태에서 `date`는 아직 문자형(character)입니다.  
그래프의 x축으로 쓰기 전에,  
날짜 전용 클래스인 `Date`로 변화해줘야 합니다.  
`ymd()`는 "2020-01-22"처럼 연-월-일 형태의 문자열을  
자동으로 인식해 날짜 객체로 바꿔줍니다.

``` r
# 날짜 변환
data_korea <- data_korea %>% 
  mutate(date=ymd(date))

# 데이터 요약
summary(data_korea)
```

    ##       date              new_cases     
    ##  Min.   :2020-01-22   Min.   :     0  
    ##  1st Qu.:2020-10-06   1st Qu.:   168  
    ##  Median :2021-06-21   Median :   967  
    ##  Mean   :2021-06-21   Mean   : 25733  
    ##  3rd Qu.:2022-03-06   3rd Qu.: 13004  
    ##  Max.   :2022-11-20   Max.   :621317  
    ##                       NA's   :1

## x축에 날짜를 배치하기

날짜를 x축에 쓰려면 `scale_x_date()`를 사용합니다.  
`breaks`로 x축 어디에 눈금을 찍을지,  
`date_labels`로 날짜를 어떤 형식으로 보여줄지 정할 수 있습니다.
아래 예시는 3개월 간격으로 날짜를 표시하고,  
라벨은 년-월 형식으로 보여주는 코드입니다.

``` r
ggplot(data=data_korea, aes(x=date, y=new_cases)) +
  geom_col() +
  scale_x_date(name="", 
               breaks=seq(ymd('2020-01-20'),ymd('2022-11-20'), by='3 month'), 
               date_labels="%Y-%m"
               ) +
  scale_y_continuous(name="신규 확진자 수(명)") +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-01-handling-dates-in-time-series-charts/unnamed-chunk-3-1.png" width="70%" style="display: block; margin: auto;" alt="한국의 일별 코로나 신규 확진자 수 그래프" />

## 날짜 라벨이 겹칠 땐

날짜가 길거나 간격이 촘촘하면  
라벨이 서로 겹쳐서 읽기 어려울 때가 있습니다.  
이럴 때는 줄 바꿈이나 회전으로 해결할 수 있습니다.

### 1. 줄 바꿈하기

`labels`에 다음과 같이 함수를 넘기면,  
날짜 문자열을 원하는 형태로 직접 바꿀 수 있습니다.
아래 코드는 연도 뒤에서 줄 바꿈을 해줍니다.

``` r
ggplot(data=data_korea, aes(x=date, y=new_cases)) +
  geom_col() +
  scale_x_date(name="",
              breaks=seq(ymd('2020-01-20'),ymd('2022-11-20'), by='3 month'),
              labels=function(x) sub("-", "-\n", as.character(x)) # 년도 다음 대쉬 뒤 줄 바꿈
              ) + 
  scale_y_continuous(name="신규 확진자 수(명)") +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-01-handling-dates-in-time-series-charts/unnamed-chunk-4-1.png" width="70%" style="display: block; margin: auto;" alt="x축 텍스트 줄 바꾼 그래프" />

### 2. 라벨 회전하기

라벨을 기울이는 것도 자주 쓰는 방법입니다.
`angle`로 회전 각도를 설정하고,  
`hjust`, `vjust`로 글자 정렬 위치를 조정합니다.

``` r
ggplot(data=data_korea, aes(x=date, y=new_cases)) +
  geom_col() +
  scale_x_date(name="",
              breaks=seq(ymd('2020-01-20'),ymd('2022-11-20'), by='3 month'),
              ) + 
  scale_y_continuous(name="신규 확진자 수(명)") +
  theme_bw() +
  theme(
    axis.text.x=element_text(angle=45, hjust=1, vjust=1)
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-01-handling-dates-in-time-series-charts/unnamed-chunk-5-1.png" width="70%" style="display: block; margin: auto;" alt="x축 텍스트 45도 회전한 그래프" />

## 일별 데이터, 월별로 정리하기

일별 데이터가 너무 많다면,  
월 단위로 요약해서 보는 편이 더 좋을 때도 많습니다.

`date`에서 년-월 문자열을 생성하고,  
이를 기준으로 그룹화하여 월별 신규 확진자 수를 합산합니다.

``` r
# 년월 열 생성 및 집계
data_korea_monthly <- data_korea %>%
  mutate(년월=format(date, "%Y-%m")) %>%
  group_by(년월) %>%
  summarise(new_cases=sum(new_cases, na.rm = TRUE))

# 데이터 확인
head(data_korea_monthly)
```

    ## # A tibble: 6 × 2
    ##   년월    new_cases
    ##   <chr>       <dbl>
    ## 1 2020-01        10
    ## 2 2020-02      3139
    ## 3 2020-03      6636
    ## 4 2020-04       988
    ## 5 2020-05       729
    ## 6 2020-06      1347

이제 날짜가 범주형 변수가 되었기 때문에,  
x축은 `scale_x_discrete()`을 사용합니다.

``` r
ggplot(data_korea_monthly, aes(x=년월, y=new_cases)) +
  geom_col() +
  scale_x_discrete(name="",
                   labels = function(x) sub("-", "\n", x)
                   ) +
  scale_y_continuous(name="신규 확진자 수(명)") +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-01-handling-dates-in-time-series-charts/unnamed-chunk-7-1.png" width="70%" style="display: block; margin: auto;" alt="한국의 월별 코로나 신규 확진자 수 그래프" />

모든 달에 연도르 쓰면 차트가 복잡해 보일 수 있습니다.  
이럴 땐 1월에만 연도를 남기고,  
나머지는 월만 표시하는 방법도 있습니다.

"YYYY-01"일 경우 연도를 유지하고,  
그 외의 경우에는 연도를 제거하라는 함수를 `labels`에 넣어줍니다.
`grepl` 함수로 1월 해당 여부를 체크합니다.  
`ifelse`, `sub` 함수를 이용해서 1월에 해당하지 않으면 년도를 삭제하고,  
그 외의 경우에는 년도 대쉬 뒤에 줄 바꿈을 해줍니다.

``` r
ggplot(data=data_korea_monthly, aes(x=년월, y=new_cases)) +
  geom_col() +
  scale_x_discrete(name="",
                   labels=function(x){
                     y=sub("-", "\n", x)
                     ifelse(!grepl(".*(01)$", y), sub("^\\d{4}", "", y), y)
                   }) +
  scale_y_continuous(name="신규 확진자 수(명)") +
  theme_bw()
```

<img src="{{ site.baseurl }}/assets/images/2026-01-01-handling-dates-in-time-series-charts/unnamed-chunk-8-1.png" width="70%" style="display: block; margin: auto;" alt="1월에만 년도를 표시한 그래프" />

마지막으로 시각적인 요소를 조금 더 정리해봅니다.
- y축 단위를 만 명 단위로 변환
- 막대가 x축에 딱 붙도록 설정
- 막대 위 값 표시
- 글꼴과 색상 통일

``` r
# 글꼴 변경
library(showtext)

font_add_google("Noto Sans KR", "noto")

showtext_auto()
showtext_opts(dpi=300)

theme.size = 10
geom.text.size = theme.size/.pt 

# 그래프 작성
ggplot(data=data_korea_monthly, 
       aes(x=년월, y=new_cases/10000, 
           label=scales::comma(new_cases/10000, accuracy=1))
       ) +
  geom_col(fill="#9AD1F5") +
  geom_text(size=geom.text.size, family="noto", vjust=-0.5) +
  scale_x_discrete(name="",
                   labels=function(x){
                     y=sub("-", "\n", x)
                     ifelse(!grepl(".*(01)$", y), sub("^\\d{4}", "", y), y)
                   }) +
  scale_y_continuous(name="신규 확진자 수(만 명)",
                     labels=scales::comma_format(),
                     expand=expansion(mult=c(0, 0.2))) +
  theme_bw(base_size=theme.size, base_family="noto")
```

<img src="{{ site.baseurl }}/assets/images/2026-01-01-handling-dates-in-time-series-charts/unnamed-chunk-9-1.png" width="70%" style="display: block; margin: auto;" alt="한국의 월별 코로나 신규 확진자 수 그래프" />

날짜 데이터는 처음엔 다루기 까다롭지만,  
날짜 클래스로 변환하고 축 설정과 라벨만 잘 조절해주면  
시계열 차트를 손쉽게 그릴 수 있습니다.

다른 시계열 데이터에도 같은 방식을 적용해보세요.