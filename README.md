
<!-- README.md is generated from README.Rmd. Please edit that file -->

# basicTables

> Create mediocre Excel tables in R.

As implied by the name, the objective of `basicTables` is to provide a
“good enough” approach to creating tables in R. The main use case is to
quickly share tables using the .xlsx format without much hassle.

Due to the limited approach, basicTables is a lot less sophisticated
than the awesome packages [gt](https://gt.rstudio.com/) (great tables,
by which the name is inspired),
[tables](https://dmurdoch.github.io/tables/), or
[expss](https://gdemin.github.io/expss/#Introduction). The main
advantage of basicTables is its simplicity and its focus on creating
.xslx tables.

## Example

basicTables assumes that you already have a perfectly summarized table
that you now want to share using .xlsx. That is, basicTables will not
create any crosstables, compute percentages or similar. It mostly adds
column names with spanners, combines rownames, and writes the result to
.xlsx files.

Let’s assume we want to share the following table:

``` r
library(dplyr)
data("mtcars")

summarized_table <- mtcars |>
  group_by(cyl, vs) |>
  summarise(N = n(),
            mean_hp = mean(hp),
            sd_hp = sd(hp),
            mean_wt = mean(wt),
            sd_wt = sd(wt))
#> `summarise()` has grouped output by 'cyl'. You can override using the `.groups`
#> argument.

print(summarized_table)
#> # A tibble: 5 × 7
#> # Groups:   cyl [3]
#>     cyl    vs     N mean_hp sd_hp mean_wt  sd_wt
#>   <dbl> <dbl> <int>   <dbl> <dbl>   <dbl>  <dbl>
#> 1     4     0     1    91   NA       2.14 NA    
#> 2     4     1    10    81.8 21.9     2.30  0.598
#> 3     6     0     3   132.  37.5     2.76  0.128
#> 4     6     1     4   115.   9.18    3.39  0.116
#> 5     8     0    14   209.  51.0     4.00  0.759
```

Now we want to create a table where we show the grouping variables as
row names and also create spanners for the horse power (`hp`) and the
weight (`wt`) variables. The result should look something like this:

    |                   | Horse Power |   Weight  |
    | Cylinder | Engine | Mean  |  SD | Mean | SD |
    | -------- | ------ | ----- | --- | ---- | -- |
    |                   |                         |

In basicTables, the table headers are defined with a formula approach
inspired by the [tables](https://dmurdoch.github.io/tables/) package.
For example, `cyl ~ mean_hp + sd_hp` defines a table with `cyl` as the
row names and `mean_hp` and `sd_hp` as columns. The output in Excel will
be similar to the following:

    |cyl | mean_hp | sd_hp|
    |:---|---------------:|
    |4   | 91          NA |
    |4   | 81.8      21.9 |

Note that the row names (`cyl`) are in a separate block to the left.

Spanners are defined using braces and spanner names. For example, the
following defines a spanner for `mean_hp` and `sd_hp` with the name
`Horsepower`: `cyl ~ (Horsepower = mean_hp + sd_hp)`.

    |    |    Horsepower  |
    |cyl | mean_hp | sd_hp|
    |:---|---------------:|
    |4   | 91          NA |
    |4   | 81.8      21.9 |

Spanners can also be nested (e.g.,
`cyl ~ (Horsepower = (Mean = mean_hp) + (SD  = sd_hp))`.

A limitation of the table above is that the names in the table - `cyl`,
`mean_hp`, and `sd_hp` - are identical to the names of the variables in
R. They include redundant information as the spanner `Horsepower`
already tells us that the `mean_hp` and `sd_hp` are the mean and
standard deviation for horsepower - we can drop the `_hp` part.

basicTables can rename variables when exporting to .xlsx with
`new_name:old_name`. For example,
`cyl ~ (Horsepower = Mean:mean_hp + SD:sd_hp)` renames `mean_hp` to
`Mean` and `sd_hp` to `SD`:

    |    |    Horsepower  |
    |cyl | Mean   | SD    |
    |:---|---------------:|
    |4   | 91          NA |
    |4   | 81.8      21.9 |

The combination of row names, spanners, and renaming of variables allows
creating the full table as follows:

``` r
tbl <- bt(data = summarized_table,
          formula = Cylinder:cyl + Engine:vs ~
            N +
            (`Horse Power` = Mean:mean_hp + SD:sd_hp) +
            (`Weight` = Mean:mean_wt + SD:sd_wt),
          title = "Motor Trend Car Road Tests",
          subtitle = "A table created with basicTables",
          footnote = "Data from the infamous mtcars data set.")

wb <- write_bt(tbl = tbl)

# Save the excel table:
# openxlsx::saveWorkbook(wb,
#                        file = "cars.xlsx", overwrite = TRUE)
```

The result looks as follows:

![](man/figures/basicTables_example_cars.png)

## Additional Settings

### Tables without row names

Using `1` on the left hand side of the formula creates a table without
row names. For example, `1 ~ (Horsepower = Mean:mean_hp + SD:sd_hp)`
defines

    |    Horsepower  |
    | Mean   | SD    |
    |---------------:|
    | 91          NA |
    | 81.8      21.9 |

## References

- gt: Iannone R, Cheng J, Schloerke B, Hughes E, Lauer A, Seo J,
  Brevoort K, Roy O (2024). gt: Easily Create Presentation-Ready Display
  Tables. R package version 0.11.1.9000,
  <https://github.com/rstudio/gt>, <https://gt.rstudio.com>.
- expss: Gregory D et al. (2024). expss: Tables with Labels in R. R
  package version 0.9.31, <https://gdemin.github.io/expss/>.
- tables: Murdoch D (2024). tables: Formula-Driven Table Generation. R
  package version 0.9.31, <https://dmurdoch.github.io/tables/>.
