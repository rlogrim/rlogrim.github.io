---
title: 반복문으로 그래프 자동 생성하고 일괄 저장하는 법
author: Rvinci
description: 똑같은 형식의 그래프 수십 개를 그려야 할 때, 아직도 하나하나 그리며 시간을 허비하고 계신가요? 17개 시도별 인구 추이 데이터를 예제로, R의 for 반복문을 활용해 차트를 자동 생성하고 일괄 저장하는 방법을 알려드립니다.
excerpt: 똑같은 형식의 그래프 수십 개를 그려야 할 때, 아직도 하나하나 그리며 시간을 허비하고 계신가요? 17개 시도별 인구 추이 데이터를 예제로, R의 for 반복문을 활용해 차트를 자동 생성하고 일괄 저장하는 방법을 알려드립니다.
categories: [chart]
tags: [r, tidyverse, ggplot2, for, ggsave]
published: true
---

## 언제 다 그리지? 수작업 지옥에서 탈출하기

데이터 분석 보고서를 작성하다 보면, 동일한 형식의 그래프를 지역별,
부서별, 혹은 제품별로 수십 개씩 만들어야 하는 상황이 생기곤 합니다.
엑셀이라면 시트마다 데이터를 필터링하고 차트를 복사해 붙여넣는 단순 반복
작업에 많은 시간을 쏟아야 하겠지만, R의 반복문(Loop) 기능을 활용하면 이
과정을 단 몇 초 만에 끝낼 수 있습니다.

이번 글에서는 1995년부터 2023년까지 17개 시도별 주민등록인구 추이
데이터를 활용하여, 반복문으로 그래프를 자동 생성하고 일괄 저장하는
방법을 알아보겠습니다.

## 글꼴 및 그림파일 크기 설정하기

자동화의 핵심은 모든 결과물이 일관된 품질을 유지하는 것입니다. 우선
필요한 패키지를 불러오고, 출력될 이미지의 규격과 글꼴 크기를 사전에
정의합니다.

``` r
# 패키지 로드
library(tidyverse)
library(scales)
library(showtext)

# 글꼴 설정
font_add("kopub", "C:/Users/.../AppData/Local/Microsoft/Windows/Fonts/Kopub dotum medium.ttf")
showtext_auto() # showtext 자동 활성화
showtext_opts(dpi=300) # dpi 300 설정

# 글꼴 크기 설정
theme.size = 10 # pt 단위
text.size = theme.size/.pt # mm 단위

# 그림파일 크기 설정
width <- 145
height <- 70
```

## 데이터 준비하기

데이터를 불러온 후, 반복문이 매끄럽게 돌아갈 수 있도록 전처리를
수행합니다. 인구수 단위를 ’만 명’으로 조정하고, 그래프가 너무 복잡해지지
않도록 2년 주기로 라벨을 표시하는 로직을 추가합니다.

``` r
# 데이터 로드
data <- read_xlsx("데이터/주민등록인구_시도_1995-2023.xlsx")

# 데이터 가공
data <- data %>% 
  rename(시도 = 행정구역별,
         주민등록인구수 = `계 (명)`) %>% 
  filter(시도 != "전국") %>% 
  mutate(시점 = as.character(시점),
         주민등록인구수 = as.numeric(주민등록인구수) / 10000,
         라벨 = case_when(시점 %in% as.character(seq(1995, 2023, by = 2)) ~ 주민등록인구수,
                        T ~ as.numeric(NA)))

# 데이터 확인
head(data)
```

    ## # A tibble: 6 × 4
    ##   시도       시점  주민등록인구수  라벨
    ##   <chr>      <chr>          <dbl> <dbl>
    ## 1 서울특별시 1995           1055. 1055.
    ## 2 서울특별시 1996           1042.   NA 
    ## 3 서울특별시 1997           1034. 1034.
    ## 4 서울특별시 1998           1027.   NA 
    ## 5 서울특별시 1999           1026. 1026.
    ## 6 서울특별시 2000           1031.   NA

## ’서울’을 예시로 검토하기

반복문을 돌리기 전, 기준이 될 ’표준 그래프’를 하나 만들어 디자인을
점검합니다. 서울특별시 데이터를 필터링하여 선 그래프와 포인트, 텍스트
라벨이 적절한 위치에 배치되는지 확인합니다.

``` r
data %>% 
  filter(시도 == "서울특별시") %>% 
  ggplot(aes(x = 시점, y = 주민등록인구수, 
             group = 시도, color = 시도,
             label = comma(라벨, accuracy = 1))) +
  geom_line(linewidth = 0.4) +
  geom_point(size = 3) +
  geom_text(vjust = -1,
            family = "kopub",
            size = text.size,
            color = "gray10") +
  scale_color_manual(values = "#CD5988") +
  scale_x_discrete(name = "",
                   expand = expansion(mult = 0.05)) +
  scale_y_continuous(name = "주민등록인구 수(만 명)",
                     expand = expansion(mult = 0.2),
                     labels = comma_format()) +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.line = element_line(linewidth = 0.5, color = "gray10"),
    axis.ticks = element_line(linewidth = 0.1, color = "gray10"),
    
    panel.grid.minor = element_blank(),
    
    axis.text = element_text(color = "gray10"),
    axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1),
    
    legend.title = element_blank(),
    legend.position = "bottom"
    )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-17-using-loops-to-automate-chart-generation/unnamed-chunk-3-1.png" alt="" width="100%" style="display: block; margin: auto;" />

## `for` 반복문으로 17개 시도 그래프 일괄 저장하기

이제 준비된 코드를 반복문 안에 집어넣을 차례입니다.
`unique(data$시도)`를 통해 17개 시도의 리스트를 뽑아내고, 각 지역을
하나씩 순회하며 그래프 생성부터 파일 저장까지 자동으로 수행하도록
합니다.

이때 가장 중요한 함수는 `paste0()`입니다. 이 함수를 사용하면 저장될 파일
이름에 해당 지역명을 동적으로 삽입하여, 각 파일이 덮어쓰기 되지 않고
고유한 이름을 가질 수 있게 합니다.

``` r
# 17개 시도 리스트 추출
지역목록 <- unique(data$시도)

for(지역 in 지역목록){

  # 1. 특정 지역 데이터 필터링 및 그래프 생성
  p <- data %>% 
    filter(시도 == 지역) %>% # 시도 필터링
    ggplot(aes(x = 시점, y = 주민등록인구수, 
               group = 시도, color = 시도,
               label = comma(라벨, accuracy = 1))) +
    geom_line(linewidth = 0.4) +
    geom_point(size = 3) +
    geom_text(vjust = -1,
              family = "kopub",
              size = text.size,
              color = "gray10") +
    scale_color_manual(values = "#CD5988") +
    scale_x_discrete(name = "",
                     expand = expansion(mult = 0.05)) +
    scale_y_continuous(name = "주민등록인구 수(만 명)",
                       expand = expansion(mult = 0.2),
                       labels = comma_format()) +
    theme_minimal(base_size = theme.size, base_family = "kopub") +
    theme(
      axis.line = element_line(linewidth = 0.5, color = "gray10"),
      axis.ticks = element_line(linewidth = 0.1, color = "gray10"),
      
      panel.grid.minor = element_blank(),
      
      axis.text = element_text(color = "gray10"),
      axis.text.x = element_text(angle = 45, vjust = 1, hjust = 1),
      
      legend.title = element_blank(),
      legend.position = "bottom"
      )
  
  # 2. 지정된 경로에 파일 저장
  ggsave(
    filename = paste0("아웃풋/1995~2023년 ", 지역, " 주민등록인구 추이.jpeg"), # 파일명에 지역명 추가
    plot = p,
    width = width,
    height = height,
    unit = "mm",
    dpi = 300
    )
  }
```

반복문이 종료되면 지정한 폴더에는 17개 지역의 그래프가 저장되어 있을
것입니다. 수작업으로 30분 이상 걸릴 일을 단 몇 초의 코드로 해결하는 것,
이것이 바로 R을 활용한 데이터 시각화 자동화의 매력입니다. 이 기술을
익히면 반복 업무에 소요되는 에너지를 줄이고, 대신 데이터가 보여주는
의미를 해석하는 ‘분석’ 본연의 가치에 더 집중할 수 있습니다. 여러분의
실무 환경에 이 방법를 적용하여 업무 효율을 극대화해 보시길 바랍니다!