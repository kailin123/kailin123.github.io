---
title: "Visualising Market Cap"
date: 2024-12-26T16:01:23+08:00
lastmod: 2024-12-26T16:01:23+08:00
draft: false
categories: ["visualisation", "r","plotly"]

menu:
  main:
    parent: "archives"
    weight: 2
---

In May 2024, DBS became the first Singapore-listed company to achieve SGD 100 billion in market capitalisation. While market cap is just one of many factors to consider when evaluating a company's performance, I was curious to see how the market cap of Singapore's local banks has changed over the years.


{{< blockquote >}}
  I used R and the fmpcloudr package to retrieve market cap data, then created a Plotly chart to make it easier to explore the numbers at specific points in time.
{{< /blockquote >}}

### Chart
<b>The below chart is best viewed on desktop.</b>

<span style="color:blue;">Hover mouse on chart to see data for specific dates</span>
{{< include-html "static/images/marketcap_visualisation.html">}}


&nbsp;
&nbsp;

### Load in required libraries and set token for fmpcloudr

```r
library(tidyverse)
library(lubridate)
library(fmpcloudr)
library(ggtext)
library(scales)
library(plotly)

fmpc_set_token('<insert token here>') # get token from: https://site.financialmodelingprep.com/developer/docs
```

### Prepare data set for plotting

```r
symbols = c('D05.SI', 'U11.SI', 'O39.SI')
data = fmpc_security_mrktcap(symbols, limit = 30*12*12)
data = data %>% mutate(date = as.Date(date, format ='%Y-%m-%d')) %>%
  mutate(name = case_when(symbol == 'D05.SI' ~ "DBS",
                          symbol == 'U11.SI' ~ "UOB",
                          symbol == 'O39.SI' ~ "OCBC"))
```

### Plot chart

```r
fig = ggplot(data, aes(date, marketCap, colour = name, group = 1, 
                       text = paste0("Date: ", format(date, "%d %b %Y"), "<br>Stock: ", name, "<br>Market Cap: <b>", paste(format(round(marketCap/1e9,1),trim = TRUE), "B"),"</b>"))) +
  geom_line() + 
  labs(title="<b>Market Cap Trend - <span style='color: #000000'>DBS</span>, <span style='color: #e11a27;'>OCBC</span> and <span style='color: #0060AE;'>UOB</span>",
       x=" ", y = "Market Cap")+ 
  scale_color_manual(values = c("#000000", "#e11a27", "#0060AE")) + 
  theme(plot.subtitle=element_text(size=18, hjust=0.5, face="italic", color="black")) +
  theme_bw() +
  theme(
    plot.title = element_markdown(size = 12),
    legend.position = "none",
  )+
  scale_y_continuous(labels = scales::label_number(scale_cut = cut_short_scale())) +
  scale_x_date(date_breaks = "6 month", 
               date_labels = "%b%y" , expand = c(0.1,0))+
  geom_hline(yintercept=100*1e9,linetype='dashed' ,col ='grey') +
  annotate("text", x = as.Date("2023-01-01"), y = 102*1e9 , label = '100 Billion', col = 'grey', size = 3)

ggplotly(fig, width = 650, height = 500, tooltip = "text") %>% layout(hovermode = "x unified")
```