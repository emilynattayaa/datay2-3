Final-report
================
Emily Sullivan
10/08/2020

``` r
library(tidyverse)
library(dplyr)
```

Q3. Explore how the household composition varies across ethnic groups in Britain?

In order to address this question, we must look at the Understanding Society data and analyse specific data sets within it. We will be looking at the gender of the individuals residing in the household as part of exploring the individual data responses, as well as the ethnicity of the individuals residing in the household and whether the individuals in the household are of the same or different gender as part of an examination into the household level data responses.

``` r
Egoalt8 <- read_tsv("/Users/emilysullivan/Lab sessions/UKDA-6614-tab/tab/ukhls_w8/h_egoalt.tab")
Hh8 <- read_tsv("/Users/emilysullivan/Lab sessions/UKDA-6614-tab/tab/ukhls_w8/h_hhresp.tab")
Ind8 <- read_tsv("/Users/emilysullivan/Lab sessions/UKDA-6614-tab/tab/ukhls_wx/xwavedat.tab")
```

Grouping and isolating parts of the data will make it easier to look at and cross examine the household composition and how it varies across different ethnicity groups in Britain. In egoalt8, we want to keep the relationship of individuals in the household along with their sex whilst sorting into groups those that are in married or cohabiting and those that are single or not cohabiting.

``` r
Partners8 <- Egoalt8 %>%
        filter(h_relationship_dv >0, h_relationship_dv < 3) %>%
        select(pidp, apidp, h_hidp, h_relationship_dv, h_sex, h_asex)

NotPartners8 <- Egoalt8 %>%
        filter(h_relationship_dv >4) %>%
        select(pidp, apidp, h_hidp, h_relationship_dv, h_sex, h_asex)

Ethn <- Ind8 %>%
        select(pidp, racel_dv)

Ethn <- Ethn %>%
        mutate(racel_dv = recode(racel_dv, `-9` = NA_real_))

Ethn <- Ethn %>%
        mutate(race = case_when(
          between(racel_dv, 1, 4) ~ "White",
          racel_dv > 4 ~ "non-White"
        ))

Hetero8 <- Partners8 %>%
        filter(h_sex != h_asex) %>%
        filter(h_sex == 2)

HeteroNonP8 <- NotPartners8 %>%
        filter(h_sex != h_asex) %>%
        filter(h_sex == 2)

JoinedEthn <- Hetero8 %>%
        left_join(Ethn, by = "pidp") 

JoinedEthn <- JoinedEthn %>%
        rename(egoRacel_dv = racel_dv) %>%
        rename(egoRace = race)

JoinedEthn <- JoinedEthn %>%
        left_join(Ethn,by = c("apidp" = "pidp"))

JoinedEthn <- JoinedEthn %>%
        rename(alterRacel_dv = racel_dv) %>%
        rename(alterRace = race)

TableRace <- JoinedEthn %>%
  filter(!is.na(egoRace) & !is.na(alterRace)) %>%
  count(egoRace, alterRace)
TableRace
```

    ## # A tibble: 4 x 3
    ##   egoRace   alterRace     n
    ##   <chr>     <chr>     <int>
    ## 1 non-White non-White  1790
    ## 2 non-White White       326
    ## 3 White     non-White   266
    ## 4 White     White      9694

This table created above shows ethnic endogamy between heterosexual couples and whether they are white or non-white. We can see that 11, 484 cohabiting partners or married couples are ethnically endogamous. This may be due to cultural stigmas surrounding interracial marriages or lack of socialisation with individuals of a different ethnicity.

In order to take this a step further, we must look at the ethnic specifics of the marriage/ cohabitation.

``` r
Ethn2 <- Ind8 %>%
        select(pidp, racel_dv)

Ethn2 <- Ethn2 %>%
        mutate(racel_dv = recode(racel_dv, `-9` = NA_real_))

Ethn2 <- Ethn2 %>%
        mutate(race = case_when(
          between(racel_dv, 1, 4) ~ "White",
          between(racel_dv, 5, 8) ~ "Mixed",
          between(racel_dv, 9, 11) ~ "South Asian",
          between(racel_dv, 12, 13) ~ "Asian",
          between(racel_dv, 14, 16) ~ "Black",
          racel_dv == 17 ~ "Arab",
          racel_dv == 97 ~ "Other"
        ))

JoinedEthn2 <- Hetero8 %>%
        full_join(Ethn2, by = "pidp") 

JoinedEthn2 <- JoinedEthn2 %>%
        rename(egoRacel_dv = racel_dv) %>%
        rename(egoRace = race)

JoinedEthn2 <- JoinedEthn2 %>%
        left_join(Ethn2,by = c("apidp" = "pidp"))

JoinedEthn2 <- JoinedEthn2 %>%
        rename(alterRacel_dv = racel_dv) %>%
        rename(alterRace = race)

TableRace2 <- JoinedEthn2 %>%
  filter(!is.na(egoRace) & !is.na(alterRace)) %>%
  count(egoRace, alterRace)
TableRace2
```

    ## # A tibble: 47 x 3
    ##    egoRace alterRace       n
    ##    <chr>   <chr>       <int>
    ##  1 Arab    Arab           30
    ##  2 Arab    Asian           3
    ##  3 Arab    Black           3
    ##  4 Arab    Mixed           2
    ##  5 Arab    South Asian     1
    ##  6 Arab    White           4
    ##  7 Asian   Arab            1
    ##  8 Asian   Asian         126
    ##  9 Asian   Black           2
    ## 10 Asian   Mixed           4
    ## # … with 37 more rows

This second table outlines the specific ethnicity of each heterosexual couple that live together in order to provide a more indepth look at the composition of a households based on their ethnic group in Britain. Here, we can identify that we have grouped the ethnicities into eight categories based on ethnic make up and geographical location.

We can identify through this table that with the exception of mixed race women, women of all other ethnicities are more likely to persue and establish marriages or cohabitations with partners of the same ethnicity or race. Mixed race women can be seen to establish marriages or cohabitations with White partners as opposed to other mixed race individuals. It is also worth noting that women identifying as "other" in their ethnic identification were in favour of White partners to cohabit with or to marry. This may be due to lesser socialisation with non-White individuals. Whites were by far the largest to display preference for an ethnically endogamous marriage or cohabitation. The large figure could be down to the survey having taken place in the United Kingdom where Whites make up a large majority of the population.

Overall, we can see that household composition varies greatly across ethnic groups in Britain with a large number of Whites and non-Whites choosing to marry or live with individuals that are of the same race or ethnicity with the largest display of this being from White people choosing to marry other white people at over 9000 individual women.
