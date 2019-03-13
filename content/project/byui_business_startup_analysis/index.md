+++
# Project title.
title = "Business Startup Analysis"

# Date this page was created.
date = 2018-08-13

# Project summary to display on homepage.
summary = "Startup's on BYU-Idaho campus"

# Tags: can be used for filtering projects.
# Example: `tags = ["royalties", "pcr"]`
tags = ["Business Finance", "Data Wrangling and Visualization Course", "Featured"]

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
sales <- read_csv("https://byuistats.github.io/M335/data/sales.csv")
```

## Background

Each semester, the business department at BYU-Idaho host the operation for a small number of student organized businesses. They all collected transactional data for us to analyze.

Posing as an investment company, we need to provide a recommendation for the best company to invest in. The decision makers would be better equipped if they know what customer traffic looks like and total income for each of the six companies.

<br>

## Data Wrangling

* Two rows were removed that were titled "missing." 

* Since transactions for all companies started during the middle of June, a few previous transactions were removed. 

* There were unexplained negative transactions in the amount column, the totals for which is summarized in a plot below. 

* Hours, week days, and months were then created from the original date variable.


```r
sales2 <- sales %>% 
  filter(Name != "Missing") %>% 
  mutate(Time = with_tz(Time, "America/Denver"),
         transaction = case_when(
           Amount < 0 ~ "Unclassified Loss",
           Amount >= 0 ~ "Gain"),
         hour = hour(Time),
         hour = factor(hour, levels = c(seq(6,23,1), 0)),
         week_day = wday(Time, label = T),
         month = month(Time))
```

## Data Exploration

As we see below, the data contain negative transactions that aren't explained. At the time of this analysis, the data collection period occurred two years prior, so we are unable to observe the data generation process. Therefore, I will assume these transactions to be a general loss of total income for each company. 


```r
sales2 %>% 
  group_by(Name, transaction) %>%  
  summarise(sum = sum(Amount)) %>%
  mutate(sum = abs(sum)) %>% 
ggplot(aes(fct_reorder(Name, sum, max), y = sum, 
           fill = factor(transaction))) +
  geom_bar(stat = "identity", position = "dodge") +
  scale_y_continuous(labels = scales::dollar,
                     breaks = c(0, 1000, 5000, 10000)) +
  scale_fill_manual(values = c("Unclassified Loss" = "#E66565",
                               "Gain" = "#85bb65")) +
  coord_flip() +
  labs(x = NULL,
       y = "Total Sales",
       fill = NULL,
       title = "All Companies Had ~ $500 - $1,000 In Negative Transactions") +
  theme_minimal()
```

<img src="/project/byui_business_startup_analysis/case_study_08_files/figure-html/unnamed-chunk-2-1.png" width="672" />

<br>


```r
sales2 %>% 
  filter(Amount > 0) %>% 
ggplot(aes(Name, Amount, fill = Type)) +
  geom_boxplot(outlier.shape = NA) +
  coord_cartesian(ylim = c(0,50)) +
  scale_y_continuous(labels = scales::dollar) +
  scale_fill_few(palette = "Medium") +
  labs(fill = NULL,
       x = NULL,
       y = "Avg. Sale Transaction\n",
       title = "Company Type: Food vs. Goods and Services") +
  theme_minimal()
```

<img src="/project/byui_business_startup_analysis/case_study_08_files/figure-html/unnamed-chunk-3-1.png" width="672" />

<br>

## Customer Traffic


```r
sales2 %>% 
  group_by(Name, week_day, hour) %>% count() %>% 
  filter(week_day != "Sun") %>% 
ggplot(aes(hour, fct_rev(week_day), fill = log(n))) +
  geom_raster() +
  scale_fill_gradient(low = "#DAFADF", high = "#00661D") +
  scale_x_discrete(labels = c(paste0(seq(6,11,1),"AM"), "12PM",
                              paste0(seq(1,11,1),"PM"), "12AM")) +
  facet_grid(Name~.) +
  labs(y = NULL,
       x = NULL,
       fill = "Customer\nTraffic Count",
       title = "Hot Diggity and Tacontento have Highest Customer Traffic",
       subtitle = "Frozone and Tacontento have meaningful Friday evening customer traffic") +
  theme_minimal() +
  theme(strip.text.y = element_text(angle = 0),
        panel.spacing = unit(.75,"lines"))
```

<img src="/project/byui_business_startup_analysis/case_study_08_files/figure-html/unnamed-chunk-4-1.png" width="672" />

<br>

## Sales


```r
sales %>% 
  mutate(Time = date(Time)) %>% 
  filter(Time > as.Date("2016-05-01"),
         Name != "Missing") %>% 
  group_by(Name, Time) %>% 
  summarise(sum = sum(Amount)) %>%
  mutate(cm = cumsum(sum)) %>% 
ggplot(aes(Time, cm, 
           col = fct_relevel(Name,
                             c("HotDiggity","LeBelle",
                               "Tacontento","SplashandDash",
                               "ShortStop","Frozone")))) +
  geom_line(size = 1) +
  geom_point(size = 3) +
  scale_color_few(palette = "Medium") +
  scale_y_continuous(labels = scales::dollar) +
  scale_x_date(date_breaks = "1 week",
               date_labels = "%b-%d") +
  labs(col = NULL,
       x = NULL,
       y = "Cummulative Sale Totals\n",
       title = "Hot Diggity Made the Most Money") +
  theme_minimal()
```

<img src="/project/byui_business_startup_analysis/case_study_08_files/figure-html/unnamed-chunk-5-1.png" width="672" />

<br>

## Conclusions

I would recommend investing in Hot Diggity for the following reasons. Hot Diggity maintained consistent increases in sale totals opposed to the competition like LeBelle who seems to have two periods of sporadic increases. This could be due to heavy discounts on their higher priced non-food items and therefore an unrealistic model for long term success. Hot Diggity also has the highest customer traffic. A fluctuation in customer flow would affect Hot Diggity much less than it would LeBelle given LeBelle has a smaller customer base.

Future analysis in encouraged to be conducted during the time of data generation to uncover the reason for negative transactions. We can only speculate now that the occurred due to returns, discounts, or expenses. The occurrence of these negative transactions, as interpreted as a general loss, are shown in the plot above after June 20th when Short Stop saw a significant negative transaction.

<br>

<br>
