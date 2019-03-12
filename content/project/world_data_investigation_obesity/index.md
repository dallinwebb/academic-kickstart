+++
# Project title.
title = "Prevalence of Obesity"

# Date this page was created.
date = 2018-05-08

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
  caption = "test image"
  
  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = "Smart"
+++

```r
obese <- read_csv("prevalence-of-obesity-in-adults-by-region.csv")
```

## Background

Data comes from [Our World in Data](https://ourworldindata.org/obesity). Obesity has been a growing problem for many parts of the world. I want to visualize this trend by region, assuming that more wealthier regions are leading the world. Then, I look at how child mortality is effected by education level.

<br>

## Obesity


```r
obese <- obese %>% 
  rename(continent  = Entity, 
         year       = Year,
         prevalence = `Prevalence of obesity in adults (18+ years old) (FAO (2017)) (%)`)

label <- obese %>% filter(year == 2014)
ggplot(obese, aes(year, (prevalence)/100, group = continent, col = continent)) +
  geom_point() +
  geom_line() +
  theme_minimal() +
  scale_y_continuous(limits = c(0,.3), labels = scales::percent) +
  geom_label_repel(aes(label = continent), data = label, nudge_x = 1000) +
  theme(legend.position = "none") +
  labs(title = "Prevalence of obesity in adults by region",
       subtitle = "The prevalence of obesity in adults, measured as the percentage of adults aged 18 years and older (both male and \nfemale) with a body-mass index (BMI) greater than 30 kilograms per metre squared.",
       x = "",
       y = "",
       caption = "Source: UN Food and Agricultural Organization/WHO")
```

<img src="/project/world_data_investigation_obesity/world_data_investigation_files/figure-html/plot_data-1.png" width="672" />

North America and Europe have lead the world in the rise of obesity, closely followed by Oceania. For these continents, obesity rates are approaching a third of their populations. These two countries along with Latin America and the Caribbean are well above the world average obesity rates.

<br>

## Child Mortality


```r
child_mortality %>% 
  select(education, child_mort, continent) %>% 
  na.omit() %>% 
  mutate(continent = as.factor(continent)) %>% 
ggplot(aes(education, log10(child_mort), col = continent)) +
  geom_point(alpha = .1) +
  geom_smooth(method = "lm", se = F) +
  theme_minimal() +
  labs(x = "Education (Higher is better)",
       y = TeX("$\\log_{10}$(Child Mortality)"),
       title = "Child Mortality Decreases as Education Increases",
       subtitle = "Europe seems to be doing particularily better than other continents \nwith lower child mortality rates while haveing the same education as other continents.",
       caption = "\nSource: UN Population Division (2017 Revision)",
       col = "Continent")
```

<img src="/project/world_data_investigation_obesity/world_data_investigation_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Regressing the log of child mortality on education level grouped by continent, we see something interesting. With the same level of education, Europe seems to be doing better than the other continents in regard to child mortality. We also notice Africa doesn't have the same opportunity for as much education as the other continents do. 

<br>
<br>
