---
title: "World Data Investigations - Child Mortality"
author: "Dallin Webb"
date: "March 11, 2019"
output:
  html_document:  
    keep_md: true
    code_folding: hide
    fig_height: 6
    fig_width: 12
    fig_align: 'center'
---






```r
finance <- ourworldindata::financing_healthcare
```

## Question

What is the relastionship between healthcare expenditure and child mortality over year? 


<br>



```r
finance %>% 
  select(year, country, health_exp_total, child_mort) %>% 
  filter(country == "United States") %>% 
  na.omit() %>% 
  mutate(child_mort = child_mort/100) %>% 
  ggplot(aes(health_exp_total, child_mort)) +
    geom_point(size = 2) +
    geom_text(aes(label = year),
              vjust = 1.4, size = 3) +
  scale_x_continuous(labels = scales::dollar) +
  scale_y_continuous(labels = scales::percent) +
  labs(x = "\nTotal Health Expenditure",
       y = "Child Mortality\n",
       title = "Child Mortality Declines as Health Expediture Increases in the U.S.",
       caption = "Source: World Bank - WDI") +
  theme_minimal() +
  theme(title = element_text(size = 12),
        axis.text = element_text(size = 9))
```

<img src="/project/world_data_investigation_child_mortality/task_07_files/figure-html/unnamed-chunk-2-1.png" width="672" />


It appears that the biggest decreases in child mortality occured in the 90's. While it looks like healthcare expenditures increased faster in the 2000's, why did it take a decade for child mortality to start decreasing again?

Data and other figures are found [here](https://ourworldindata.org/financing-healthcare)

<br>
<br>
