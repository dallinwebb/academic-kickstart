+++
# Project title.
title = "Child Mortality"

# Date this page was created.
date = 2018-05-14

# Project summary to display on homepage.
summary = ""

# Tags: can be used for filtering projects.
# Example: `tags = ["royalties", "pcr"]`
tags = ["Data Wrangling and Visualization Course", "World Data"]

# Optional external URL for project (replaces project detail page).
external_link = ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references 
#   `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides = ""

# Links (optional).
url_pdf = ""
url_slides = ""
url_video = ""
url_code = ""

# Custom links (optional).
#   Uncomment line below to enable. For multiple links, use the form #`[{...}, {...}, {...}]`.


# Featured image
# To use, add an image named `featured.jpg/png` to your project's folder. 
[image]
  # Caption (optional)
  caption = ""
  
  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = "Smart"
+++

```r
devtools::install_github("drsimonj/ourworldindata")
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
              vjust = 1.3) +
  scale_x_continuous(labels = scales::dollar) +
  scale_y_continuous(labels = scales::percent) +
  labs(x = "\nTotal Health Expenditure",
       y = "Child Mortality\n",
       title = "Child Mortality Declines with Increased Health Expediture in the U.S.",
       caption = "Source: World Bank - WDI") +
  theme_minimal() +
  theme(title = element_text(size = 16),
        axis.text = element_text(size = 11))
```

<img src="/project/world_data_investigation_child_mortality/task_07_files/figure-html/unnamed-chunk-2-1.png" width="672" />


It appears that the biggest decreases in child mortality occured in the 90's. While it looks like healthcare expenditures increased faster in the 2000's, why did it take a decade for child mortality to start decreasing again?

Data and other figures are found [here](https://ourworldindata.org/financing-healthcare)

<br>
<br>
