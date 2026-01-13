---
title: 라벨에 백분율과 절대값 함께 표기하기
author: Rvinci
description: 백분율만 적어 둔 그래프를 보고 ‘그래서 실제로 몇 명인가요?’라는 질문을 받은 적이 있나요? 2023년 시도별 시, 군, 구 주민등록인구 비중 데이터를 예로, 누적막대그래프 라벨에 백분율과 절대값을 함께 표기하는 방법을 알려드립니다.
excerpt: 백분율만 적어 둔 그래프를 보고 ‘그래서 실제로 몇 명인가요?’라는 질문을 받은 적이 있나요? 2023년 시도별 시, 군, 구 주민등록인구 비중 데이터를 예로, 누적막대그래프 라벨에 백분율과 절대값을 함께 표기하는 방법을 알려드립니다.
categories: [chart]
tags: [r, tidyverse, ggplot2, geom_col, position_stack, label, paste0]
published: true
---

그래프에 백분율을 표시하면 구성비를 빠르게 파악할 수 있습니다. 다만
막대가 ’얼마나 큰 덩어리인지’는 감이 잘 오지 않는 경우가 많습니다. 예를
들어 ’여성이 40%’라는 정보만으로는 그것이 몇 만 명인지, 다른 지역과
비교해 어느 정도 수준인지 판단하기 어렵습니다. 그래서 라벨에 백분율과
절대값을 함께 적어두면 좀 더 정확한 정보를 전달할 수 있습니다.

이 글에서는 2023년 시도별 시, 군, 구 주민등록인구 비중 데이터를 사용해
누적막대그래프 라벨에 ’백분율 + 주민등록인구 수’를 병기하는 과정을
보여드립니다.

## 누적막대그래프에 백분율 표기하기

먼저, 누적막대그래프 조각마다 백분율을 적는 방법을 복습합니다. 시도별
비율을 막대로 쌓고, 이 값을 라벨로 표시합니다. 데이터 준비 과정과
누적막대그래프를 그리는 방법은 [누적막대그래프
그리기](https://rlogrim.github.io/chart/creating-stacked-bar-charts/)
글에 정리해 두었습니다.

``` r
ggplot(
  data = share2,
  aes(
    x = 시도, y = 비율, fill = 구분,
    label = percent(비율, accuracy = 0.1, suffix = "")
    )
  ) +
  geom_col(color = "white", linewidth = 0.4, width = 0.6) +
  geom_text_repel(
    family = "kopub",
    position = position_stack(vjust = 0.5),
    size = text.size,
    force = 0.001,
    force_pull = 1000
  ) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  scale_fill_brewer(palette = "Pastel2") +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.title = element_blank(),
    axis.text.y = element_blank(),
    
    axis.line.y = element_blank(),
    axis.line.x = element_line(color = "#959595", linewidth = 1.5),
    
    legend.title = element_blank(),
    legend.position = "bottom",
    legend.box.background = element_rect(color = "#959595", linewidth = 1),
    legend.key.size = unit(theme.size, "pt"),
    
    panel.grid = element_blank()
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-13-showing-percentages-and-absolute-values-together/unnamed-chunk-1-1.png" width="100%" style="display: block; margin: auto;" />

## 백분율에 절대값을 추가하기

이제 라벨에 절대값을 한 줄 더 추가합니다. 방법은 간단합니다. `label`에
들어갈 문자열을 직접 만들면 됩니다. 백분율은 `percent()`로, 절대값은
`comma()`로 보기 좋게 포맷팅한 뒤 `paste0()`로 이어 붙입니다.

여기서는 주민등록인구 수를 ‘만 명’ 단위로 바꿔 라벨이 지나치게 길어지지
않도록 했습니다. 라벨이 두 줄이 되면 텍스트가 더 복잡해 보일 수 있으니,
글자 크기를 조금 줄이고(`size = text.size * 0.8`), 줄 간격도
`lineheight`로 줄여줍니다.

``` r
ggplot(
  data = share2,
  aes(
    x = 시도, y = 비율, fill = 구분,
    label = paste0(percent(비율, accuracy = .1),
                   "\n(",
                   comma(주민등록인구수 / 10000, accuracy = 1, suffix = "만 명"),
                   ")"
                   )
    )
  ) +
  geom_col(color = "white", linewidth = 0.4, width = 0.6) +
  geom_text_repel(
    family = "kopub",
    position = position_stack(vjust = 0.5),
    size = text.size * 0.8,
    lineheight = 1,
    force = 0.001,
    force_pull = 1000
  ) +
  scale_y_continuous(expand = expansion(mult = c(0, 0.1))) +
  scale_fill_brewer(palette = "Pastel2") +
  theme_minimal(base_size = theme.size, base_family = "kopub") +
  theme(
    axis.title = element_blank(),
    axis.text.y = element_blank(),
    
    axis.line.y = element_blank(),
    axis.line.x = element_line(color = "#959595", linewidth = 1.5),
    
    legend.title = element_blank(),
    legend.position = "bottom",
    legend.box.background = element_rect(color = "#959595", linewidth = 1),
    legend.key.size = unit(theme.size, "pt"),
    
    panel.grid = element_blank()
  )
```

<img src="{{ site.baseurl }}/assets/images/2026-01-13-showing-percentages-and-absolute-values-together/unnamed-chunk-2-1.png" width="100%" style="display: block; margin: auto;" alt="백분율과 절대값을 함께 표기한 막대그래프" />
