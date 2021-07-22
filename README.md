ThaiGov’s 2022 Budget
================

<blockquote class="twitter-tweet">
<p lang="th" dir="ltr">
\[ Excel
<a href="https://twitter.com/hashtag/%E0%B8%87%E0%B8%9A65?src=hash&amp;ref_src=twsrc%5Etfw">\#งบ65</a>
สำเร็จแล้ว! ใช้ไปเลยชั่วลูกชั่วหลาน \]<br><br>ดาวน์โหลด:
<a href="https://t.co/PuhflF8DEF">https://t.co/PuhflF8DEF</a><br><br>เราจะทำตามสัญญา
ขอเวลาอีกไม่นาน …<br><br>หลัง งปม. วาระ 1 ผ่านสภา
พวกเราเคยให้สัญญากับทุกท่านไว้ว่า จะรวมพลังสาย dev เพื่อแปลงงบ pdf สู่
machine-readable แบบทำครั้งเดียว ใช้ได้ชั่วลูกชั่วหลาน
<a href="https://t.co/b0pWMF3jDc">pic.twitter.com/b0pWMF3jDc</a>
</p>
— ณัฐพงษ์ เรืองปัญญาวุฒิ (เท้ง) (@teng\_mfp)
<a href="https://twitter.com/teng_mfp/status/1417872910383865859?ref_src=twsrc%5Etfw">July
21, 2021</a>
</blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

This repository contains R code that uses [Google Translation
API](https://cloud.google.com/translate) to translate [2022 Thai
Government Budget
data](https://github.com/kaogeek/thailand-budget-pdf2csv) from Thai to
English. A translated version of the data can be viewed and downloaded
from here:

> <https://docs.google.com/spreadsheets/d/1rKR1kLuSDssT0_xLpGE_oRm2tPD5ZRhzErWq-8UzH6A/edit?usp=sharing>

So far I have only translated `ministry`, `budgetary_unit`,
`budget_plan`, `output`, and `category_lv1` columns using my free Google
monthly quota. If you are interested to contribute, please submit a pull
request with other columns translated to English. Feel free to use the R
code below. :)

``` r
options(tidyverse.quiet = TRUE)
tar_option_set(packages = c("dplyr", "ggplot2", "googlesheets4", "tidyr", "skimr", "magrittr", "googleLanguageR", "data.table"))
merge_translation <- function(x, translation, column) {
  translation %>%
    merge(x, ., by.x = column, by.y = "text") %>%
    select(-detectedSourceLanguage) %>%
    rename_with( ~ gsub("translatedText", paste0(column, "_en"), .x, fixed = TRUE))
}
#> Established _targets.R and _targets_r/globals/unnamed-chunk-3.R.
```

# Targets

``` r
tar_target(budget_raw, {
  read_sheet("https://docs.google.com/spreadsheets/d/1yyWXSTbq3CD_gNxks-krcSBzbszv3c_2Nq54lckoQ24/edit#gid=1625073248")
})
#> Defined target budget_raw automatically from chunk code.
#> Established _targets.R and _targets_r/targets/budget_raw.R.
```

``` r
list(
  tar_target(budget, budget_raw %>% janitor::clean_names()),
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

<!-- ```{targets chunks_of_unique_sentences, tar_simple = TRUE} -->
<!-- unique_sentences %>% -->
<!--   sample(size = 20, replace = FALSE) %>% -->
<!--   split(., ceiling(seq_along(.) / 5)) -->
<!-- ``` -->

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
  tar_target(translated_output, {
    budget$output %>%
      unique() %>%
      gl_translate(target = "en")
  }),
  tar_target(translated_category_lv1, {
    budget$category_lv1 %>%
      unique() %>%
      gl_translate(target = "en")
  })
)
#> Established _targets.R and _targets_r/targets/translation.R.
```

<!-- ```{targets translate-by-chunk}  -->
<!-- list( -->
<!--   tar_target( -->
<!--     translated_chucks,  -->
<!--     gl_translate(chunks_of_unique_sentences[[1]], target = "en"),  -->
<!--     pattern = map(chunks_of_unique_sentences),  -->
<!--     iteration = "list" -->
<!--   ) -->
<!-- ) -->
<!-- ``` -->

``` r
tar_target(translated_budget, {
  budget %>%
    merge_translation(translated_ministry, "ministry") %>%
    merge_translation(translated_budgetary_unit, "budgetary_unit") %>%
    merge_translation(translated_budget_plan, "budget_plan") %>%
    merge_translation(translated_output, "output") %>%
    merge_translation(translated_category_lv1, "category_lv1") %>%
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
#> ✓ skip target unique_sentences
#> ✓ skip target translated_ministry
#> ✓ skip target translated_budgetary_unit
#> ✓ skip target translated_budget
#> • start target upload_to_gsheet
#> ✓ Successfully auto-authenticated via /home/amarin/erudite-mote-320609-e5378672f621.json
#> ! Using an auto-discovered, cached token.
#>   To suppress this message, modify your code or options to clearly consent to
#>   the use of a cached token.
#>   See gargle's "Non-interactive auth" vignette for more details:
#>   <https://gargle.r-lib.org/articles/non-interactive-auth.html>
#> ℹ The googlesheets4 package is using a cached token for
#>   'rciti.emission.ec2@gmail.com'.
#> ✓ Writing to "65_thailand_budget_extracted_b4_cleansing_with_ENlang".
#> ✓ Writing to sheet 'DATA'.
#> • built target upload_to_gsheet
#> • end pipeline
```

``` r
tar_visnetwork()
```

![](README_files/figure-gfm/visnetwork-1.png)<!-- -->

# Output

``` r
library(dplyr)
library(skimr)
library(ggplot2)
library(kableExtra)
library(stringr)
library(forcats)
library(scales)
```

``` r
tar_read(unique_sentences) %>%
  head()
#> [1] "งบกลาง"                  "สำนักนายกรัฐมนตรี"         
#> [3] "กระทรวงกลาโหม"           "กระทรวงการคลัง"          
#> [5] "กระทรวงการต่างประเทศ"     "กระทรวงการท่องเที่ยวและกีฬา"
```

``` r
tar_load(translated_budget)
cols_to_select <- names(translated_budget)[grepl("_en", names(translated_budget))]
cols_to_select <- c(gsub("_en", "", cols_to_select), cols_to_select) %>% sort()

translated_budget %>%
  dplyr::select(cols_to_select) %>%
  dplyr::slice_sample(n = 10) %>%
  kableExtra::kbl() 
#> Note: Using an external vector in selections is ambiguous.
#> ℹ Use `all_of(cols_to_select)` instead of `cols_to_select` to silence this message.
#> ℹ See <https://tidyselect.r-lib.org/reference/faq-external-vector.html>.
#> This message is displayed once per session.
```

<table>
<thead>
<tr>
<th style="text-align:left;">
budget\_plan
</th>
<th style="text-align:left;">
budget\_plan\_en
</th>
<th style="text-align:left;">
budgetary\_unit
</th>
<th style="text-align:left;">
budgetary\_unit\_en
</th>
<th style="text-align:left;">
category\_lv1
</th>
<th style="text-align:left;">
category\_lv1\_en
</th>
<th style="text-align:left;">
ministry
</th>
<th style="text-align:left;">
ministry\_en
</th>
<th style="text-align:left;">
output
</th>
<th style="text-align:left;">
output\_en
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
แผนงานยุทธศาสตร์เสริมสร้างความมั่นคงของสถาบันหลักของชาติ
</td>
<td style="text-align:left;">
Strategic work plan to strengthen the security of the main national
institutions
</td>
<td style="text-align:left;">
สำนักงานตำรวจแห่งชาติ
</td>
<td style="text-align:left;">
Royal Thai Police
</td>
<td style="text-align:left;">
งบดำเนินงาน
</td>
<td style="text-align:left;">
operating budget
</td>
<td style="text-align:left;">
ส่วนราชการไม่สังกัดสำนักนายกรัฐมนตรี กระทรวง หรือทบวง
และหน่วยงานภายใต้การควบคุมดูแลของนายกรัฐมนตรี
</td>
<td style="text-align:left;">
Government agencies not affiliated with the Prime Minister’s Office,
Ministries or Ta-buangs and agencies under the supervision of the Prime
Minister
</td>
<td style="text-align:left;">
NA
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานพื้นฐานด้านการสร้างความสามารถในการแข่งขัน
</td>
<td style="text-align:left;">
Fundamental Plan for Competitiveness
</td>
<td style="text-align:left;">
กรมเจ้าท่า
</td>
<td style="text-align:left;">
Marine Department
</td>
<td style="text-align:left;">
งบดำเนินงาน
</td>
<td style="text-align:left;">
operating budget
</td>
<td style="text-align:left;">
กระทรวงคมนาคม
</td>
<td style="text-align:left;">
Ministry of Transport
</td>
<td style="text-align:left;">
การพัฒนาและบำรุงรักษาโครงสร้างพื้นฐานด้านการขนส่งทางน้ำ
</td>
<td style="text-align:left;">
Development and maintenance of water transport infrastructure
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานบุคลากรภาครัฐ
</td>
<td style="text-align:left;">
Government Personnel Program
</td>
<td style="text-align:left;">
กรมที่ดิน
</td>
<td style="text-align:left;">
Department of Lands
</td>
<td style="text-align:left;">
งบบุคลากร
</td>
<td style="text-align:left;">
personnel budget
</td>
<td style="text-align:left;">
กระทรวงมหาดไทย
</td>
<td style="text-align:left;">
Ministry of Interior
</td>
<td style="text-align:left;">
NA
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานยุทธศาสตร์ส่งเสริมการพัฒนาจังหวัดและกลุ่มจังหวัดแบบบูรณาการ
</td>
<td style="text-align:left;">
Strategic work plan to promote the development of provinces and
integrated provincial groups
</td>
<td style="text-align:left;">
จังหวัดน่าน
</td>
<td style="text-align:left;">
Nan Province
</td>
<td style="text-align:left;">
งบรายจ่ายอื่น
</td>
<td style="text-align:left;">
Other expenditures
</td>
<td style="text-align:left;">
จังหวัดและกลุ่มจังหวัด (2)
</td>
<td style="text-align:left;">
Provinces and provincial groups (2)
</td>
<td style="text-align:left;">
NA
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานพื้นฐานด้านการสร้างความสามารถในการแข่งขัน
</td>
<td style="text-align:left;">
Fundamental Plan for Competitiveness
</td>
<td style="text-align:left;">
กรมการขนส่งทางราง
</td>
<td style="text-align:left;">
Department of Rail Transport
</td>
<td style="text-align:left;">
งบดำเนินงาน
</td>
<td style="text-align:left;">
operating budget
</td>
<td style="text-align:left;">
กระทรวงคมนาคม
</td>
<td style="text-align:left;">
Ministry of Transport
</td>
<td style="text-align:left;">
นโยบายและแผนการพัฒนาด้านการขนส่งทางราง และมาตรฐานเกี่ยวกับการขนส่งทางราง
</td>
<td style="text-align:left;">
Policies and plans for the development of rail transport and standards
on rail transport
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานพื้นฐานด้านการสร้างโอกาสและความเสมอภาคทางสังคม
</td>
<td style="text-align:left;">
Fundamentals of Opportunity Building and Social Equality
</td>
<td style="text-align:left;">
สำนักงานปลัดกระทรวงวัฒนธรรม
</td>
<td style="text-align:left;">
Office of the Permanent Secretary, Ministry of Culture
</td>
<td style="text-align:left;">
งบดำเนินงาน
</td>
<td style="text-align:left;">
operating budget
</td>
<td style="text-align:left;">
กระทรวงวัฒนธรรม
</td>
<td style="text-align:left;">
Ministry of Culture
</td>
<td style="text-align:left;">
นโยบาย แผนด้านศิลปะ ศาสนา และวัฒนธรรม ได้รับการบริหารจัดการ
</td>
<td style="text-align:left;">
Policies, plans on arts, religion and culture are managed.
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานยุทธศาสตร์พัฒนาศักยภาพการป้องกันประเทศและความพร้อมเผชิญภัยคุกคามทุกมิติ
</td>
<td style="text-align:left;">
Strategic plans to develop national defense capabilities and readiness
to face threats in all dimensions
</td>
<td style="text-align:left;">
กองทัพบก
</td>
<td style="text-align:left;">
army
</td>
<td style="text-align:left;">
งบรายจ่ายอื่น
</td>
<td style="text-align:left;">
Other expenditures
</td>
<td style="text-align:left;">
กระทรวงกลาโหม
</td>
<td style="text-align:left;">
Ministry of Defense
</td>
<td style="text-align:left;">
NA
</td>
<td style="text-align:left;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานยุทธศาสตร์ส่งเสริมการกระจายอำนาจให้แก่องค์กรปกครองส่วนท้องถิ่น
</td>
<td style="text-align:left;">
Strategic plans to promote decentralization to local government
organizations
</td>
<td style="text-align:left;">
เทศบาลนครในพื้นที่จังหวัดตรัง เทศบาลนครตรัง
</td>
<td style="text-align:left;">
Municipality in Trang Province Trang
</td>
<td style="text-align:left;">
งบเงินอุดหนุน
</td>
<td style="text-align:left;">
subsidy budget
</td>
<td style="text-align:left;">
องค์กรปกครองส่วนท้องถิ่น
</td>
<td style="text-align:left;">
local government organization
</td>
<td style="text-align:left;">
ผลผลิตการจัดบริการสาธารณะ
</td>
<td style="text-align:left;">
Product of Public Service Arrangement
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานพื้นฐานด้านการพัฒนาและเสริมสร้างศักยภาพทรัพยากรมนุษย์
</td>
<td style="text-align:left;">
Fundamental Plan for Human Resources Development and Enhancement
</td>
<td style="text-align:left;">
กรมอนามัย
</td>
<td style="text-align:left;">
Department of Health
</td>
<td style="text-align:left;">
งบดำเนินงาน
</td>
<td style="text-align:left;">
operating budget
</td>
<td style="text-align:left;">
กระทรวงสาธารณสุข
</td>
<td style="text-align:left;">
Ministry of Health
</td>
<td style="text-align:left;">
พัฒนาและบริการการส่งเสริมสุขภาพและอนามัยสิ่งแวดล้อม ที่เหมาะสมกับประชาชน
ชุมชนและภาคีเครือข่าย
</td>
<td style="text-align:left;">
Developing and promoting health and environmental health services
suitable for the people Community and network partners
</td>
</tr>
<tr>
<td style="text-align:left;">
แผนงานยุทธศาสตร์การเกษตรสร้างมูลค่า
</td>
<td style="text-align:left;">
Value-Creating Agriculture Strategic Work Plan
</td>
<td style="text-align:left;">
กรมประมง
</td>
<td style="text-align:left;">
Department of Fisheries
</td>
<td style="text-align:left;">
งบดำเนินงาน
</td>
<td style="text-align:left;">
operating budget
</td>
<td style="text-align:left;">
กระทรวงเกษตรและสหกรณ์
</td>
<td style="text-align:left;">
Ministry of Agriculture and Cooperatives
</td>
<td style="text-align:left;">
NA
</td>
<td style="text-align:left;">
NA
</td>
</tr>
</tbody>
</table>

``` r
tar_read(budget_en) %>%
  mutate(ministry_en = ifelse(grepl("Ministry of Higher Education", ministry_en) == TRUE, 
                               "Ministry of Higher Education, Science, Research and Innovation",
                               ministry_en)) %>%
  group_by(ministry_en) %>%
  summarise(amount = sum(amount, na.rm = TRUE)) %>%
  ggplot(data = ., aes(
    x = amount,
    y = forcats::fct_reorder(stringr::str_wrap(ministry_en, 40), amount)
  )) +
  geom_col() +
  scale_x_continuous(labels = unit_format(unit = "M", scale = 1e-6, big.mark = ",")) +
  scale_y_discrete(expand = c(0, 0)) +
  theme_bw(base_size = 12) +
  theme(axis.text.x = element_text(angle = 90, hjust = 1)) +
  labs(x = "Amount in Million Thai Bahts", y = "Ministry")
```

![](README_files/figure-gfm/total-budget-by-ministry-1.png)<!-- -->

``` r
tar_read(budget) %>%
  skimr::skim()
```

<table style="width: auto;" class="table table-condensed">
<caption>
Data summary
</caption>
<thead>
<tr>
<th style="text-align:left;">
</th>
<th style="text-align:left;">
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Name
</td>
<td style="text-align:left;">
Piped data
</td>
</tr>
<tr>
<td style="text-align:left;">
Number of rows
</td>
<td style="text-align:left;">
51767
</td>
</tr>
<tr>
<td style="text-align:left;">
Number of columns
</td>
<td style="text-align:left;">
20
</td>
</tr>
<tr>
<td style="text-align:left;">
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
</td>
<td style="text-align:left;">
</td>
</tr>
<tr>
<td style="text-align:left;">
Column type frequency:
</td>
<td style="text-align:left;">
</td>
</tr>
<tr>
<td style="text-align:left;">
character
</td>
<td style="text-align:left;">
13
</td>
</tr>
<tr>
<td style="text-align:left;">
logical
</td>
<td style="text-align:left;">
4
</td>
</tr>
<tr>
<td style="text-align:left;">
numeric
</td>
<td style="text-align:left;">
3
</td>
</tr>
<tr>
<td style="text-align:left;">
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
</td>
<td style="text-align:left;">
</td>
</tr>
<tr>
<td style="text-align:left;">
Group variables
</td>
<td style="text-align:left;">
None
</td>
</tr>
</tbody>
</table>

**Variable type: character**

<table>
<thead>
<tr>
<th style="text-align:left;">
skim\_variable
</th>
<th style="text-align:right;">
n\_missing
</th>
<th style="text-align:right;">
complete\_rate
</th>
<th style="text-align:right;">
min
</th>
<th style="text-align:right;">
max
</th>
<th style="text-align:right;">
empty
</th>
<th style="text-align:right;">
n\_unique
</th>
<th style="text-align:right;">
whitespace
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
item\_id
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1.00
</td>
<td style="text-align:right;">
10
</td>
<td style="text-align:right;">
17
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
51767
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
ref\_doc
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1.00
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
12
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
24
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
ministry
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1.00
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
99
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
36
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
budgetary\_unit
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1.00
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
79
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
718
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
budget\_plan
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1.00
</td>
<td style="text-align:right;">
19
</td>
<td style="text-align:right;">
754
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
99
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
output
</td>
<td style="text-align:right;">
26898
</td>
<td style="text-align:right;">
0.48
</td>
<td style="text-align:right;">
11
</td>
<td style="text-align:right;">
196
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
371
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
project
</td>
<td style="text-align:right;">
27073
</td>
<td style="text-align:right;">
0.48
</td>
<td style="text-align:right;">
19
</td>
<td style="text-align:right;">
230
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1351
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
category\_lv1
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
1.00
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
72
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
31
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
category\_lv2
</td>
<td style="text-align:right;">
122
</td>
<td style="text-align:right;">
1.00
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:right;">
354
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1777
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
category\_lv3
</td>
<td style="text-align:right;">
13654
</td>
<td style="text-align:right;">
0.74
</td>
<td style="text-align:right;">
7
</td>
<td style="text-align:right;">
348
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1395
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
category\_lv4
</td>
<td style="text-align:right;">
29904
</td>
<td style="text-align:right;">
0.42
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
118
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
217
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
item\_description
</td>
<td style="text-align:right;">
5429
</td>
<td style="text-align:right;">
0.90
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
466
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
19806
</td>
<td style="text-align:right;">
0
</td>
</tr>
<tr>
<td style="text-align:left;">
debug\_log
</td>
<td style="text-align:right;">
50733
</td>
<td style="text-align:right;">
0.02
</td>
<td style="text-align:right;">
167
</td>
<td style="text-align:right;">
847
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
767
</td>
<td style="text-align:right;">
0
</td>
</tr>
</tbody>
</table>

**Variable type: logical**

<table>
<thead>
<tr>
<th style="text-align:left;">
skim\_variable
</th>
<th style="text-align:right;">
n\_missing
</th>
<th style="text-align:right;">
complete\_rate
</th>
<th style="text-align:right;">
mean
</th>
<th style="text-align:left;">
count
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
cross\_func
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
0.15
</td>
<td style="text-align:left;">
FAL: 43971, TRU: 7796
</td>
</tr>
<tr>
<td style="text-align:left;">
category\_lv5
</td>
<td style="text-align:right;">
51767
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
NaN
</td>
<td style="text-align:left;">
:
</td>
</tr>
<tr>
<td style="text-align:left;">
category\_lv6
</td>
<td style="text-align:right;">
51767
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
NaN
</td>
<td style="text-align:left;">
:
</td>
</tr>
<tr>
<td style="text-align:left;">
obliged
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
0.32
</td>
<td style="text-align:left;">
FAL: 35279, TRU: 16488
</td>
</tr>
</tbody>
</table>

**Variable type: numeric**

<table>
<thead>
<tr>
<th style="text-align:left;">
skim\_variable
</th>
<th style="text-align:right;">
n\_missing
</th>
<th style="text-align:right;">
complete\_rate
</th>
<th style="text-align:right;">
mean
</th>
<th style="text-align:right;">
sd
</th>
<th style="text-align:right;">
p0
</th>
<th style="text-align:right;">
p25
</th>
<th style="text-align:right;">
p50
</th>
<th style="text-align:right;">
p75
</th>
<th style="text-align:right;">
p100
</th>
<th style="text-align:left;">
hist
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
ref\_page\_no
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
304.33
</td>
<td style="text-align:right;">
1.951500e+02
</td>
<td style="text-align:right;">
5
</td>
<td style="text-align:right;">
147
</td>
<td style="text-align:right;">
279
</td>
<td style="text-align:right;">
429
</td>
<td style="text-align:right;">
9.190e+02
</td>
<td style="text-align:left;">
▇▇▅▂▁
</td>
</tr>
<tr>
<td style="text-align:left;">
fiscal\_year
</td>
<td style="text-align:right;">
0
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
2021.73
</td>
<td style="text-align:right;">
1.445000e+01
</td>
<td style="text-align:right;">
-287
</td>
<td style="text-align:right;">
2022
</td>
<td style="text-align:right;">
2022
</td>
<td style="text-align:right;">
2022
</td>
<td style="text-align:right;">
2.066e+03
</td>
<td style="text-align:left;">
▁▁▁▁▇
</td>
</tr>
<tr>
<td style="text-align:left;">
amount
</td>
<td style="text-align:right;">
43
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
97021729.28
</td>
<td style="text-align:right;">
2.218307e+09
</td>
<td style="text-align:right;">
-2551
</td>
<td style="text-align:right;">
1188000
</td>
<td style="text-align:right;">
5736000
</td>
<td style="text-align:right;">
21816175
</td>
<td style="text-align:right;">
3.106e+11
</td>
<td style="text-align:left;">
▇▁▁▁▁
</td>
</tr>
</tbody>
</table>
