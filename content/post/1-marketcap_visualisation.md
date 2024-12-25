---
title: "Visualising Market Cap"
date: 2024-12-15T16:01:23+08:00
lastmod: 2024-12-15T16:01:23+08:00
draft: false
# tags: ["visualisation", "r"]
# categories: ["Data Exploration"]
categories: ["visualisation", "r"]

menu:
  main:
    parent: "archives"
    weight: 1
---

In May 2024, DBS became the first Singapore-listed company to achieve SGD 100 billion in market capitalisation. While market cap is just one of many factors to consider when evaluating a company's performance, I was curious to explore how the market capitalisations of Singapore's local banks have evolved over the years.

I used R and the fmpcloudr package to retrieve market cap data and plotted them over time

### End Result
&nbsp;
![This is an image in `static/image` folder.](/images/post1-marketcap.png)

&nbsp;
&nbsp;

### Load in required libraries and set token for fmpcloudr

```r
library(tidyverse)
library(lubridate)
library(fmpcloudr)
library(ggtext)
library(scales)

fmpc_set_token('<insert token here>') # get token from: https://site.financialmodelingprep.com/developer/docs
```

### Prepare data set for plotting

```r
symbols = c('D05.SI', 'U11.SI', 'O39.SI')
data = fmpc_security_mrktcap(symbols, limit = 30*12*12)
data = data %>% mutate(date = ymd(date)) %>%
  mutate(name = case_when(symbol == 'D05.SI' ~ "DBS",
                          symbol == 'U11.SI' ~ "UOB",
                          symbol == 'O39.SI' ~ "OCBC"))
```

### Plot chart

```r
ggplot(data, aes(date, marketCap, colour = symbol)) +
  geom_line() + 
  labs(title="<b>Market Cap Evolution - <span style='color: #000000'>DBS</span>, 
  <span style='color: #e11a27;'>OCBC</span> and
 <span style='color: #002469;'>UOB</span>",
       subtitle = "From November 2019 to Present",
       caption = "Data Source: site.financialmodelingprep.com (fmpcloudr package)",
       x=" ", y = "Market Cap")+ 
  scale_color_manual(values = c("#000000", "#e11a27", "#002469")) + 
  theme(plot.subtitle=element_text(size=18, hjust=0.5, face="italic", color="black")) +
  theme_bw() +
  theme(
    plot.title = element_markdown(lineheight = 1.1),
    plot.subtitle = element_markdown(size = 10),
    legend.position = "none",
    plot.caption = element_markdown(size = 10)
  )+
  scale_y_continuous(labels = scales::label_number(scale_cut = cut_short_scale())) +
  scale_x_date(date_breaks = "6 month", # Date labels for each month
               date_labels = "%b-%y" , expand = c(0.1,0))+
  geom_label(data = data %>% filter(date==max(date)),
             aes(label = paste0(name,' (',symbols, ') \n', paste(format(round(marketCap/1e9,1),trim = TRUE), "B"))),
             label.size = 0.2, size=3,
             label.padding = unit(0.2, "lines"), fontface="bold",
             nudge_x=200) +
  geom_label(data = data %>% filter(date == '2019-12-02'),
             aes(label = paste(format(round(marketCap/1e9,1),trim = TRUE), "B")   ),
             label.size = 0.2, size=3,
             label.padding = unit(0.2, "lines"), fontface="bold",
             nudge_x=-100) +
  geom_hline(yintercept=100*1e9,linetype='dashed' ,col ='blue') +
  annotate("text", x = min(data$date), y = 100*1e9, label = '100 Billion mark', vjust = -0.5, col = 'blue', size = 4)
```