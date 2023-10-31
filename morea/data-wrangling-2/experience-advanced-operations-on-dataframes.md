---
title: "6. Advanced Operations on Series and DataFrames"
published: true
morea_id: experience-advanced-operations-on-dataframes
morea_type: experience
morea_summary: "Understanding advanced functions that can be applied on Series and DataFrames like groupby"
morea_sort_order: 3
morea_labels:
  - 2:10pm
morea_enable_toc: true
---

# 6. Advanced Operations on Series and DataFrames

<a target="_blank" href="https://colab.research.google.com/github/mahdi-b/change-hi.github.io/blob/main/morea/data-wrangling-2/Notebooks/06-advanced-operations-on-series-and-dataframes.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>

<div class="alert alert-success mt-3" role="alert" markdown="1">
<i class="fa-solid fa-globe fa-xl"></i> **Overview**
<hr/>

**Questions**
* How do you create data chunks using values on one or more columns?
* How do you apply some operation on created data chunks?
* How do you combined the results (processed chunks) into a new `DataFrame` ?

**Objectives**
* Learn how to group data based on a column/s using `groupby`.
* Learn how to process created data groups.
* Learn how to combine the results together.

</div>

## Global Vs. Group-Specific Processing

Function application falls into one of two categories:

1. **Global Processing**
2. **Group Specific Processing**

Global processing is applying the same function to every entry (referring to a singular data point or an entire row or column) in a `Series` or `DataFrame`. Group specific processing, on the other hand, is applying functions to entries that belong to a certain group based on some defining characteristic.

We will begin by covering global processing in the following sections.

### 1. Global Processing

In one clever way or another, every global processing problem you will ever run into when working with `DataFrames` will fit into one of two levels of granularity. Corresponding to these two levels are two `DataFrame` methods, `apply()` and `applymap()`.

To apply a function to every row or column of a `DataFrame` we use the `apply()` DataFrame method. The `apply()` method takes a function that will be applied to the specified axis (columns or rows), the axis, and other keyword arguments that are defined by default. Depending on the function passed to the `apply()` can behave in the same way as the `applymap()` function.

#### 1.1) Global Processing-apply()
Let us look at two examples use cases, using a reducing function, and a universal function. The function passed to the `apply()` method will process a `Series`, either a `DataFrame` row or column depending on the axis parameter, and return a result. 

Most of the time when you call the `apply()` method you should be using a reducing function. A reducing function is one which takes a `Series` object and reduces the `Series` to a either a new `Series` or a single entry using a process that relies on data in the `Series`. You are already familiar with some reducing functions such as the Series `sum()` method, which returns the sum of all the entries in the calling `Series`. Consider the following example of calling `apply()` with a reducing function.

<div class="alert alert-secondary" role="alert" markdown="1">
Code:
```python
df = pd.DataFrame([[ 0,  3,  6], [ 9, 12, 15], [18, 21, 24]], columns=['a', 'b', 'c'])
def square_sum(x_series):
  return x_series.sum() ** 2
df.apply(square_sum, axis=1) # apply to each row
```

Output:
```python
0      81
1    1296
2    3969
dtype: int64
```
</div>

The reducing function, `square_sum()` in the example above sums all the entries in the `Series` and the squares the result. You can define custom reducing functions just like we showed above to to achieve your desired analysis.

A universal function will return a new `Series` that was created by universally applying the same procedure to each `Series` entry. A universal function can be defined using the `Series` `map()` method. The `map()` method will take a function as an argument which will process each individual `Series` entry according to the function definition. For instance refer to the following code example.

<div class="alert alert-secondary" role="alert" markdown="1">
Code:
```python
df = pd.DataFrame([[ 0,  3,  6], [ 9, 12, 15], [18, 21, 24]], columns=['a', 'b', 'c'])
def divide_by_three(x_series):
   return x_series.map(lambda x: x / 3)
df.apply(divide_by_three, axis=0) # apply to each column
```

Output:
```python
     a    b    c
0  0.0  1.0  2.0
1  3.0  4.0  5.0
2  6.0  7.0  8.0
```
</div>

The example above shows how the `apply()` method behaves when a universal function is passed as the argument. The resulting `DataFrame` is constructed from original `DataFrame` except each individual entry is divided by three.

#### 1.2) Global Processing-applymap()
There is a shorthand way achieve the same exact behavior shown in the example of applying a universal function in the **Global Processing-apply()** cell above, and the method is appropriately named `applymap()`, as first we call the `apply()` DataFrame method and then we call the `map()` method. 

To apply a function to every individual element in a `DataFrame` we can use the `applymap()` DataFrame method.  The `applymap()` method is a function which takes one positional argument as input and that is a callable function which takes a single value and returns a single value. The `applymap()` method will apply the function passed to every single entry in the calling `DataFrame` and return a new `DataFrame` with the processed entries.

Let us see a simple example. We will construct a DataFrame `df` that is 3x3, i.e. there are three rows and three columns. The entries will be consecutive multiples of 3. To each entry we will apply the anonymous function: `lambda x: x / 3` which will divide a given input by 3. The result will be a new 3x3 `DataFrame` with the same index and columns as the caller with entries that are the results of the passed function.

<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
df = pd.DataFrame([[ 0,  3,  6], [ 9, 12, 15], [18, 21, 24]], columns=['a', 'b', 'c'])
df.applymap(lambda x: x / 3)
```

Output:
```python
     a    b    c
0  0.0  1.0  2.0
1  3.0  4.0  5.0
2  6.0  7.0  8.0
```
</div>

This type of processing is relatively rare since it should be the case that it makes sense to apply the same function to every entry regardless of its location. But when you need this functionality, the `applymap()` function is quite useful. 

### 2. Group Specific Processing

A common scenario is applying a function to a specific group of data. By group of data I mean a subset of the data that is the same based on a criterion. 

The `groupby()` DataFrame method is used to group rows of data by one or more of the column entries . The `groupby()` method accepts the parameter `by` which specifies how you want to group the rows of the calling `DataFrame`. The creation of groups `by` can be a single column label, a list of column lables, or a callable function. The method will return a `pandas` `GroupBy` object, an object we have not seen before. This object has certain attributes and methods that will be useful to us. In this chapter we will only cover the case of setting the `by` parameter of the `groupby()` method to a single column entry, if you are interested you can read more about the method [here](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.groupby.html). 

If `by` is a single label then the calling `DataFrame` will be grouped by the values in the column with the passed label, i.e. every entry with the same value in the specified column will be in the same group. 

For example, consider the [`20_sales_records.xlsx`](https://github.com/mahdi-b/change-hi.github.io/raw/main/morea/data-wrangling-1/Notebook/data/20_sales_records.xlsx) DataFrame from the previous week. To simply things, we will select only `Region`, `Order Priority`, `Sales Channel`, `Total Revenue` and `Total Profit` columns and call it `df`.
<div class="alert alert-secondary" role="alert" markdown="1">
Code:
```python
import pandas as pd
URL = "https://github.com/mahdi-b/change-hi.github.io/raw/main/morea/data-wrangling-1/Notebook/data/20_sales_records.xlsx"
df = pd.read_excel(URL)

df = df.loc[:, ["Region", "Order Priority", "Sales Channel", "Total Revenue", "Total Profit"]]
df
```

Output:

|  | Region                          | Order Priority | Sales Channel | Total Revenue | Total Profit |
|-------|---------------------------------|----------------|---------------|---------------|--------------|
| 0     | Australia and Oceania           | H              | Offline       | 2533654.00    | 951410.50    |
| 1     | Central America and the Caribbean| C              | none          | 576782.80     | 248406.36    |
| 2     | Europe                          | none           | Offline       | 1158502.59    | 224598.75    |
| 3     | Sub-Saharan Africa              | L              | Online        | 75591.66      | 19525.82     |
| 4     | Sub-Saharan Africa              | L              | Offline       | 3296425.02    | 639077.50    |
| 5     | Australia and Oceania           | C              | Online        | 759202.72     | 285087.64    |
| 6     | Sub-Saharan Africa              | M              | none          | 2798046.49    | 693911.51    |
| 7     | Sub-Saharan Africa              | H              | Online        | 1245112.92    | 510216.66    |
| 8     | Sub-Saharan Africa              | M              | Offline       | 496101.10     | 152114.20    |
| 9     | Sub-Saharan Africa              | H              | Online        | 1356180.10    | 584073.87    |
| 10    | Asia                            | H              | Online        | 19103.44      | 7828.12      |
| 11    | Sub-Saharan Africa              | H              | Offline       | 455479.04     | 306097.92    |
| 12    | Asia                            | L              | Online        | 902980.64     | 606834.72    |
| 13    | Central America and the Caribbean| H              | Offline       | 5997054.98    | 1487261.02   |
| 14    | Asia                            | C              | Offline       | 400558.73     | 122819.06    |
| 15    | Europe                          | M              | Online        | 182825.44     | 122865.12    |
| 16    | Asia                            | M              | Offline       | 3039414.40    | 1208744.24   |
| 17    | Sub-Saharan Africa              | C              | Offline       | 257653.50     | 85033.80     |
| 18    | Asia                            | L              | Offline       | 2559474.10    | 634745.90    |
{: .table}
</div>


For example, let us group the above `df` DataFrame by the values in the `Region` column and save the returned GroupBy object to the variable we will call `grouped_by_region`. To do this we use the following code.

For example, consider the following `DataFrame`:
<div class="alert alert-secondary" role="alert" markdown="1">
Code:
```python
grouped_by_region = df.groupby('Region')
```
</div>

`GroupBy` objects have a handy method called `get_group()`, which returns all the entries of a specified group as a `DataFrame`. The `get_group()` method will take a positional argument that is the name of the group to access. Then the method returns a `DataFrame`, which is a subset of the initial `DataFrame` used to instantiate the `GroupBy` object. The entries of the returned `DataFrame` are all those entries in the column specified by the `by` parameter in the original `groupby()` call that match the name used in the `get_group()` call.

Continuing with the example of the `grouped_by_region` object, let us see how we would retrieve the group of rows from the `df` whos entries in the `df` column were all the same value of 'Asia'. This group will conviently have the name 'Asia', thus when we use the `get_group` method we will simply pass the value 'Asia'.

<div class="alert alert-secondary" role="alert" markdown="1">
Code:
```python
grouped_by_region.get_group('Asia')
```

Output:

|   | Region | Order Priority | Sales Channel | Total Revenue | Total Profit |
|-------|--------|----------------|---------------|---------------|--------------|
| 10    | Asia   | H              | Online        | 19103.44      | 7828.12      |
| 12    | Asia   | L              | Online        | 902980.64     | 606834.72    |
| 14    | Asia   | C              | Offline       | 400558.73     | 122819.06    |
| 16    | Asia   | M              | Offline       | 3039414.40    | 1208744.24   |
| 18    | Asia   | L              | Offline       | 2559474.10    | 634745.90    |
{: .table}
</div>

#### 2.1) Split-Apply-Combine
Getting groups can be easily implemented using subsetting. For instance, we could have obtained the "A" group of the `df` DataFrame by subsetting `df` with the boolean `Series` returned from the following operation.

<div class="alert alert-secondary" role="alert" markdown="1">
Code:
```python
df[df.loc[:,"Region"]=="Asia"]
```

Output:

|   | Region | Order Priority | Sales Channel | Total Revenue | Total Profit |
|-------|--------|----------------|---------------|---------------|--------------|
| 10    | Asia   | H              | Online        | 19103.44      | 7828.12      |
| 12    | Asia   | L              | Online        | 902980.64     | 606834.72    |
| 14    | Asia   | C              | Offline       | 400558.73     | 122819.06    |
| 16    | Asia   | M              | Offline       | 3039414.40    | 1208744.24   |
| 18    | Asia   | L              | Offline       | 2559474.10    | 634745.90    |
{: .table}
</div>

We see in the above example that the returned `DataFrame` is exactly the same as the result we saw in the previous cell introducing `groupby()` and `get_group()`. So why use `GroupBy` objects anyway?

An ideal usage of `groupby()`, and the resulting `GroupBy object`, will apply operations to **each** group independently. Furthermore, `GroupBy` objects are intended to be applied in the context of the data processing paradigm called 
"split-apply-combine"

* **Split** the data into chunks defined using one or more columns.
* **Apply** some operation on the chunks generated. 
* **Combine** the results of the applied operation into a new `DataFrame`.

For instance, suppose we wanted to compute the the sum by `Region` and save the result to a news `DataFrame`, the steps we would need to take are:

   1. Split the data by `X`, i.e. `groupby('Region')`
   2. Apply the `sum()` method to the `Y` column for each group
   3. Combine the results from each group into a new `DataFrame`

![](fig/E6_split_apply_combine_example.png)

So rather than manually subsetting each group and then applying the desired operation we could automate this workflow using the helpful `GroupBy` methods implemented by `pandas` to save ourselves some time and effort.

#### 2.2) The 3 Classes of Opearations on Groups

There are 3 classes of split-apply-combine operations that can be applied to group data.

a. __Aggregations__ generate a single value for each group
  
b.  __Transformations__ convert the data and generate a group of the same size as the original group.

c.  __Filters__ retain or discard a group based on group-specific boolean computations.

<img src="fig/E6_aggregate.png" width="80%">
<img src="fig/E6_transform.png" width="80%">
<img src="fig/E6_filter.png" width="80%">

##### 2.2.a) Aggregate

__Aggregations__ aggregate the data in each group, i.e., they reduce the data to a single value. This includes, for instance, computing group sums, means, maximums, minimums, _etc_. Some of the interesting/important summary aggregation methods of `GroupBy`  objects are:

|Methods| Decription|
|:----------|:----------------|
| `mean`, `median` | Computes the mean and the median in each group| 
| `min` , `max` | computes the min and max in each group| 
| `size` | computes the number of values in each group|
{: .table}

When one of these methods are called by the `GroupBy` object, they are applied to each group individually and then the group is combined into a new `DataFrame`.

For example suppose we wanted to group `df` by `Region`, apply the `sum()` method to calculate the aggregate `Total Revenue` by `Region` and aggregate `Total Profit` by `Region`. Then we can combine the results into a new `DataFrame` which holds the aggregate `Total Revenue` by `Region` and the aggregate `Total Profit` by `Region`. We could achieve this by first splitting the data using the `groupby()` `DataFrame` method to obtain a new `GroupBy` object, we will call it `grouped_by_region`. Then we could apply and combine using the `GroupBy` object's `sum()` method.

<div class="alert alert-secondary" role="alert" markdown="1">
Code:
```python
grouped_by_region = df.loc[:, ['Region', 'Total Revenue', 'Total Profit']].groupby("Region")
grouped_by_region.sum()
```

Output:

|        | Total Revenue | Total Profit |
|--------|---------------|--------------|
| Region |               |              |
| Asia   | 6921531.31    | 2580972.04   |
| Australia and Oceania | 3292856.72 | 1236498.14 |
| Central America and the Caribbean | 6573837.78 | 1735667.38 |
| Europe | 1341328.03 | 347463.87 |
| Sub-Saharan Africa | 9980589.83 | 2990051.28 |
{: .table}
</div>

We see from the above example that the `GroupBy` `sum()` method returns a `DataFrame` with an index labeling the group that the row entry corresponds to and entries telling us the aggregate `Total Revenue` and aggregate `Total Profit` by `Region`.

##### Aggregate Continued

As discussed in the previous cell, `pandas` has implemented for us the most common aggregate methods for us, like `sum()` and `mean()`, but sometimes our data requires unique processing. The `GroupBy` method `agg()` can be used where complex or custom aggregation logic is required. The method `agg()` will take a function and use it to aggregate the group in the same way that we saw `sum()` do in the previous cell. The function passed must take a `DataFrame` as an argument, and that passed `DataFrame` will be each group of the calling `GroupBy` object.

For example, suppose we wanted to find the aggregate `Total Revenue` by Region in Canadian dollars. We can define a function called `sum_total_revenue_CAD()` to return the sum of the `Total Revenue` of a group in Canadian Dollars. Then we can create a new `GroupBy` object, call it `grouped_by_region`, using a subset of the `df` DataFrame only containing the `Region` and `Total Revenue` columns. Lastly, we can can call `agg()` with the `grouped_by_region` GroupBy object and pass it the `sum_total_revenue_CAD` function.

<div class="alert alert-secondary" role="alert" markdown="1">
Code:
```python
def sum_total_revenue_CAD(x):
    return x.sum() * 1.33

grouped_by_region = df.loc[:, ["Region", "Total Revenue"]].groupby("Region")
grouped_by_region.agg(sum_total_revenue_CAD)
```

Output:

|        | Total Revenue |
|--------|---------------|
| **Region** |               |
| Asia                           | 9.205637e+06  |
| Australia and Oceania          | 4.379499e+06  |
| Central America and the Caribbean | 8.743204e+06 |
| Europe                         | 1.783966e+06  |
| Sub-Saharan Africa             | 1.327418e+07  |
{: .table}

</div>

We see in the above example that the result is a new `DataFrame` with the unique `Region` values as the index and values corresponding the sum of the `Total Revenue` by `Region` in Candian dollars.

To customize group specific processing even further `agg()` can also take a dictionary of functions to aggregate on. The dictionary should be the name of a column of the group and the value a callable function that will take a `Series`. 

For example, suppose we wanted to create a new `DataFrame` that tells us the sum of `Total Revenue` and the max `Total Profit` by Region from `df`. To do this we would first `groupby()` `Region` and then call `agg()` with the new `GroupBy` object, passing it the dictionary: `{'Total Revenue' :sum,'Total Profit' : max}`, which specifies that we want to sum the `Total Revenue` column and find the max of the `Total Profit` column.

<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
grouped_by_region = df.groupby("Region")
grouped_by_region.agg({'Total Revenue' :sum,'Total Profit' : max})
```

Output:

|        | Total Revenue | Total Profit |
|--------|---------------|--------------|
| **Region** |               |              |
| Asia                           | 6921531.31    | 1208744.24   |
| Australia and Oceania          | 3292856.72    | 951410.50    |
| Central America and the Caribbean | 6573837.78 | 1487261.02   |
| Europe                         | 1341328.03    | 224598.75    |
| Sub-Saharan Africa             | 9980589.83    | 693911.51    |
{: .table}
</div>

##### 2.2.b) Transform

 __Transformations__ change the data in a way that is group-specific. As opposed to aggregations, which reduce the data into a single value, transformations modify the data but don't change the shape of the groups.

Applying a transformation is done using the `transform()` `GroupBy` method. The `transform()` method takes as input a function name, which it calls on each group of the `GroupBy` object. The function passed to `transform()` must take a `DataFrame`, which will be a group of the calling `GroupBy` object. 

For example, suppose we wanted to transform the `Total Revenue` column of the `df` DataFrame to hold the percentage of the Total Revenue by Region that rows makes up.  First, we would define a function which will take a `DataFrame` and calculate the the percentage of the total each entry takes up. Then we will create a new `GroupBy` object groupded by `specialty` from a subset of `df` that only has the columns `Total Revenue` and `Region`. Then we will call transform passing it the name of our defined function. 

<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
def my_function(x):
    return (x   / x.sum() ) * 100
grouped_by_region = df.loc[:, ["Region", "Total Revenue"]].groupby("Region")
grouped_by_region.transform(my_function)
```

Output:

|    | Total Revenue |
|----|---------------|
| 0  | 76.943949     |
| 1  | 8.773913      |
| 2  | 86.369819     |
| 3  | 0.757387      |
| 4  | 33.028359     |
| 5  | 23.056051     |
| 6  | 28.034881     |
| 7  | 12.475344     |
| 8  | 4.970659      |
| 9  | 13.588176     |
| 10 | 0.276000      |
| 11 | 4.563649      |
| 12 | 13.045966     |
| 13 | 91.226087     |
| 14 | 5.787140      |
| 15 | 13.630181     |
| 16 | 43.912456     |
| 17 | 2.581546      |
| 18 | 36.978437     |
{: .table}
</div>

We see that the result is a new `DataFrame` with an index matching that of the original `DataFrame` used to initialize the `GroupBy` object. This is different than the aggregation example because aggregation reduces the group to a single value, while transformation maintains the shape of the calling `DataFrame`.

Let us save these results into a new column in `df` called `Total_Revenue_Percentage`.

<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
df["Total_Revenue_Percentage"] = grouped_by_region.transform(my_function)
```
</div>

Suppose we wanted to see the percent Total Revenue by `Order Priority` and `Region`. One solution to achieve this would be to group on both the `Region` and the `Order Priority` columns and then sum the `Total_Revenue_Percentage` that was computed previously.

<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
order_priority_pct = df.loc[:, ["Region", "Order Priority", "Total_Revenue_Percentage"]].groupby(["Region", "Order Priority"]).sum()
order_priority_pct.head(n=4)
```

Output:

|        |                | Total_Revenue_Percentage |
| Region | Order Priority |                          |
|--------|----------------|--------------------------|
| Asia   | C              | 5.787140                 |
|        | H              | 0.276000                 |
|        | L              | 50.024403                |
|        | M              | 43.912456                |
{: .table}
</div>

Notice that since we are grouping on two columns, the resulting index of `order_priority_pct` also contains two columns. Now we can sort first on the values in the `Region` column, so that all the entries with a common `Region` are clustered together, and then on the values in the `Total_Revenue_Percentage` columns.

<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
order_priority_pct = df.loc[:, ["Region", "Order Priority", "Total_Revenue_Percentage"]].groupby(["Region", "Order Priority"]).sum()
order_priority_pct.sort_values(["Region", "Total_Revenue_Percentage"], ascending=[True, False]).head(n=6)
```

Output:

|                       |                | Total_Revenue_Percentage |
| Region                | Order Priority |                          |
|---------------------- |----------------|--------------------------|
| Asia                  | L              | 50.024403                |
|                       | M              | 43.912456                |
|                       | C              | 5.787140                 |
|                       | H              | 0.276000                 |
| Australia and Oceania | H              | 76.943949                |
|                       | C              | 23.056051                |
{: .table}
</div>

##### 2.2.c) Filter

 __Filtering__  a group consists of dropping or retaining groups in a way that depends on a group-specific computation that returns `True` or `False`. Groups that are retained will be left unmodified. For instance, we can filter specialties from `spending_df` that don't have enough entries or for which the mean `spending` is below a certain threshold.

Filtering a group is done using the `GroupBy` method `filter()`. The method `filter()` takes as input a function name, which it calls on each group of the `GroupBy` object. The function must return either `True` or `False` and groups for which the function returns `False` are dropped. The resulting `DataFrame` will have entries in the same order as the original `DataFrame`.


Suppose we want to filter out the **Regions** that are having low Total Revenue, i.e. we want to filter out the Regions for which the aggregate `Total Revenue` is less than some defined threshold, let say $4,000,000. To do this we can define a function named `filter_on_total_revenue()`. The defined function will take a `DataFrame`, determine whether the sum of the `Total Revenue` column in that `DataFrame` is less than 5000000, and then return `True` if it is or `False` if not. 

Then, to apply the filter on `df`, we first subset the `DataFrame` so that only the columns `Region` and `Total Revenue` are remaining and then group by `Region`. Then the `GroupBy filter()` method can be called with the `filter_on_total_revenue()` function passed as an argument. We can save the results into a new `DataFrame` named `low_revenue_df`. Then to see which Regions are less than the $4,000,000 Total Revenue threshold we can print the unique values in the `Region` column of `low_revenue_df`.


<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
def filter_on_total_revenue(x):
    return x['Total Revenue'].sum() < 4000000
low_revenue_df = df[["Region", "Total Revenue"]].groupby("Region").filter(filter_on_total_revenue)
low_revenue_df["Region"].unique()
```

Output:
```python
array(['Australia and Oceania', 'Europe'], dtype=object)
```
</div>

We see that only two Regions are under the threshold of Total Revenue.

###### Thinning Data and The Flexible `apply()`  GroupBy Method

`pandas` provides a few built-in `GroupBy` methods for thinning the data including `nlargest()`, `nsmallest()`, and more. An example usage of `nlargest()`, a thinning method, would be grouping a subset of `df` which contains only the `Total Revenue` and `Region` columns by `Region` and then obtaining the 2 largest of each Region. The result will be a new `DataFrame` with only the top 2 spenders from each unique specialty.


<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
grouped_by_region = df.loc[:, ["Region", "Total Revenue"]].groupby("Region")
grouped_by_region["Total Revenue"].nlargest(2).head(n=4)
```

Output:
```python
Region                   
Asia                   16    3039414.40
                       18    2559474.10
Australia and Oceania  0     2533654.00
                       5      759202.72
Name: Total Revenue, dtype: float64
```
</div>
    
Though `pandas` has the more common and basic aggregation, transformation, and thinning methods implmented for us, they could not possibly cover all cases. Therefore cases that do not fit into any one of these categories may be carried out by using the more flexible `apply() GroupBy` method. `apply()` takes as input a function name, which it calls on each group of the calling `GroupBy` object.

For example suppose we wanted to thin our dataset so that there are only 50% of each specialty represented. To do this we can define a new function, we will call it, `sample_50p`, and this function will utilize the `sample()` `DataFrame` method. The `sample()` `DataFrame` method will take a parameter `frac` that specifies the fraction of the original `DataFrame` that is to be returned. We can then use the `apply()` method and pass it the name of our newly defined function to obtain a new `DataFrame` that is filtered at the group specific level. 

<div class="alert alert-secondary" role="alert" markdown="1">

Code:
```python
def sample_50p(x):
    return x.sample(frac=0.5)
grouped_by_region = df.loc[:, ["Region", "Total Revenue", "Order Priority"]].groupby("Region")
grouped_by_region.apply(sample_50p).head(n=5)
```

Output:

|                              |       | Region                   | Total Revenue | Order Priority |
| Region                       |       |                          |               |                |
|------------------------------|-------|--------------------------|---------------|----------------|
| Asia                         | 10    | Asia                     | 19103.44      | H              |
|                              | 18    | Asia                     | 2559474.10    | L              |
| Australia and Oceania        | 0     | Australia and Oceania    | 2533654.00    | H              |
| Central America and the Caribbean | 1 | Central America and the Caribbean | 576782.80 | C              |
| Europe                       | 15    | Europe                   | 182825.44     | M              |
{: .table}
</div>












<div class="alert alert-success mt-3" role="alert" markdown="1">
<i class="fa-solid fa-globe fa-xl"></i> **Key Points**
<hr/>
**Function Application and Mapping**

* **Global Processing**
  * To apply a function to every row or column in a `DataFrame` we can use the `apply()` method
  * To apply a function to every element in a `Series` we can use the `map()` method
  * To apply a function to every element in a `DataFrame` we can use the `applymap()` method
  
* **Group Specific Processing**

  * The `groupby()` method is used to group the data using values on one or more columns

  * `groupby()` is often applied in the context of the data processing paradigm called "split-apply-combine"
    * **Split**: you need to split the data into chunks defined using one or more columns
    * **Apply**: apply some operation on the chunks generated. 
    * **Combine**: combine the results of the applied operation into a new `DataFrame`

  * There are 3 common classes of split-apply-combine operations that can be applied to group data.

    1. __Aggregations__ generate a single value for each group
  
    2.  __Transformations__ convert the data and generate a group of the same size as the original group.

    3.  __Filters__ retain or discard a group based on group-specific boolean computations.
</div>



<div class="alert alert-info" role="alert" markdown="1">
<i class="fa-solid fa-circle-info fa-xl"></i> **For more information**
<hr/>
You can read more about `groupby` from the official pandas documentation of [groupby](https://pandas.pydata.org/docs/user_guide/groupby.html)
</div>


{% include next-button.html
top-label="7. Real Example Cleanup>"
bottom-label="3:50pm"
url="/morea/data-wrangling-2/experience-real-example-cleanup.html" %}