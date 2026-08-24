# CRISP-DM Phase 4: Visualizing Netflix Weekly Data. Part 2: Bar Charts and Histograms

In the previous tutorial, we used line charts to analyze audience changes over time. We will now use bar charts and histograms to compare titles and categories, examine audience distributions, and evaluate content longevity and runtime.

## Learning objectives

By the end of this tutorial, you will be able to:

* create and customize interactive bar charts and histograms with Plotly Express;
* group, summarize, sort, and filter data for visualization; and
* interpret comparisons and distributions to generate business insights.

## Business question

Our merchandising company needs to identify titles with large accumulated audiences, compare audience patterns among categories, and estimate how long popular content remains visible in the Top 10. This information can support decisions about which titles offer merchandising potential and how quickly related products should reach the market.

---

## 1. Load the libraries

We will use Pandas to handle the data and Plotly Express to create interactive visualizations.

```python
import pandas as pd
import plotly.express as px
```

---

## 2. Load the cleaned dataset

Upload `netflix_weekly_clean.csv` to Google Colab and load it into a pandas DataFrame.

```python
df = pd.read_csv("netflix_weekly_clean.csv")
df.info()
```

---

## 3. Bar chart: Accumulated audience by title

A bar chart compares numerical values across different categories.

We will group the records by `show_title` and calculate the accumulated `weekly_views` for each title.

* **X-axis:** `show_title`
* **Y-axis:** Total `weekly_views`
* **Business question:** Which Netflix titles accumulated the largest audiences?

```python
title_audience = df.groupby("show_title", as_index=False)["weekly_views"].sum()
print(title_audience)
```

Now, create a bar chart containing every title:

```python
fig_bar_1 = px.bar(
    title_audience,
    x="show_title",
    y="weekly_views",
    title="Accumulated Audience by Title",
    labels={
        "show_title": "Title",
        "weekly_views": "Total Views"
    },
)

fig_bar_1.update_layout(title_x=0.5)
fig_bar_1.show()
```

The chart contains thousands of bars, making the title labels and differences difficult to interpret. We can solve this problem by displaying only the titles with the largest accumulated audiences.

---

## 4. Bar chart: Ten most-viewed titles

First, sort the titles from the highest to the lowest accumulated audience and select the first ten records:

```python
top_10_titles = title_audience.sort_values("weekly_views", ascending=False).head(10)
print(top_10_titles)
```

The `ascending=False` argument sorts the values from highest to lowest.

The `head(10)` instruction keeps only the first ten records after sorting.

Now, create the bar chart:

```python
fig_bar_2 = px.bar(
    top_10_titles,
    x="show_title",
    y="weekly_views",
    title="Ten Most-Viewed Netflix Titles",
    labels={
        "show_title": "Title",
        "weekly_views": "Total Views"
    },
)
fig_bar_2.update_layout(title_x=0.5)
fig_bar_2.show()
```

### Tuning the graph

The `bar` function accepts the following optional arguments:


`color="show_title"` assigns a different color to each title.

`color_discrete_sequence=px.colors.qualitative.Pastel` changes the color palette.

`text="weekly_views"` displays the value on each bar.

`text_auto=".2s"` displays the numerical value on each bar using two significant digits and abbreviated units, such as `k` for thousands, `M` for millions, and `G` for billions.

`opacity=0.8` controls the transparency of the bars.

`width=900` and `height=600` control the dimensions of the chart.

`template="plotly_dark"` changes the visual theme.


Use the following instruction to change the order of the bars:

```python
fig_bar_2.update_xaxes(categoryorder="total ascending", tickangle=-35)
```

The `tickangle` argument changes the orientation of the text.

The `categoryorder` argument accepts the following options:

`categoryorder="category ascending"` sorts the titles alphabetically from A to Z.

`categoryorder="category descending"` sorts the titles alphabetically from Z to A.

`categoryorder="total ascending"` sorts the bars from the smallest to the largest value.

`categoryorder="total descending"` sorts the bars from the largest to the smallest value.


To change the orientation of the bar chart, add the following argument to the `px.bar()` function:

`orientation="h"` creates horizontal bars. When using this option, assign `weekly_views` to the X-axis and `show_title` to the Y-axis.

---

## 5. Grouped bar chart: Average weekly audience by rank and category

A grouped bar chart compares several groups for each value on the X-axis.

We will calculate the average `weekly_views` for every combination of `weekly_rank` and `category`.

* **X-axis:** `weekly_rank`
* **Y-axis:** Average `weekly_views`
* **Groups:** `category`
* **Business question:** How does the average audience decrease from rank 1 to rank 10, and how does this pattern differ among categories?

```python
rank_category_audience = df.groupby(["weekly_rank", "category"], as_index=False)["weekly_views"].mean()
print(rank_category_audience)
```

Replace the numerical category codes with descriptive names:

```python
category_names = {
    1: "Films (English)",
    2: "Films (Non-English)",
    3: "TV (English)",
    4: "TV (Non-English)"
}
rank_category_audience["category"] = rank_category_audience["category"].map(category_names)
```

Now, create the grouped bar chart:

```python
fig_bar_3 = px.bar(
    rank_category_audience,
    x="weekly_rank",
    y="weekly_views",
    title="Average Weekly Audience by Rank and Category",
    labels={
        "weekly_rank": "Weekly Rank",
        "weekly_views": "Average Weekly Views",
        "category": "Category"
    },
)

fig_bar_3.update_layout(title_x=0.5)
fig_bar_3.show()
```

### Tuning the graph

The `bar` function accepts the following optional arguments:

`color="category"` assigns a different color to each category.

`barmode="group"` places the category bars next to each other for every ranking position.

`color_discrete_sequence=px.colors.qualitative.Pastel` changes the color palette.

`opacity=0.8` controls the transparency of the bars.

`animation_frame="category"` animates the graph.

`range_y=[0,rank_category_audience["weekly_views"].max() * 1.1]` fixes the vertical scale for the animation.

`template="plotly_dark"` changes the visual theme.

Use the following command to display every rank from 1 to 10 on the X-axis.
```python
fig_bar_3.update_xaxes(dtick=1)
```

---

## 6. Histogram: Time spent in the Top 10

A histogram shows how frequently numerical values occur within a dataset.

We will calculate the number of weeks in which each `show_title` appears in the Netflix Top 10. This provides an estimate of how long a title remains visible to the audience and how much time the company may have to design, produce, and release related merchandise.

* **X-axis:** Number of weeks in the Top 10
* **Y-axis:** Number of titles
* **Business question:** How long do Netflix titles typically remain in the Top 10, and what merchandising opportunity window does this provide?

Group the records by `show_title` and `category`, and count the number of unique values in the `week` column:

```python
title_longevity = df.groupby(["show_title", "category"], as_index=False)["week"].nunique()
title_longevity = title_longevity.rename(columns={"week": "weeks_in_top_10"})
print(title_longevity)
```

For example, if the original DataFrame contains the following records:

```text
         week  category  show_title
0  2026-08-02         1     Movie A
1  2026-08-09         1     Movie A
2  2026-08-16         1     Movie A
3  2026-08-02         3      Show B
4  2026-08-02         3      Show B
5  2026-08-09         3      Show B
6  2026-08-09         4      Show C
```

After executing the code, we will obtain:

```text
  show_title  category  weeks_in_top_10
0     Movie A         1                3
1      Show B         3                2
2      Show C         4                1
```

The `nunique()` function counts each week only once. Therefore, if different seasons of the same series appear during the same week, that week is not counted multiple times.

The `rename()` function changes the resulting column name from `week` to `weeks_in_top_10`.

Now, create the histogram:

```python
fig_hist_1 = px.histogram(
    title_longevity,
    x="weeks_in_top_10",
    title="Time Spent in the Netflix Top 10",
    labels={
        "weeks_in_top_10": "Weeks in the Top 10"
    },
)

fig_hist_1.update_layout(title_x=0.5)
fig_hist_1.show()
```

### Tuning the graph

The `histogram` function accepts the following optional arguments:

`nbins=20` defines the approximate maximum number of intervals used to group the values.

`histnorm="percent"` displays percentages instead of the number of titles.

`cumulative=True` displays the accumulated frequency across the histogram.

`marginal="box"` adds a boxplot above the histogram. Other available options are `"rug"` and `"violin"`.

`text_auto=True` displays the frequency value on each bar.

`opacity=0.8` controls the transparency of the bars.

`width=900` and `height=600` control the dimensions of the chart.

`template="plotly_dark"` changes the visual theme.


Use the following instruction to create intervals with a width of one week:

```python
fig_hist_1.update_traces(xbins={"size": 1})
```

Use the following instruction to display one label every week on the X-axis:

```python
fig_hist_1.update_xaxes(dtick=1)
```

Use the following instruction to add space between the histogram bars:

```python
fig_hist_1.update_layout(bargap=0.1)
```
Use the following instruction to add a border around each histogram bar:

```python
fig_hist_1.update_traces(
    marker_line_color="white",
    marker_line_width=1,
    selector={"type": "histogram"}
)
```

The `selector={"type": "histogram"}` argument applies the border only to the histogram and prevents it from affecting the boxplot.



### Animated histogram by category

Create a descriptive category column using the `category_names` dictionary:

```python
category_names = {
    1: "Films (English)",
    2: "Films (Non-English)",
    3: "TV (English)",
    4: "TV (Non-English)"
}
title_longevity["category_name"] = title_longevity["category"].map(category_names)
```

Now, create an animated histogram that displays one category at a time:

```python
fig_hist_2 = px.histogram(
    title_longevity,
    x="weeks_in_top_10",
    title="Time Spent in the Netflix Top 10 by Category",
    labels={
        "weeks_in_top_10": "Weeks in the Top 10",
        "category_name": "Category"
    },
)
fig_hist_2.update_layout(title_x=0.5)
fig_hist_2.update_layout(bargap=0.1)
fig_hist_2.show()
```

### Tuning the graph

The grouped histogram accepts the same arguments used in the previous histogram.

`color="category_name"` assigns a different color to each category.

`color_discrete_sequence=px.colors.qualitative.Pastel` changes the color palette.

`barmode="stack"` places the category bars on top of each other.

`barmode="overlay"` places the category distributions over each other. When using this option, reduce the opacity to distinguish overlapping bars.

`barmode="group"` places the category bars next to each other within every interval.

`animation_frame="category_name"` creates one animation frame for each Netflix category.

`histnorm="percent"` displays the distribution of each category as percentages instead of counts.

`opacity=0.7` makes overlapping bars partially transparent.

`template="plotly_dark"` changes the visual theme.

---

## Checkpoint: Create your own bar charts and histograms

Create a visualization for each question and use the graph to write a short business insight.


### 1. Which titles accumulated the most viewing hours?

**Business question:** Which Netflix titles generated the greatest total audience engagement and may offer strong merchandising potential?

**Hint:** Group the records by `show_title`, calculate the sum of `weekly_hours_viewed`, sort the results from highest to lowest, and select the first ten titles. Create a bar chart using `show_title` and `weekly_hours_viewed`.


### 2. Which category accumulated the largest audience?

**Business question:** Which Netflix content category represents the largest potential market for merchandise?

**Hint:** Group the records by `category` and calculate the sum of `weekly_views`. Replace the category codes with their descriptive names and create a bar chart.


### 3. How does the distribution of viewing hours differ among categories?

**Business question:** Do the four Netflix categories show similar audience engagement, or do some categories usually accumulate more viewing hours?

**Hint:** Create a histogram using `weekly_hours_viewed` on the X-axis. Use `color="category"` to compare the categories and test `barmode="group"` or `barmode="overlay"`.


### 4. What is the typical runtime of Netflix Top 10 titles?

**Business question:** What is the typical duration of successful Netflix content, and are there unusually short or long titles?

**Hint:** Remove records containing missing, zero, or negative `runtime` values:

```python
runtime_data = df.dropna(subset=["runtime"])
runtime_data = runtime_data[runtime_data["runtime"] > 0]
```

Create a histogram using `runtime` on the X-axis and add `marginal="box"`. The boxplot shows the median, the dispersion of the central 50% of the data (values between Q1 and Q3), and possible outliers (samples outside the lower and upper fences).

---

## Expected result

At the end of this activity, you should be able to:

- create interactive bar charts and histograms using Plotly Express;
- group, sort, and filter data;
- examine the distribution and frequency of Netflix content;
- generate business insights from the visualizations.

---

## Sources

- Netflix, *Top 10*:  
  <https://www.netflix.com/tudum/top10>
- Plotly, *Plotly Express Documentation*:  
  <https://plotly.com/python/plotly-express/>
- CRISP-DM course material, Phase 4: Modeling.
