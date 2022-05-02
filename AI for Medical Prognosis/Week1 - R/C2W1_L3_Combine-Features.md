Week 1 Labs 3: Combine features - using R
================
Juan Li (based on python code on Github)
04/27/2022

In this exercise, you will practice how to combine features in a dataframe. This will help you in the graded assignment at the end of the week.

In addition, you will explore why it makes more sense to multiply two features rather than add them in order to create interaction terms.

First, you will generate some data to work with.

``` r
Xdata <- read.csv("X_data.csv", header = T)
ydata <- read.csv("y_data.csv", header = T)

# load package for manipulation 
library(dplyr)

# Similar to Labs 1, in R, it is more common to have features (X) and labels (y) in the same dataframe.
# Generate a dataframe of features (X) and labels (y) by binding columns.
data_raw <- bind_cols(Xdata, ydata)
```

``` r
# Random select 100 samples for later excercise.
data     <- sample_n(data_raw, 100)
```

``` r
head(data[-5])
#        Age Systolic_BP Diastolic_BP Cholesterol
# 1 73.77568   130.72125    101.20300   105.19896
# 2 47.64871   100.15534     88.07377   106.39597
# 3 60.49253   101.91407     75.59504    82.92737
# 4 61.06990    88.24461     84.27361    88.08008
# 5 52.42710   105.68607     92.16006    89.80134
# 6 60.57646   105.80934     95.54373   101.38954
```

``` r
feature_names <- names(data)[-5]
feature_names
# [1] "Age"          "Systolic_BP"  "Diastolic_BP" "Cholesterol"
```

## Combine strings

Even though you can visually see feature names and type the name of the combined feature, you can programmatically create interaction features so that you can apply this to any dataframe.

``` r
name1 <- names(data)[1] # Note indices start from 1 in R, while start from 0 in Python
name2 <- names(data)[2]

print(paste("name1:", name1))
# [1] "name1: Age"
print(paste("name2:", name2))
# [1] "name2: Systolic_BP"
```

``` r
# Combine the names of two features into a single string, separated by '_&_' for clarity
combined_names <- paste(name1, "_&_", name2, sep = "")
combined_names
# [1] "Age_&_Systolic_BP"
```

## Add two columns

-   Add the values from two columns and put them into a new column. Note I will use the dplyr pipes for this.
-   You'll do something similar in this week's assignment.

``` r
data <- data %>% 
  mutate(new_col = Age + Systolic_BP) 
names(data)[6] <- combined_names

# Note here the new column will be added after `y`
head(data, 2)
#        Age Systolic_BP Diastolic_BP Cholesterol y Age_&_Systolic_BP
# 1 73.77568    130.7213    101.20300     105.199 1          204.4969
# 2 47.64871    100.1553     88.07377     106.396 0          147.8041
```

``` r
# Not required, but you can rearrange the column orders
data <- data %>% 
  relocate(y, .after = last_col())
head(data, 2)
#        Age Systolic_BP Diastolic_BP Cholesterol Age_&_Systolic_BP y
# 1 73.77568    130.7213    101.20300     105.199          204.4969 1
# 2 47.64871    100.1553     88.07377     106.396          147.8041 0
```

## Why we multiply two features instead of adding

Why do you think it makes more sense to multiply two features together rather than adding them together?

Please take a look at two features, and compare what you get when you add them, versus when you multiply them together.

``` r
# Generate a small dataset with two features
df <- data.frame(v1 = c(1,1,1,2,2,2,3,3,3),
                 v2 = c(100,200,300,100,200,300,100,200,300))

# add the two features together
df <- df %>% mutate("v1 + v2" = v1 + v2)

# multiply the two features together
df <- df %>% mutate("v1 * v2" = v1 * v2)
df
#   v1  v2 v1 + v2 v1 * v2
# 1  1 100     101     100
# 2  1 200     201     200
# 3  1 300     301     300
# 4  2 100     102     200
# 5  2 200     202     400
# 6  2 300     302     600
# 7  3 100     103     300
# 8  3 200     203     600
# 9  3 300     303     900
```

It may not be immediately apparent how adding or multiplying makes a difference; either way you get unique values for each of these operations.

To view the data in a more helpful way, rearrange the data (pivot it) so that:

-   feature 1 is the row index
-   feature 2 is the column name.
-   Then set the sum of the two features as the value.

Display the resulting data in a heatmap.

``` r
# Pivot the data so that v1 + v2 is the value
df_add <- df %>% 
  select(-"v1 * v2") %>% 
  tidyr::pivot_wider(names_from = v2, values_from = "v1 + v2")
print("v1 + v2")
# [1] "v1 + v2"
df_add
# Warning: `...` is not empty.
# 
# We detected these problematic arguments:
# * `needs_dots`
# 
# These dots only exist to allow future extensions and should be empty.
# Did you misspecify an argument?
# # A tibble: 3 x 4
#      v1 `100` `200` `300`
#   <dbl> <dbl> <dbl> <dbl>
# 1     1   101   201   301
# 2     2   102   202   302
# 3     3   103   203   303

# Note using ggplot2::geom_tile you don't need to pivot data to have heatmap.
library(ggplot2)
ggplot(df, aes(v2, v1)) +              
  geom_tile(aes(fill = v1 + v2))
```

<img src="C2W1_L3_Combine-Features_files/figure-markdown_github/unnamed-chunk-11-1.png" width="//textwidth" />

Notice that it doesn't seem like you can easily distinguish clearly when you vary feature 1 (which ranges from 1 to 3), since feature 2 is so much larger in magnitude (100 to 300). This is because you added the two features together.

## View the 'multiply' interaction

Now pivot the data so that:

-   feature 1 is the row index
-   feature 2 is the column name.
-   The values are 'v1 x v2'

Use a heatmap to visualize the table.

``` r
df_mult <- df %>% 
  select(-"v1 + v2") %>% 
  tidyr::pivot_wider(names_from = v2, values_from = "v1 * v2")
print("v1 * v2")
# [1] "v1 * v2"
df_mult
# Warning: `...` is not empty.
# 
# We detected these problematic arguments:
# * `needs_dots`
# 
# These dots only exist to allow future extensions and should be empty.
# Did you misspecify an argument?
# # A tibble: 3 x 4
#      v1 `100` `200` `300`
#   <dbl> <dbl> <dbl> <dbl>
# 1     1   100   200   300
# 2     2   200   400   600
# 3     3   300   600   900

ggplot(df, aes(v2, v1)) +              
  geom_tile(aes(fill = v1 * v2))
```

<img src="C2W1_L3_Combine-Features_files/figure-markdown_github/unnamed-chunk-12-1.png" width="//textwidth" />

Notice how when you multiply the features, the heatmap looks more like a 'grid' shape instead of three vertical bars.

This means that you are more clearly able to make a distinction as feature 1 varies from 1 to 2 to 3.

## Discussion

When you find the interaction between two features, you ideally hope to see how varying one feature makes an impact on the interaction term. This is better achieved by multiplying the two features together rather than adding them together.

Another way to think of this is that you want to separate the feature space into a "grid", which you can do by multiplying the features together.

In this week's assignment, you will create interaction terms!

## This is the end of this practice section.

Please continue on with the lecture videos!