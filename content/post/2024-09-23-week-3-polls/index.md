---
title: 'Week 3: Polls'
author: Jay Chooi
date: '2024-09-23'
slug: week-3-polls
categories: []
tags: []
---


``` r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

``` r
library(ggplot2)

swing_states <- c("Wisconsin", "Michigan", "Arizona", "Nevada", "North Carolina", "Georgia")

pp16 <- read.csv("president_polls_2016.csv")
pp20 <- read.csv("president_polls_2020.csv")  |> mutate(state = ifelse(state=='', "U.S.", state))
pp24 <- read.csv("president_polls_2024.csv") |> 
  mutate(state = ifelse(state=='', "U.S.", state))
```


``` r
# Combine the dataframes into a list
pp16$date <- as.character(pp16$enddate)
pp20$date <- as.character(pp20$end_date)
pp24$date <- as.character(pp24$end_date)

pp20$grade <- pp20$fte_grade
pp24$grade <- pp24$numeric_grade

pp16 <- pp16 |> filter(substr(date,1,2) == "8/")
pp20 <- pp20 |> filter(substr(date,1,2) == "8/")
pp24 <- pp24 |> filter(substr(date,1,2) == "8/")


polls <- list(pp16, pp20, pp24)

# Apply lapply to select only the 'cycle' and 'state' columns from each dataframe
polls <- lapply(polls, function(x) {
  x |> select(cycle, state, date)
})

# Use do.call with rbind to combine the list of dataframes into one dataframe
cnt_polls <- do.call(rbind, polls)

cnt_polls <- cnt_polls |>
  group_by(cycle,state) |>
  summarize(count=n())
```

```
## `summarise()` has grouped output by 'cycle'. You can override using the
## `.groups` argument.
```

``` r
# View the combined dataframe
cnt_polls
```

```
## # A tibble: 144 × 3
## # Groups:   cycle [3]
##    cycle state                count
##    <int> <chr>                <int>
##  1  2016 Alabama                 15
##  2  2016 Alaska                  15
##  3  2016 Arizona                 36
##  4  2016 Arkansas                15
##  5  2016 California              21
##  6  2016 Colorado                24
##  7  2016 Connecticut             15
##  8  2016 Delaware                12
##  9  2016 District of Columbia     9
## 10  2016 Florida                 51
## # ℹ 134 more rows
```


``` r
selected_cnt_polls <- cnt_polls |>
  filter(state == "U.S." | state %in% swing_states)

ggplot(data = selected_cnt_polls, aes(x = cycle, y = count, fill = state)) +
  geom_bar(stat = "identity", position = "dodge") +  # Use 'position = dodge' for side-by-side bars
  scale_x_continuous(breaks = c(2016, 2020, 2024)) +  # Set the breaks for the x-axis
  theme_minimal() + 
  labs(title = "Number of polls ending in October in the U.S. and swing states", x = "Cycle", y = "Count", fill = "State")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

``` r
# install.packages("patchwork")
library(patchwork)

# Define the order of grades from worst to best for both datasets
grade_levels <- c("F", "D", "C-", "C/D", "C", "C+", "B/C", "B-", "B", "B+", "A/B", "A-", "A", "A+")

# Ensure 'pp16', 'pp20', and 'pp24' have grades ordered from worst to best
pp16$grade <- factor(pp16$grade, levels = grade_levels)
pp20$grade <- factor(pp20$grade, levels = grade_levels)

# Plot for pp16
plot_pp16 <- ggplot(pp16 |> filter(!is.na(grade)), aes(x = grade)) +
  geom_bar(fill = "skyblue") +
  labs(title = "Grade Distribution for 2016 Polls", x = "Grade", y = "Count") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))  # Rotate x-axis labels for readability

# Plot for pp20
plot_pp20 <- ggplot(pp20 |> filter(!is.na(grade)), aes(x = grade)) +
  geom_bar(fill = "lightgreen") +
  labs(title = "Grade Distribution for 2020 Polls", x = "Grade", y = "Count") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))  # Rotate x-axis labels for readability

# Plot for pp24
plot_pp24 <- ggplot(pp24 |> filter(!is.na(grade)), aes(x = grade)) +
  geom_bar(fill = "lightcoral") +
  labs(title = "Grade Distribution for 2024 Polls", x = "Grade", y = "Count") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))  # Rotate x-axis labels for readability

# Combine the plots using patchwork (you can adjust layout)
combined_plot <- (plot_pp16 | plot_pp20) / plot_pp24  # Side-by-side for pp16 and pp20, pp24 below

# Display the combined plot
combined_plot
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />



``` r
# Function to get the top 20% of rows by grade
get_top_20_percent <- function(df) {
  df |> filter(state=="U.S.") %>%
    arrange(desc(grade)) %>%  # Arrange by grade in descending order
    top_n(n = ceiling(0.2 * nrow(df)), wt = grade)  # Select top 20% rows
}

# Function to get the worst 20% of rows by grade
get_bottom_20_percent <- function(df) {
  df  |> filter(state=="U.S.") %>%
    arrange(grade) %>%  # Arrange by grade in descending order
    top_n(n = ceiling(0.2 * nrow(df)), wt = grade)  # Select top 20% rows
}


# Apply the function to each dataset
pp16_top20 <- get_top_20_percent(pp16)
pp20_top20 <- get_top_20_percent(pp20)
pp24_top20 <- get_top_20_percent(pp24)

# Apply the function to each dataset for bottom 20%
pp16_bottom20 <- get_bottom_20_percent(pp16)
pp20_bottom20 <- get_bottom_20_percent(pp20)
pp24_bottom20 <- get_bottom_20_percent(pp24)
```


``` r
pp16_top20_R_vote <- mean(pp16_top20$adjpoll_trump)
pp20_top20_R_vote <- mean((pp20_top20|>filter(candidate_name=="Donald Trump"))$pct, na.rm = TRUE)
pp24_top20_R_vote <- mean((pp24_top20|>filter(candidate_name=="Donald Trump"))$pct, na.rm = TRUE)

pp16_bottom20_R_vote <- mean(pp16_bottom20$adjpoll_trump, na.rm = TRUE)
pp20_bottom20_R_vote <- mean((pp20_bottom20 |> filter(candidate_name == "Donald Trump"))$pct, na.rm = TRUE)
pp24_bottom20_R_vote <- mean((pp24_bottom20 |> filter(candidate_name == "Donald Trump"))$pct, na.rm = TRUE)
```


``` r
# Create a dataframe with the top and bottom 20% Trump vote for each year
trump_vote_df <- data.frame(
  year = c(2016, 2020, 2024, 2016, 2020, 2024),
  group = c("Top 20%", "Top 20%", "Top 20%", "Bottom 20%", "Bottom 20%", "Bottom 20%"),
  vote = c(
    pp16_top20_R_vote, pp20_top20_R_vote, pp24_top20_R_vote,  # Top 20% votes
    pp16_bottom20_R_vote, pp20_bottom20_R_vote, pp24_bottom20_R_vote  # Bottom 20% votes
  )
)

# Plot the bar chart
ggplot(trump_vote_df, aes(x = factor(year), y = vote, fill = group)) +
  geom_bar(stat = "identity", position = "dodge") +  # Dodge to place bars side by side
  labs(title = "Trump Vote Percentage for Top 20% Polls vs Bottom 20% Polls", 
       x = "Year", 
       y = "Trump Vote Percentage") +
  scale_fill_manual(values = c("Top 20%" = "skyblue", "Bottom 20%" = "lightcoral")) +  # Custom colors for top and bottom
  theme_minimal() +
  labs(fill="Poll Grade")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />




``` r
sp <- read.csv("state_polls_1968-2024.csv")
```



``` r
# Load popular vote data. 
d_popvote <- read.csv("popvote_1948-2020.csv")
d_state <- read.csv("clean_wide_state_2pv_1948_2020.csv")
d_state <- d_state |>
  right_join(d_popvote, by=c("year")) |>
  left_join(sp, by=c("year", "state")) |>
  filter(party.y=="REP") |>
  filter(party.x=="republican")
```

```
## Warning in right_join(d_state, d_popvote, by = c("year")): Detected an unexpected many-to-many relationship between `x` and `y`.
## ℹ Row 1 of `x` matches multiple rows in `y`.
## ℹ Row 1 of `y` matches multiple rows in `x`.
## ℹ If a many-to-many relationship is expected, set `relationship =
##   "many-to-many"` to silence this warning.
```

```
## Warning in left_join(right_join(d_state, d_popvote, by = c("year")), sp, : Detected an unexpected many-to-many relationship between `x` and `y`.
## ℹ Row 601 of `x` matches multiple rows in `y`.
## ℹ Row 177953 of `y` matches multiple rows in `x`.
## ℹ If a many-to-many relationship is expected, set `relationship =
##   "many-to-many"` to silence this warning.
```

``` r
d_state2 <- d_state |> 
  select(c(year,state,R_pv2p,weeks_left,poll_support))
```

