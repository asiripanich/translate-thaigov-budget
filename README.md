ThaiGov’s 2022 Budget
================

![<https://img.shields.io/badge/open%20data-thailand-blue>](https://img.shields.io/badge/open%20data-thailand-blue)

This repository contains R code that uses [Google Translation
API](https://cloud.google.com/translate) to translate [2022 Thai
Government Budget
data](https://github.com/kaogeek/thailand-budget-pdf2csv) from Thai to
English. A *partially* translated version of the data can be viewed and
downloaded from here:

> <https://docs.google.com/spreadsheets/d/1rKR1kLuSDssT0_xLpGE_oRm2tPD5ZRhzErWq-8UzH6A/edit?usp=sharing>

So far I have only translated `ministry`, `budgetary_unit`,
`budget_plan`, `output`, `project`, `category_lv1`, `category_lv2`, and
`category_lv3` columns using my free Google monthly quota. If you are
interested to contribute, please submit a pull request with other
columns translated to English. Feel free to use the R code below. :)

| budget\_plan                                                 | budget\_plan\_en                                   | budgetary\_unit                                 | budgetary\_unit\_en                                                        | category\_lv1 | category\_lv1\_en | category\_lv2                     | category\_lv2\_en                        | category\_lv3            | category\_lv3\_en             | ministry                            | ministry\_en                            | output | output\_en | project                                                          | project\_en                                                                                       |
|:-------------------------------------------------------------|:---------------------------------------------------|:------------------------------------------------|:---------------------------------------------------------------------------|:--------------|:------------------|:----------------------------------|:-----------------------------------------|:-------------------------|:------------------------------|:------------------------------------|:----------------------------------------|:-------|:-----------|:-----------------------------------------------------------------|:--------------------------------------------------------------------------------------------------|
| แผนงานยุทธศาสตร์จัดการผลกระทบจากการเปลี่ยนแปลงสภาวะภูมิอากาศ | Climate Change Impact Strategic Work Plan          | สำนักงานปลัดกระทรวงดิจิทัลเพื่อเศรษฐกิจและสังคม | Office of the Permanent Secretary, Ministry of Digital Economy and Society | งบลงทุน       | investment budget | ค่าครุภัณฑ์ ที่ดินและสิ่งก่อสร้าง | Cost of equipment, land and construction | ค่าครุภัณฑ์              | cost of equipment             | กระทรวงดิจิทัลเพื่อเศรษฐกิจและสังคม | Ministry of Digital Economy and Society | NA     | NA         | โครงการจัดหาเครื่องวัดลมเฉือน (Lidar) และเครื่องมือ ตรวจลมชั้นบน | Project for procurement of wind shear gauges (lidar) and instruments for wind monitoring upstairs |
| แผนงานบูรณาการสร้างรายได้จากการท่องเที่ยว                    | Integrated program to generate income from tourism | กรมโยธาธิการและผังเมือง                         | Department of Public Works and Town & Country Planning                     | งบลงทุน       | investment budget | ค่าครุภัณฑ์ ที่ดินและสิ่งก่อสร้าง | Cost of equipment, land and construction | ค่าที่ดินและสิ่งก่อสร้าง | cost of land and construction | กระทรวงมหาดไทย                      | Ministry of Interior                    | NA     | NA         | โครงการพัฒนาปัจจัยพื้นฐานด้านการท่องเที่ยว                       | Tourism Fundamentals Development Project                                                          |

![](README_files/figure-gfm/total-budget-by-ministry-1.png)<!-- -->

# Targets

``` r
options(tidyverse.quiet = TRUE)
tar_option_set(packages = c("dplyr", "ggplot2", "googlesheets4", "tidyr", "skimr", "magrittr", "googleLanguageR", "data.table"))
merge_translation <- function(x, translation, column) {
  translation %>%
    merge(x, ., by.x = column, by.y = "text") %>%
    select(-detectedSourceLanguage) %>%
    rename_with( ~ gsub("translatedText", paste0(column, "_en"), .x, fixed = TRUE))
}
#> Established _targets.R and _targets_r/globals/unnamed-chunk-4.R.
```

``` r
tar_target(budget_raw, {
  read_sheet("https://docs.google.com/spreadsheets/d/1yyWXSTbq3CD_gNxks-krcSBzbszv3c_2Nq54lckoQ24/edit#gid=1625073248")
})
#> Defined target budget_raw automatically from chunk code.
#> Established _targets.R and _targets_r/targets/budget_raw.R.
```

``` r
list(
  tar_target(budget,
    budget_raw %>%
      janitor::clean_names()
  ),
  tar_target(unique_sentences, {
    budget %>%
      select(
        ministry,
        budgetary_unit,
        budget_plan,
        output,
        project,
        starts_with("category"),
        item_description
      ) %>% {
        as.character(unique(unlist(.)))
      }
  })
)
#> Established _targets.R and _targets_r/targets/data-prep.R.
```

``` r
list(
  tar_target(translated_ministry, {
    budget$ministry %>%
      unique() %>%
      gl_translate(target = "en")
  }),
  tar_target(translated_budgetary_unit, {
    budget$budgetary_unit %>%
      unique() %>%
      gl_translate(target = "en")
  }),
  tar_target(translated_budget_plan, {
    budget$budget_plan %>%
      unique() %>%
      gl_translate(target = "en")
  }),
  tar_target(translated_project, {
    budget$project %>%
      unique() %>%
      gl_translate(target = "en")
  }),
  tar_target(translated_output, {
    budget$output %>%
      unique() %>%
      gl_translate(target = "en")
  }),
  tar_target(translated_category_lv1, {
    budget$category_lv1 %>%
      unique() %>%
      gl_translate(target = "en")
  }),
  tar_target(translated_category_lv2, {
    budget$category_lv2 %>%
      unique() %>%
      gl_translate(target = "en")
  }),
  tar_target(translated_category_lv3, {
    budget$category_lv2 %>%
      unique() %>%
      gl_translate(target = "en")
  })
)
#> Established _targets.R and _targets_r/targets/translation.R.
```

``` r
tar_target(translated_budget, {
  budget %>%
    merge_translation(translated_ministry, "ministry") %>%
    merge_translation(translated_budgetary_unit, "budgetary_unit") %>%
    merge_translation(translated_budget_plan, "budget_plan") %>%
    merge_translation(translated_output, "output") %>%
    merge_translation(translated_project, "project") %>%
    merge_translation(translated_category_lv1, "category_lv1") %>%
    merge_translation(translated_category_lv2, "category_lv2") %>%
    merge_translation(translated_category_lv3, "category_lv3") %>%
    select(names(budget), everything())
})
#> Defined target translated_budget automatically from chunk code.
#> Established _targets.R and _targets_r/targets/translated_budget.R.
```

``` r
tar_target(upload_to_gsheet, {
  # googlesheets4::gs4_create("65_thailand_budget_extracted_b4_cleansing_with_ENlang",
  #            sheets = list(DATA = head(translated_budget, 10)))
  googlesheets4::sheet_write(
    data = translated_budget,
    ss = "https://docs.google.com/spreadsheets/d/1rKR1kLuSDssT0_xLpGE_oRm2tPD5ZRhzErWq-8UzH6A/edit?usp=sharing",
    sheet = "DATA"
  )
})
#> Defined target upload_to_gsheet automatically from chunk code.
#> Established _targets.R and _targets_r/targets/upload_to_gsheet.R.
```

# Pipeline

``` r
tar_make()
#> ✓ skip target budget_raw
#> ✓ skip target budget
#> ✓ skip target translated_output
#> ✓ skip target translated_budget_plan
#> ✓ skip target translated_category_lv1
#> ✓ skip target translated_category_lv2
#> ✓ skip target translated_category_lv3
#> ✓ skip target unique_sentences
#> ✓ skip target translated_ministry
#> ✓ skip target translated_budgetary_unit
#> ✓ skip target translated_project
#> ✓ skip target translated_budget
#> ✓ skip target upload_to_gsheet
#> ✓ skip pipeline
```

``` r
tar_visnetwork()
```

<img src="README_files/figure-gfm/target-visnetwork-1.png" width="100%" />

# Data summary

``` r
library(targets)
library(skimr)
```

``` r
tar_read(budget) %>%
  skimr::skim()
```

|                                                  |            |
|:-------------------------------------------------|:-----------|
| Name                                             | Piped data |
| Number of rows                                   | 51767      |
| Number of columns                                | 20         |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_   |            |
| Column type frequency:                           |            |
| character                                        | 13         |
| logical                                          | 4          |
| numeric                                          | 3          |
| \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ |            |
| Group variables                                  | None       |

Data summary

**Variable type: character**

| skim\_variable    | n\_missing | complete\_rate | min | max | empty | n\_unique | whitespace |
|:------------------|-----------:|---------------:|----:|----:|------:|----------:|-----------:|
| item\_id          |          0 |           1.00 |  10 |  17 |     0 |     51767 |          0 |
| ref\_doc          |          0 |           1.00 |   8 |  12 |     0 |        24 |          0 |
| ministry          |          0 |           1.00 |   6 |  99 |     0 |        36 |          0 |
| budgetary\_unit   |          0 |           1.00 |   6 |  79 |     0 |       718 |          0 |
| budget\_plan      |          0 |           1.00 |  19 | 754 |     0 |        99 |          0 |
| output            |      26898 |           0.48 |  11 | 196 |     0 |       371 |          0 |
| project           |      27073 |           0.48 |  19 | 230 |     0 |      1351 |          0 |
| category\_lv1     |          3 |           1.00 |   2 |  72 |     0 |        31 |          0 |
| category\_lv2     |        122 |           1.00 |   7 | 354 |     0 |      1777 |          0 |
| category\_lv3     |      13654 |           0.74 |   7 | 348 |     0 |      1395 |          0 |
| category\_lv4     |      29904 |           0.42 |   8 | 118 |     0 |       217 |          0 |
| item\_description |       5429 |           0.90 |   8 | 466 |     0 |     19806 |          0 |
| debug\_log        |      50733 |           0.02 | 167 | 847 |     0 |       767 |          0 |

**Variable type: logical**

| skim\_variable | n\_missing | complete\_rate | mean | count                  |
|:---------------|-----------:|---------------:|-----:|:-----------------------|
| cross\_func    |          0 |              1 | 0.15 | FAL: 43971, TRU: 7796  |
| category\_lv5  |      51767 |              0 |  NaN | :                      |
| category\_lv6  |      51767 |              0 |  NaN | :                      |
| obliged        |          0 |              1 | 0.32 | FAL: 35279, TRU: 16488 |

**Variable type: numeric**

| skim\_variable | n\_missing | complete\_rate |        mean |           sd |    p0 |     p25 |     p50 |      p75 |      p100 | hist  |
|:---------------|-----------:|---------------:|------------:|-------------:|------:|--------:|--------:|---------:|----------:|:------|
| ref\_page\_no  |          0 |              1 |      304.33 | 1.951500e+02 |     5 |     147 |     279 |      429 | 9.190e+02 | ▇▇▅▂▁ |
| fiscal\_year   |          0 |              1 |     2021.73 | 1.445000e+01 |  -287 |    2022 |    2022 |     2022 | 2.066e+03 | ▁▁▁▁▇ |
| amount         |         43 |              1 | 97021729.28 | 2.218307e+09 | -2551 | 1188000 | 5736000 | 21816175 | 3.106e+11 | ▇▁▁▁▁ |
