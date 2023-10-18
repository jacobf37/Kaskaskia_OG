---
title: "R Notebook"
output:
  html_document:
    keep_md: true
---




```r
tb <- read_xlsx('./Data/2023 kaskaskia woods.xlsx', sheet = 'Sheet2', range = 'A1:O427') %>% 
  select(PLOT = `Plot #`, STAT_2012 = `2012 Status`, DBH_2012 = `2012 DBH (cm)`, 
         STAT_2023 = `2023 Status`, DBH_2023 = `2023 DBH (cm)`) %>% 
  mutate(PLOT = as.factor(PLOT))

tb_hist <- read_xls('./Data/Historical Data - KEFAllPlots.xls', sheet = 'KEFPLots1to8', range = 'A3:V1324') %>% 
  select(-DBH83...9, -DBH97...10) %>% 
  rename(DBH83 = DBH83...18, DBH97 = DBH97...19) %>% 
  mutate(PLOT = as.factor(PLOT),
         TPH_35 = ifelse(DBH35 == 0, 0, 9.88),
         TPH_40 = ifelse(DBH40 == 0, 0, 9.88),
         TPH_58 = ifelse(DBH58 == 0, 0, 9.88),
         TPH_64 = ifelse(DBH64 == 0, 0, 9.88),
         TPH_73 = ifelse(DBH73 == 0, 0, 9.88),
         TPH_78 = ifelse(DBH78 == 0, 0, 9.88),
         TPH_83 = ifelse(DBH83 == 0, 0, 9.88),
         TPH_97 = ifelse(DBH97 == 0, 0, 9.88))
```



```r
plot_summ <- tb_hist %>% group_by(PLOT) %>% 
  summarise(BA_1935 = sum((DBH35 * 2.54)^2 * 0.00007854 * 9.88, na.rm = T),
            BA_1940 = sum((DBH40 * 2.54)^2 * 0.00007854 * 9.88, na.rm = T),
            BA_1958 = sum((DBH58 * 2.54)^2 * 0.00007854 * 9.88, na.rm = T),
            BA_1964 = sum((DBH64 * 2.54)^2 * 0.00007854 * 9.88, na.rm = T),
            BA_1973 = sum((DBH73 * 2.54)^2 * 0.00007854 * 9.88, na.rm = T),
            BA_1978 = sum((DBH78 * 2.54)^2 * 0.00007854 * 9.88, na.rm = T),
            BA_1983 = sum((DBH83 * 2.54)^2 * 0.00007854 * 9.88, na.rm = T),
            BA_1997 = sum((DBH97 * 2.54)^2 * 0.00007854 * 9.88, na.rm = T),
            TPH_1935 = sum(TPH_35, na.rm = T),
            TPH_1940 = sum(TPH_40, na.rm = T),
            TPH_1958 = sum(TPH_58, na.rm = T),
            TPH_1964 = sum(TPH_64, na.rm = T),
            TPH_1973 = sum(TPH_73, na.rm = T),
            TPH_1978 = sum(TPH_78, na.rm = T),
            TPH_1983 = sum(TPH_83, na.rm = T),
            TPH_1997 = sum(TPH_97, na.rm = T)) %>% 
  left_join(tb %>% 
      mutate(DBH_2023 = ifelse(STAT_2023 == 'LV', DBH_2023, 0.0),
             TPH_2023 = ifelse(STAT_2023 == 'LV', 9.88, 0.0),
             TPH_2012 = 9.88) %>% 
      group_by(PLOT) %>% 
      summarise(BA_2012 = sum((DBH_2012)^2 * 0.00007854 * 9.88, na.rm = T),
                BA_2023 = sum((DBH_2023)^2 * 0.00007854 * 9.88, na.rm = T),
                TPH_2012 = sum(TPH_2012, na.rm = T),
                TPH_2023 = sum(TPH_2023, na.rm = T)),
      by = 'PLOT'
      )
```

Mean basal area

```r
plot_summ %>% 
  ungroup() %>% 
  select(-PLOT) %>% colMeans() %>% data.frame(Value = .) %>% 
  mutate(Metric = rownames(.)) %>% 
  filter(str_starts(Metric, 'BA_')) %>% 
  mutate(Year = as.integer(str_remove(Metric, pattern = 'BA_'))) %>% 
  ggplot(aes(x = Year, y = Value, group = 1)) +
  geom_line(linewidth = 1) +
  geom_point(shape = 19, size = 3) +
  scale_x_continuous(limits = c(1935, 2025), breaks = seq(from = 1935, to = 2025, by = 10)) +
  scale_y_continuous(limits = c(0,50)) +
  theme_classic()
```

![](Inventory_analysis_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

Mean stem density

```r
plot_summ %>% 
  ungroup() %>% 
  select(-PLOT) %>% colMeans() %>% data.frame(Value = .) %>% 
  mutate(Metric = rownames(.)) %>% 
  filter(str_starts(Metric, 'TPH_')) %>% 
  mutate(Year = as.integer(str_remove(Metric, pattern = 'TPH_'))) %>% 
  ggplot(aes(x = Year, y = Value, group = 1)) +
  geom_line(linewidth = 1) +
  geom_point(shape = 19, size = 3) +
  scale_x_continuous(limits = c(1935, 2025), breaks = seq(from = 1935, to = 2025, by = 10)) +
  scale_y_continuous(limits = c(0,1000)) +
  theme_classic()
```

![](Inventory_analysis_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

Plot basal area

```r
plot_summ %>% select(PLOT, starts_with('BA_')) %>% 
  pivot_longer(!PLOT, names_to = 'YEAR', values_to = 'BAH') %>% 
  mutate(YEAR = as.integer(str_remove(YEAR, pattern = 'BA_'))) %>% 
  ggplot(aes(x = YEAR, y = BAH, group = PLOT, color = PLOT)) +
    geom_line(linewidth = 1) +
    geom_point(shape = 21, size = 3) +
    scale_x_continuous(limits = c(1935, 2025), breaks = seq(from = 1935, to = 2025, by = 10)) +
    scale_y_continuous(limits = c(0,55)) +
    theme_classic()
```

![](Inventory_analysis_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

Plot stem density

```r
plot_summ %>% select(PLOT, starts_with('TPH_')) %>% 
  pivot_longer(!PLOT, names_to = 'YEAR', values_to = 'TPH') %>% 
  mutate(YEAR = as.integer(str_remove(YEAR, pattern = 'TPH_'))) %>% 
  ggplot(aes(x = YEAR, y = TPH, group = PLOT, color = PLOT)) +
    geom_line(linewidth = 1) +
    geom_point(shape = 19, size = 3) +
    scale_x_continuous(limits = c(1935, 2025), breaks = seq(from = 1935, to = 2025, by = 10)) +
    theme_classic()
```

![](Inventory_analysis_files/figure-html/unnamed-chunk-7-1.png)<!-- -->
