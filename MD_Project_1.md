## Table of Contents

-   [Executive Summary](#executive-summary)
-   [1 Introduction](#1-introduction)
-   [2 Data Description](#2-data-description)
-   [3 Data Preparation](#3-data-preparation)
    -   [3.1 Import Data](#31-import-data)
    -   [3.2 Assign and Create
        Variables](#32-assign-and-create-variables)
    -   [3.3 Data Cleaning](#33-data-cleaning)
-   [4 Exploratory Data Analysis EDA](#4-exploratory-data-analysis-eda)
    -   [4.1 Summary Table](#41-summary-table)
    -   [4.2 Bar Charts](#42-bar-charts)
    -   [4.3 Pie Charts](#43-pie-charts)
    -   [4.4 Box Plots](#44-box-plots)
    -   [4.5 Summary Report](#45-summary-report)
-   [5 Statistical Modeling](#5-statistical-modeling)
    -   [5.1 Unadjusted Logistic
        Regression](#51-unadjusted-logistic-regression)
    -   [5.2 Adjusted Logistic
        Regression](#52-adjusted-logistic-regression)
    -   [5.3 Adjusted LRM Table](#53-adjusted-lrm-table)
-   [6 Discussion](#6-discussion)
-   [7 Conclusion](#7-conclusion)
-   [8 References and Resources](#8-references-and-resources)

# Project 1: HIV Testing Behaviour Analysis - NYC Community Health Survey 2020

**Rawda Alaswad**  
**Date:** July 15, 2025

## Executive Summary

This report analyzes data from the 2020 New York City Community Health
Survey (CHS) to examine demographic and behavioral factors associated
with HIV testing among adults in NYC. The survey included approximately
10,000 adults, with data collected via a cross-sectional, self-reported
telephone survey in multiple languages.

Key variables analyzed include race/ethnicity, age group, HIV testing in
the past 12 months, condom use, and number of sex partners. Data were
cleaned and recoded for clarity, and survey weights were applied to
account for the complex sampling design.

Descriptive analyses showed that the largest racial/ethnic group was
White/North African/Middle Eastern, non-Hispanic, and most participants
were aged 25–64. Only 28.6% of respondents reported being tested for HIV
in the past year, and 17.3% reported condom use at last sex.

Logistic regression models were used to assess associations between HIV
testing and behavioral/demographic factors. In un-adjusted analysis,
condom use was significantly associated with higher odds of HIV testing.
However, after adjusting for age group, race/ethnicity, and number of
sex partners, this association was no longer statistically significant.
Black, Hispanic, and Other non-Hispanic participants, as well as those
with more sex partners, were more likely to have been tested for HIV,
while older age groups were less likely.

These findings highlight disparities in HIV testing and suggest the need
for targeted public health interventions to increase testing rates among
specific demographic groups in NYC.

## 1 Introduction

This report utilizes data from the 2020 New York City Community Health
Survey (CHS) to explore patterns of HIV testing among adults in NYC. The
analysis focuses on the relationships between HIV testing and key
variables such as race/ethnicity, age group, condom use, and number of
sex partners. By identifying groups with lower testing rates and
examining associated behaviors, this study aims to provide actionable
insights for improving HIV prevention and care in NYC.

## 2 Data Description

**Source:** The data used in this project is the New York City Community
Health Survey (CHS) of 2020, conducted annually by the NYC Department of
Health and Mental Hygiene (DOHMH).
<https://www.nyc.gov/site/doh/data/data-sets/community-health-survey-public-use-data.page>

**Sample Size:** The survey includes approximately 10,000 adults aged 18
and older, residing in New York City and its five boroughs.

**Collection Method:** Data were collected via a cross-sectional,
self-reported telephone survey. Interviews were conducted in English,
Spanish, Russian, and Chinese (Mandarin and Cantonese).

**Variables Key:**

| Original Variable Name | Recoded Name/ Label | Description |
|------------------------|------------------------|------------------------|
| newrace | Race/Ethnicity | Self-identified race/ethnicity of participant |
| agegroup | Age Group | Age group in years |
| hiv12months20 | Tested for HIV (Past 12 Months) | Whether participant was tested for HIV in past 12 months |
| condom20 | Condom Use (Past 12 Months) | Whether participant used a condom the last time they had sex |
| sexpartner | Number of Sex Partners (Past 12 Months) | Number of sex partners in the past 12 months |

**Statistical Methods:** Survey weights and design variables were used
to account for the complex sampling design. Descriptive statistics,
summary tables, and visualizations were produced. Logistic regression
models (both unadjusted and adjusted for confounders) were fitted using
the ‘survey’ package in R to assess associations between HIV testing and
behavioral/ demographic factors.

**Libraries Used:**

``` r
library(haven)
library(dplyr)
library(survey)
library(ggplot2)
library(forcats)
library(broom)
library(knitr)
library(viridis)
```

## 3 Data Preparation

### 3.1 Import Data

-   The data was downlaoded on to local harddrive from the NYC CHS
    website (2020 SAS Data File named chs2020_public.sas7bdat). The data
    set was assigned the named chs20 for this project.

``` r
library(haven)
chs20 <- read_sas("C:/Users/rawda/Desktop/chs2020_public.sas7bdat", 
    NULL)

View(chs20)
```

### 3.2 Assign and Create Variables

-   Assigning numeric values to the categories of each factor to reocde
    them for the epi table. The recoded data set is called chs20_recode.

``` r
library(dplyr)

chs20_recode <- chs20 %>%
  mutate(
    newrace = recode_factor(newrace,
      `1` = "White/N Afri/MidEastern, non-Hispanic",
      `2` = "Black, non-Hispanic",
      `3` = "Hispanic",
      `4` = "Asian/PI, non-Hispanic",
      `5` = "Other, non-Hispanic"
    ),
    agegroup = recode_factor(agegroup,
      `1` = "18-24",
      `2` = "25-44",
      `3` = "45-64",
      `4` = "65+"
    ),
    hiv12months20 = recode_factor(hiv12months20,
      `2` = "No",
      `1` = "Yes"
    ),
    condom20 = recode_factor(condom20,
      `2` = "No",
      `1` = "Yes"
    ),
    sexpartner = factor(sexpartner, ordered = TRUE),
    strata = as.character(strata)
  ) %>%
  rename(
    "Race/Ethnicity" = newrace,
    "Age Group" = agegroup,
    "Tested for HIV (Past 12 Months)" = hiv12months20,
    "Condom Use (Past 12 Months)" = condom20,
    "Number of Sex Partners (Past 12 Months)" = sexpartner
  )
```

### 3.3 Data Cleaning

-   The recoded data was checked to ensure that the recoding process was
    completed correctly and that all variables and cases are present.

``` r
vars_of_interest <- c("Race/Ethnicity", "Age Group","Tested for HIV (Past 12 Months)","Condom Use (Past 12 Months)", "Number of Sex Partners (Past 12 Months)")

colSums(is.na(chs20_recode[vars_of_interest]))
```

    ##                          Race/Ethnicity                               Age Group 
    ##                                       0                                      14 
    ##         Tested for HIV (Past 12 Months)             Condom Use (Past 12 Months) 
    ##                                     152                                    3688 
    ## Number of Sex Partners (Past 12 Months) 
    ##                                     861

``` r
for (var in vars_of_interest) {
  print(var)
  print(table(chs20_recode[[var]], useNA = "ifany"))
}
```

    ## [1] "Race/Ethnicity"
    ## 
    ## White/N Afri/MidEastern, non-Hispanic                   Black, non-Hispanic 
    ##                                  2859                                  1837 
    ##                              Hispanic                Asian/PI, non-Hispanic 
    ##                                  2457                                  1340 
    ##                   Other, non-Hispanic 
    ##                                   288 
    ## [1] "Age Group"
    ## 
    ## 18-24 25-44 45-64   65+  <NA> 
    ##   706  3189  2951  1921    14 
    ## [1] "Tested for HIV (Past 12 Months)"
    ## 
    ##   No  Yes <NA> 
    ## 6122 2507  152 
    ## [1] "Condom Use (Past 12 Months)"
    ## 
    ##   No  Yes <NA> 
    ## 3570 1523 3688 
    ## [1] "Number of Sex Partners (Past 12 Months)"
    ## 
    ##    1    2    3    4 <NA> 
    ## 2686 4263  374  597  861

## 4 Exploratory Data Analysis (EDA)

### 4.1. Summary Table

``` r
# Creating Survey
library(survey)
chs.dsgn <-
  svydesign(
    ids = ~ 1,
    strata = ~ strata,
    weights =  ~ wt21_dual, 
    data = chs20_recode,  
    nest = TRUE,
    na.rm = TRUE
  )
```

``` r
knitr::kable(
  summary(chs20_recode[, c("Race/Ethnicity", "Age Group", "Tested for HIV (Past 12 Months)", "Condom Use (Past 12 Months)", "Number of Sex Partners (Past 12 Months)" )]),
  caption = "Epidemiological Summary Table"
)
```

|  | Race/Ethnicity | Age Group | Tested for HIV (Past 12 Months) | Condom Use (Past 12 Months) | Number of Sex Partners (Past 12 Months) |
|:--|:-------------------|:-----|:--------------|:------------|:-----------------|
|  | White/N Afri/MidEastern, non-Hispanic:2859 | 18-24: 706 | No :6122 | No :3570 | 1 :2686 |
|  | Black, non-Hispanic :1837 | 25-44:3189 | Yes :2507 | Yes :1523 | 2 :4263 |
|  | Hispanic :2457 | 45-64:2951 | NA’s: 152 | NA’s:3688 | 3 : 374 |
|  | Asian/PI, non-Hispanic :1340 | 65+ :1921 | NA | NA | 4 : 597 |
|  | Other, non-Hispanic : 288 | NA’s : 14 | NA | NA | NA’s: 861 |

Epidemiological Summary Table

<figure>
<img src="Visuals/descriptive_statistics_table.png"
alt="Descriptive Statistics Table" />
<figcaption aria-hidden="true">Descriptive Statistics Table</figcaption>
</figure>

### 4.2 Bar Charts

``` r
library(ggplot2)

bar_vars <- c(
  "Race/Ethnicity",
  "Age Group",
  "Tested for HIV (Past 12 Months)",
  "Condom Use (Past 12 Months)",
  "Number of Sex Partners (Past 12 Months)"
)

short_names <- c(
  "Race_Ethnicity",
  "Age_Group",
  "Tested_for_HIV",
  "Condom_Use",
  "Num_Sex_Partners"
)
names(short_names) <- bar_vars

for (var in bar_vars) {
  p <- ggplot(chs20_recode, aes(x = .data[[var]])) +
    geom_bar(fill = "light blue") +
    geom_text(stat = "count", aes(label = ..count..), vjust = -0.5, size = 3) +
    labs(
      title = paste("Distribution of", var),
      x = var,
      y = "Count"
    ) +
    theme(
      panel.background = element_rect(fill = "white"),
      plot.background = element_rect(fill = "white"),
      legend.background = element_rect(fill = "white")
    )
  print(p)
  safe_var <- short_names[var]
  ggsave(filename = paste0("C:/Users/rawda/Desktop/Notion/Visuals/", safe_var, "_bar_chart.png"), plot = p, width = 6, height = 4)
}
```

![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-7-1.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-7-2.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-7-3.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-7-4.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-7-5.png)

![](Visuals/Race_Ethnicity_bar_chart.png)
![](Visuals/Age_Group_bar_chart.png)
![](Visuals/Tested_for_HIV_bar_chart.png)
![](Visuals/Condom_Use_bar_chart.png)
![](Visuals/Num_Sex_Partners_bar_chart.png)

### 4.3 Pie Charts

``` r
library(ggplot2)
library(dplyr)

cat_vars <- c(
  "Race/Ethnicity",
  "Age Group",
  "Tested for HIV (Past 12 Months)",
  "Condom Use (Past 12 Months)",
  "Number of Sex Partners (Past 12 Months)"
)

short_names <- c(
  "Race_Ethnicity",
  "Age_Group",
  "Tested_for_HIV",
  "Condom_Use",
  "Num_Sex_Partners"
)
names(short_names) <- cat_vars

for (var in cat_vars) {
  pie_data <- chs20_recode %>%
    count(.data[[var]]) %>%
    mutate(perc = n / sum(n) * 100,
           label = paste0(.data[[var]], " (", round(perc, 1), "%)"))
  
  p <- ggplot(pie_data, aes(x = "", y = n, fill = .data[[var]])) +
    geom_col(width = 1) +
    coord_polar(theta = "y") +
    labs(
      title = paste("Distribution of", var),
      fill = var
    ) +
    theme_void() +
    geom_text(aes(label = label), position = position_stack(vjust = 0.5), size = 3) +
    theme(
      plot.background = element_rect(fill = "white", color = NA),
      panel.background = element_rect(fill = "white", color = NA),
      legend.background = element_rect(fill = "white", color = NA)
    )
  
  print(p) 
  safe_var <- short_names[var]
  ggsave(
    filename = paste0("C:/Users/rawda/Desktop/Notion/Visuals/", safe_var, "_pie_chart.png"),
    plot = p,
    width = 6,
    height = 6,
    bg = "white" 
  )
}
```

![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-8-1.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-8-2.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-8-3.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-8-4.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-8-5.png)

![](Visuals/Race_Ethnicity_pie_chart.png)
![](Visuals/Age_Group_pie_chart.png)
![](Visuals/Tested_for_HIV_pie_chart.png)
![](Visuals/Condom_Use_pie_chart.png)
![](Visuals/Num_Sex_Partners_pie_chart.png)

### 4.4 Box Plots

``` r
library(ggplot2)
library(forcats)
library(dplyr)
library(viridis)

box_vars <- c(
  "Condom Use (Past 12 Months)",
  "Race/Ethnicity",
  "Number of Sex Partners (Past 12 Months)"
)

short_names <- c(
  "Condom_Use",
  "Race_Ethnicity",
  "Num_Sex_Partners"
)
names(short_names) <- box_vars

chs20_recode$`Age Group` <- forcats::fct_explicit_na(chs20_recode$`Age Group`, na_level = "Missing")
chs20_recode$`Age Group` <- factor(
  chs20_recode$`Age Group`,
  levels = c("18-24", "25-44", "45-64", "65+", "Missing")
)

for (fill_var in box_vars) {
  plot_data <- chs20_recode %>%
    group_by(`Tested for HIV (Past 12 Months)`, .data[[fill_var]], `Age Group`) %>%
    summarise(n = n(), .groups = "drop") %>%
    group_by(`Tested for HIV (Past 12 Months)`, `Age Group`) %>%
    mutate(perc = n / sum(n) * 100)
  
  p <- ggplot(plot_data, aes(x = `Tested for HIV (Past 12 Months)`, y = n, fill = .data[[fill_var]])) +
    geom_bar(stat = "identity", position = "stack", color = "black") +
    facet_grid(. ~ `Age Group`, labeller = label_value) +
    labs(
      title = paste("HIV Testing by", fill_var),
      subtitle = "Stratified by Age Group",
      x = "Tested for HIV (Past 12 Months)",
      y = "Number of Participants",
      fill = fill_var
    ) +
    scale_fill_viridis_d(option = "D") +
    theme_bw() +
    theme(plot.background = element_rect(fill = "white", color = NA))
  
  print(p)
  safe_var <- short_names[fill_var]
  ggsave(filename = paste0("C:/Users/rawda/Desktop/Notion/Visuals/HIV_Testing_by_", safe_var, "_bar_chart.png"), plot = p, width = 7, height = 4, bg = "white")
}
```

![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-9-1.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-9-2.png)![](MD_Project_1_files/figure-markdown_github/unnamed-chunk-9-3.png)

![](Visuals/HIV_Testing_by_Condom_Use_Stacked.png)

![](Visuals/HIV_Testing_by_Sex_Partners_Stacked.png)

![](Visuals/HIV_Testing_by_Race_Stacked.png)

![](Visuals/HIV_Testing_by_Age_Stacked.png)

![](Visuals/HIV_Testing_by_Condom_Use_bar_chart.png)
![](Visuals/HIV_Testing_by_Race_Ethnicity_bar_chart.png)
![](Visuals/HIV_Testing_by_Num_Sex_Partners_bar_chart.png)

### 4.5 Summary Report

**Race/Ethnicity:** The largest group was White/North African/Middle
Eastern, non-Hispanic (32.6%), followed by Hispanic (28.0%), Black,
non-Hispanic (20.9%), Asian/Pacific Islander, non-Hispanic (15.3%), and
Other, non-Hispanic (3.3%).

**Age Group:** Most participants were aged 25–44 (36.3%) or 45–64
(33.6%), with smaller proportions aged 65+ (21.9%) and 18–24 (8.0%).

**HIV Testing (Past 12 Months):** 28.6% reported being tested for HIV in
the past year, while 69.7% had not been tested.

**Condom Use (Past 12 Months):** 17.3% reported using a condom the last
time they had sex, 40.7% did not, and 42.0% had missing data for this
question.

**Number of Sex Partners (Past 12 Months):** Nearly half (48.5%)
reported having 2 sex partners, 30.6% had 1 partner, and smaller
proportions reported 3 (4.3%) or 4 (6.8%) partners. About 9.8% had
missing data for this variable.

## 5 Statistical Modeling

### 5.1 Unadjusted Logistic Regression

This step was done to highlight the effect of confounded on p-value
significance. As you can see below there is an extremely signifncant
statistical association between condom HIV testing and condom use.
Condom use in the past 12 months was significantly associated with
higher odds of HIV testing in the past year (p \< 0.001).

``` r
fit_unadj <- svyglm(`Tested for HIV (Past 12 Months)` ~ `Condom Use (Past 12 Months)`, 
                    design = chs.dsgn, 
                    family = quasibinomial())
summary(fit_unadj)
```

    ## 
    ## Call:
    ## svyglm(formula = `Tested for HIV (Past 12 Months)` ~ `Condom Use (Past 12 Months)`, 
    ##     design = chs.dsgn, family = quasibinomial())
    ## 
    ## Survey design:
    ## svydesign(ids = ~1, strata = ~strata, weights = ~wt21_dual, data = chs20_recode, 
    ##     nest = TRUE, na.rm = TRUE)
    ## 
    ## Coefficients:
    ##                                  Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                      -0.75078    0.05229 -14.359  < 2e-16 ***
    ## `Condom Use (Past 12 Months)`Yes  0.49119    0.09332   5.263 1.47e-07 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for quasibinomial family taken to be 1.072588)
    ## 
    ## Number of Fisher Scoring iterations: 4

<figure>
<img src="Visuals/UNadjusted_regression_table.png"
alt="Unadjusted Regression Table" />
<figcaption aria-hidden="true">Unadjusted Regression Table</figcaption>
</figure>

### 5.2 Adjusted Logistic Regression

After adjusting for age group, race/ethnicity, and number of sex
partners, condom use was no longer significantly associated with HIV
testing in the past year (p = 0.34).

``` r
fit_adj <- svyglm(
  `Tested for HIV (Past 12 Months)` ~ 
  `Condom Use (Past 12 Months)` + `Age Group` + `Race/Ethnicity` + `Number of Sex Partners (Past 12 Months)`,
  design = chs.dsgn,
  family = quasibinomial()
)
summary(fit_adj)
```

    ## 
    ## Call:
    ## svyglm(formula = `Tested for HIV (Past 12 Months)` ~ `Condom Use (Past 12 Months)` + 
    ##     `Age Group` + `Race/Ethnicity` + `Number of Sex Partners (Past 12 Months)`, 
    ##     design = chs.dsgn, family = quasibinomial())
    ## 
    ## Survey design:
    ## svydesign(ids = ~1, strata = ~strata, weights = ~wt21_dual, data = chs20_recode, 
    ##     nest = TRUE, na.rm = TRUE)
    ## 
    ## Coefficients:
    ##                                              Estimate Std. Error t value
    ## (Intercept)                                 -0.937320   0.178702  -5.245
    ## `Condom Use (Past 12 Months)`Yes             0.099512   0.105003   0.948
    ## `Age Group`25-44                             0.200233   0.166301   1.204
    ## `Age Group`45-64                            -0.397506   0.177955  -2.234
    ## `Age Group`65+                              -1.187429   0.257802  -4.606
    ## `Race/Ethnicity`Black, non-Hispanic          1.437544   0.130397  11.024
    ## `Race/Ethnicity`Hispanic                     1.327000   0.118557  11.193
    ## `Race/Ethnicity`Asian/PI, non-Hispanic       0.229180   0.175734   1.304
    ## `Race/Ethnicity`Other, non-Hispanic          0.729676   0.288983   2.525
    ## `Number of Sex Partners (Past 12 Months)`.L  0.862675   0.112311   7.681
    ## `Number of Sex Partners (Past 12 Months)`.Q -0.008584   0.149061  -0.058
    ##                                             Pr(>|t|)    
    ## (Intercept)                                 1.63e-07 ***
    ## `Condom Use (Past 12 Months)`Yes              0.3433    
    ## `Age Group`25-44                              0.2286    
    ## `Age Group`45-64                              0.0255 *  
    ## `Age Group`65+                              4.21e-06 ***
    ## `Race/Ethnicity`Black, non-Hispanic          < 2e-16 ***
    ## `Race/Ethnicity`Hispanic                     < 2e-16 ***
    ## `Race/Ethnicity`Asian/PI, non-Hispanic        0.1923    
    ## `Race/Ethnicity`Other, non-Hispanic           0.0116 *  
    ## `Number of Sex Partners (Past 12 Months)`.L 1.89e-14 ***
    ## `Number of Sex Partners (Past 12 Months)`.Q   0.9541    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for quasibinomial family taken to be 1.051299)
    ## 
    ## Number of Fisher Scoring iterations: 4

<figure>
<img src="Visuals/adjusted_regression_table.png"
alt="Adjusted Regression Table" />
<figcaption aria-hidden="true">Adjusted Regression Table</figcaption>
</figure>

<figure>
<img src="Visuals/HIV_Testing_Forest_Plot.png"
alt="Forst Plot for Adjusted Regression Table" />
<figcaption aria-hidden="true">Forst Plot for Adjusted Regression
Table</figcaption>
</figure>

### 5.3 Adjusted LRM Table

``` r
library(broom)
library(dplyr)
library(knitr)

var_labels <- c(
  "Condom Use (Past 12 Months)",
  "Age Group",
  "Race/Ethnicity",
  "Number of Sex Partners (Past 12 Months)"
)

fit_adj_tidy <- tidy(fit_adj, exponentiate = TRUE, conf.int = TRUE)

pretty_term <- function(term) {
  if (term == "(Intercept)") return("Intercept")
  if (term == "Condom Use (Past 12 Months)Yes") return("Condom Use (Past 12 Months): Yes")
  if (grepl("^Age Group", term)) return(paste0("Age Group: ", sub("Age Group", "", term)))
  if (grepl("^Race/Ethnicity", term)) return(paste0("Race/Ethnicity: ", sub("Race/Ethnicity", "", term)))
  if (grepl("Number of Sex Partners (Past 12 Months).L", term)) return("Number of Sex Partners (Past 12 Months): Linear")
  if (grepl("Number of Sex Partners (Past 12 Months).Q", term)) return("Number of Sex Partners (Past 12 Months): Quadratic")
  return(term)
}

fit_adj_tidy <- fit_adj_tidy %>%
  mutate(
    term = sapply(term, pretty_term),
    across(where(is.numeric), ~round(., 3)),
    p.value = ifelse(
      p.value < 0.001,
      "0.000",
      sprintf("%.3f", p.value)
    ),
    p.value = ifelse(as.numeric(p.value) < 0.05, paste0(p.value, " *"), p.value)
  )

fit_adj_tidy %>%
  select(term, estimate, std.error, statistic, p.value, conf.low, conf.high) %>%
  knitr::kable(
    caption = "Adjusted Logistic Regression Results: Odds Ratios and 95% Confidence Intervals",
    col.names = c("Variable", "Odds Ratio", "Std. Error", "z value", "p-value", "CI Lower", "CI Upper"),
    align = "c"
  )
```

| Variable | Odds Ratio | Std. Error | z value | p-value | CI Lower | CI Upper |
|:--------------------------:|:-------:|:-------:|:-----:|:-----:|:-----:|:-----:|
| Intercept | 0.392 | 0.179 | -5.245 | 0.000 \* | 0.276 | 0.556 |
| `Condom Use (Past 12 Months)`Yes | 1.105 | 0.105 | 0.948 | 0.343 | 0.899 | 1.357 |
| `Age Group`25-44 | 1.222 | 0.166 | 1.204 | 0.229 | 0.882 | 1.693 |
| `Age Group`45-64 | 0.672 | 0.178 | -2.234 | 0.026 \* | 0.474 | 0.953 |
| `Age Group`65+ | 0.305 | 0.258 | -4.606 | 0.000 \* | 0.184 | 0.506 |
| `Race/Ethnicity`Black, non-Hispanic | 4.210 | 0.130 | 11.024 | 0.000 \* | 3.261 | 5.437 |
| `Race/Ethnicity`Hispanic | 3.770 | 0.119 | 11.193 | 0.000 \* | 2.988 | 4.756 |
| `Race/Ethnicity`Asian/PI, non-Hispanic | 1.258 | 0.176 | 1.304 | 0.192 | 0.891 | 1.775 |
| `Race/Ethnicity`Other, non-Hispanic | 2.074 | 0.289 | 2.525 | 0.012 \* | 1.177 | 3.655 |
| `Number of Sex Partners (Past 12 Months)`.L | 2.369 | 0.112 | 7.681 | 0.000 \* | 1.901 | 2.953 |
| `Number of Sex Partners (Past 12 Months)`.Q | 0.991 | 0.149 | -0.058 | 0.954 | 0.740 | 1.328 |

Adjusted Logistic Regression Results: Odds Ratios and 95% Confidence
Intervals

## 6 Discussion

**LRM:** Without accounting for hypothesized confounders (age, race, \#
of partners) the association of HIV testing and condom usage is
significant. In relation to adjusting for the hypothesized confounders
we can see that participants who used condoms the last time the had sex
compared to those who didn’t had a 0.09951 increase in the log odds of
having HIV testing with in the last 12 months. However, the positive
association is not significant as the p-value is 0.3433.

## 7 Conclusion

Participants are more likely to have HIV testing done if they identify
as: Black- non-Hispanic, Hispanic, or Other-non-Hispanic, compared to
the other ethno-racial groups. Survey participants were less likely to
HIV testing done if they were aged: 45-64, 65+ as opposed to the other
age groups. Furthermore, there is a positive linear relationship between
the number of sex partners and HIV testing meaning the more sex partners
someone has the more likely a participant is to have HIV testing done.

## 8 Refrences and Resources

**Libraries Used**

-   Wickham, H. & Miller, E. (2024). haven: Import and Export ‘SPSS’,
    ‘Stata’ and ‘SAS’ Files. R package version 2.5.4.
    <https://CRAN.R-project.org/package=haven>

-   Wickham, H., François, R., Henry, L., & Müller, K. (2023). dplyr: A
    Grammar of Data Manipulation. R package version 1.1.4.
    <https://CRAN.R-project.org/package=dplyr>

-   Lumley, T. (2024). survey: analysis of complex survey samples. R
    package version 4.3-2. <https://CRAN.R-project.org/package=survey>

-   Harrell, F.E. Jr. (2024). Hmisc: Harrell Miscellaneous. R package
    version 5.1-1. <https://CRAN.R-project.org/package=Hmisc>

-   Rich, B. (2023). table1: Tables of Descriptive Statistics in HTML. R
    package version 1.4.4. <https://CRAN.R-project.org/package=table1>

-   Wickham, H. (2016). ggplot2: Elegant Graphics for Data Analysis.
    Springer-Verlag New York. <https://ggplot2.tidyverse.org/>

-   Wickham, H. (2024). forcats: Tools for Working with Categorical
    Variables (Factors). R package version 1.0.1.
    <https://CRAN.R-project.org/package=forcats>

-   Robinson, D., Hayes, A., & Couch, S. (2024). broom: Convert
    Statistical Objects into Tidy Tibbles. R package version 1.0.5.
    <https://CRAN.R-project.org/package=broom>

-   Xie, Y. (2024). knitr: A General-Purpose Package for Dynamic Report
    Generation in R. R package version 1.46. <https://yihui.org/knitr/>

-   Zhu, H. (2024). kableExtra: Construct Complex Table with ‘kable’ and
    Pipe Syntax. R package version 1.4.0.
    <https://CRAN.R-project.org/package=kableExtra>

-   Garnier, S. (2024). viridis: Colorblind-Friendly Color Maps for R. R
    package version 0.6.5. <https://CRAN.R-project.org/package=viridis>
