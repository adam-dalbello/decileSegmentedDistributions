# segmentedDistributions Function
Groups a ```dimension``` by its own quartiles (the thresholds for each group being p25, p50 and p75) then outputs an aggregate of a ```metric```. ```dimension``` must be either an integer or numeric in class.

Matrices, data.tables and dataframes will be cast as tibbles. Variables passed through ```dimension``` and ```metric``` must be numeric or integers.


### Languages and Tools
<div>
  <img src="https://github.com/devicons/devicon/blob/master/icons/r/r-original.svg" title = "r" alt = "r" width = "60" height = "60"/>&nbsp;
  <img src="https://github.com/devicons/devicon/blob/master/icons/rstudio/rstudio-original.svg" title = "RStudio" alt = "RStudio" width = "60" height = "60"/>&nbsp;
</div>

### Packages
<div>
  <img src="https://github.com/tidyverse/dplyr/raw/main/man/figures/logo.png" height = "100" style = "max-width: 100%;"/>&nbsp;
  <img src="https://github.com/tidyverse/rlang/raw/main/man/figures/logo.png" height = "100" style = "max-width: 100%;"/>&nbsp;
</div>
<br>
<br>

### Arguments
| Argument | Description |
| --- | --- |
| ```.data``` | A data frame, tibble, matrix, or data table. Will be cast as a tibble internally. |
| ```dimension``` | The variable/column/field to segment. |
| ```metric``` | The variable/column/field distributions will be printed out for. |
| ```na.rm``` | Determines if NA values are to be included. By default ignores NA values. |

# Function
```r
segmentedDistributions <- function(.data, dimension, metric, na.rm = TRUE) {
  require(dplyr)
  require(rlang)
  
  .data <- as_tibble(.data)
  
  if (is.numeric(.data %>% select( {{ dimension }}, {{ metric }} ) %>% as.matrix() )  )   {
    vector1 <- .data %>% pull( {{ dimension }} )
    thresholds <- quantile(vector1, probs = c(0.25, 0.5, 0.75), na.rm = na.rm)
    vector2 <- if_else(vector1 <= thresholds[[1]], '> p0, <= p25',
                       if_else(vector1 <= thresholds[[2]], '> p25, <= p50',
                               if_else(vector1 <= thresholds[[3]], '> p50, <= p75', '> p75, <= p100'
                                       )
                               )
                       ) %>%
      as_tibble() %>%
      rename(dimension_class := 'value') 
    
    bind_cols(.data, vector2) %>% 
      group_by(dimension_class) %>% 
      summarise(
        observations = n(),
        metric_min = min( {{ metric }}, na.rm = na.rm),
        metric_p25 = quantile( {{metric }}, prob = 0.25, na.rm = na.rm),
        metric_p50 = quantile( {{ metric }}, prob = 0.50, na.rm = na.rm),
        metric_mean = mean( {{ metric }}, .groups = 'drop', na.rm = na.rm),
        metric_p75 = quantile( {{ metric }}, prob = 0.75, na.rm = na.rm),
        metric_max = max( {{ metric }}, na.rm = na.rm)
      )
  } else {
    stop('Pass numeric dimension and metric variables. Only numeric data permissable.')
  }
  
}


# # A tibble: 3 x 8
#   dimension_class observations metric_min metric_p25 metric_p50 metric_mean metric_p75 metric_max
#   <chr>                  <int>      <dbl>      <dbl>      <dbl>       <dbl>      <dbl>      <dbl>
# 1 > p0, <= p25            6317          0          0          0        0             0          0
# 2 > p50, <= p75           4185          1          1          2        1.65          2          2
# 3 > p75, <= p100          1213          3          3          3        3             3          3
```
