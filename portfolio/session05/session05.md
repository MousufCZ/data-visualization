---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 5: Shaping Data

## Table of Contents

1.  [Introduction](#1-introduction)
2.  [Manual Editing](#2-manual-editing)
3.  [Literate Data](#3-literate-data)
4.  [Tidy Data](#4-tidy-data)
5.  [Transformations in Vega-Lite](#5-transformations-in-vega-lite)
6.  [Conclusions](#6-conclusions)
7.  [Recommending Reading](#7-recommended-reading-and-listening)
8.  [Practical Exercises](#8-practical-exercises)

{(aim|}

This session is designed to show how you can manipulate data and 'shape' tables so they are more amenable to encoding for visualization.

By the end of this session you should be able to:

- Recognise and adopt three different approaches to data shaping – externally; within Elm; and within Vega-Lite.
- Create _literate data_ that embed data sources in a narrative that explains how the data have been shaped before visualizing.
- Tidy data into tables where each column is a single _variable_ and each row is an _observation_.
- Use the [Tidy package](https://package.elm-lang.org/packages/gicentre/tidy/latest/) to tidy data programmatically in Elm.
- Use elm-vegalite's [transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#transform) function to specify data transformations within a visualization specification.

{|aim)}

---

## 1. Introduction

You may recall from the first session that elm-vegaLite (and Vega, Vega-Lite, ggPlot2 and others) is structured around Wilkinson's _Grammar of Graphics_ system. This framework identifies seven transformation steps between a dataset and its eventual visualization. Now that you are a little more familiar with specifying visualizations in elm-vegalite, let's reconsider the model along with some of the functions you have used to achieve each transformation step:

![grammar of graphics](https://staff.city.ac.uk/~jwo/datavis2021b/session05/images/grammarOfGraphics.png?q=0)

You have generated specifications involving all but one of the transformation steps – what Wilkinson refers to as _algebra_. This session will consider those first two steps of assembling the data into variables and combining them with algebra, in a process we might more generally refer to as _data shaping_.

You have several alternatives when assembling data in a form ready for the _scales_ stage and beyond.

![data shaping options](https://staff.city.ac.uk/~jwo/datavis2021b/session05/images/shapingOptions.png)

Option 1 is to edit the dataset(s) externally to elm-vegalite, for example by loading the data into a spreadsheet and deleting the columns you do not need, replacing missing values, calculating derived variables etc. This option may be appropriate if there is not much editing to do and/or you are not feeling confident applying the other options. This is also a useful approach if you are familiar with other programming approaches to data creation (e.g. Python code used by data scientists). But this 'loose coupling' of data and visualization tends not to scale well for larger datasets or for cases where the input dataset might change over the lifetime of the visualization (imagine if you had to produce a daily report based on data that are updated every day).

Option 2 involves writing Elm functions in a litvis document that do the data editing so that you can automate the process of shaping the data ready for use by elm-vegalite. If you document how the data have been edited in a litvis document, we can refer to this as the process of creating _literate data_. This has the advantage of making it clearer to others how data have been manipulated before being visualized while also being scalable to larger and more complex datasets.

Option 3 is to use Vega-Lite's data transformation functions to do the data shaping. This has the advantage of working well with external data specified via [dataFromUrl](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataFromUrl) where you do not have local access to the data. It is an attractive option for data that are dynamically updated externally to your visualization. It has the disadvantage that the range of data shaping options is more limited than those available via option 2.

### A Titanic dataset

To illustrate some of the differences between the three options, consider the following dataset describing the passengers of the Titanic who either survived or lost their lives in the 1912 disaster. The same data are available in three different formats:

- [Excel spreadsheet](https://staff.city.ac.uk/~jwo/datavis2021b/session05/titanic.xlsx)
- [CSV (comma separated values) file](https://gicentre.github.io/data/titanic/titanic.csv)
- [JSON file](https://gicentre.github.io/data/titanic/titanic.json)

{(infobox|}

The Titanic dataset is one of the "classic" datasets used repeatedly in data science fields for learning and testing analysis and visualization (e.g. [the Kaggle machine learning challenge](https://www.kaggle.com/c/titanic)). It has a [long history of being used in data visualization](http://www.datavis.ca/papers/JSM-2019-proceedings-final.pdf). Other examples include [Iris morphology data](https://en.wikipedia.org/wiki/Iris_flower_data_set), the [dataExpo 1983 cars dataset](http://stat-computing.org/dataexpo/1983.html) and [many others](https://archive.ics.uci.edu/ml/datasets.php).

While the data have already been highly investigated, they provide useful benchmarks for comparing your visualization designs (and analysis) with those of others. Although there are [disadvantages in using these data](https://medium.com/multiple-views-visualization-research-explained/doing-more-with-sample-datasets-d9ea622cecd7).

{|infobox)}

Viewing the spreadsheet version of the data we see many of the common problems associated with the data we typically have to handle:

![Titanic Spreadsheet](https://staff.city.ac.uk/~jwo/datavis2021b/session05/images/titanicSpreadsheet.png)

Some of these problems we can ignore, while others will need some form of data editing or shaping. How we do this will depend on which of the three options above we might take to work with the data.

The CSV version of the data is a simple transformation of the tabular format with column headings represented by the first line of comma-separated values and the data in subsequent lines:

```txt
pclass,survived,name,sex,age,sibsp,parch,ticket,fare,cabin,embarked,boat,body,home.dest
1,1,"Allen, Miss. Elisabeth Walton",female,29,0,0,24160,211.3375,B5,S,2,,"St Louis, MO"
1,1,"Allison, Master. Hudson Trevor",male,0.9167,1,2,113781,151.55,C22 C26,S,11,,"Montreal, PQ / Chesterville, ON"
1,0,"Allison, Miss. Helen Loraine",female,2,1,2,113781,151.55,C22 C26,S,,,"Montreal, PQ / Chesterville, ON"
1,0,"Allison, Mr. Hudson Joshua Creighton",male,30,1,2,113781,151.55,C22 C26,S,,135,"Montreal, PQ / Chesterville, ON"
etc.
```

The JSON version captures each table row as an _object_ in `{ }` braces, naming the 'column' each data item belongs to, separated from its value by a `:` . All the objects are stored in an _array_ (list) indicated by square brackets and separated by commas:

```javascript
[
  {
    "pclass": 1,
    "survived": 1,
    "name": "Allen, Miss. Elisabeth Walton",
    "sex": "female",
    "age": 29,
    "sibsp": 0,
    "parch": 0,
    "ticket": 24160,
    "fare": 211.3375,
    "cabin": "B5",
    "embarked": "S",
    "boat": 2,
    "body": "",
    "home.dest": "St Louis, MO"
  },
  {
    "pclass": 1,
    "survived": 1,
    "name": "Allison, Master. Hudson Trevor",
    "sex": "male",
    "age": 0.9167,
    "sibsp": 1,
    "parch": 2,
    "ticket": 113781,
    "fare": 151.55,
    "cabin": "C22 C26",
    "embarked": "S",
    "boat": 11,
    "body": "",
    "home.dest": "Montreal, PQ / Chesterville, ON"
  },
  etc.
]

```

The metadata describing the meaning of each variable are not directly part of the files, but are described below:

| column    | description                                                    |
| --------- | -------------------------------------------------------------- |
| pclass    | Ticket class (1st, 2nd, 3rd)                                   |
| survived  | 1=yes, 0=no                                                    |
| name      | Recorded name of passenger                                     |
| sex       | male or female                                                 |
| age       | age of passenger                                               |
| sibsp     | Number of siblings/spouses aboard                              |
| parch     | Number of parents/children aboard                              |
| ticket    | Ticket number                                                  |
| fare      | Passenger fare in £ sterling                                   |
| cabin     | Cabin location                                                 |
| embarked  | Embarkation location: S=Southampton; C=Cherbourg; Q=Queenstown |
| boat      | Lifeboat                                                       |
| body      | Body identification number                                     |
| home.dest | Home or destination location                                   |

Let's consider a research question we might want to answer with the data and some visualization:

- **What is the the relationship (if any) between likelihood of survival, gender and ticket class?**

and a secondary question:

- **To what extent does this relationship (if it exists) vary with age of passenger?**

## 2. Manual Editing

We might start by creating a simple bar chart showing the number of men and of women who died following the sinking, grouping by the class of ticket they held. Following option 1, we could create a new dataset containing just the relevant data. This involves removing the unused columns and requires us to select only the rows of the table where `survived` was equal to 0 (conveniently achieved with Excel's _Data->Filter_ option). This is the _variables_ stage of Wilkinson's _Grammar of Graphics_.

The start of such a table might look like this:

![Titanic spreadsheet edit1](https://staff.city.ac.uk/~jwo/datavis2021b/session05/images/titanicSpreadsheetEdit1.jpg)

Assuming we saved a copy of the table as a CSV file in the same folder as our litvis document (the example below uses an external URL, but you should be able to save a local copy), we could then create a specification to show the numbers of deaths of men and women grouped by ticket class:

```elm {l}
fmColours =
    categoricalDomainMap
        [ ( "female", "rgb(180,90,90)" )
        , ( "male", "rgb(100,130,170)" )
        ]
```

```elm {l v}
barsByClass : Spec
barsByClass =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/titanic/titanicEdit1.csv" []

        enc =
            encoding
                << position X
                    [ pName "pclass"
                    , pAxis [ axTitle "Passenger class", axLabelAngle 0, axTicks False ]
                    ]
                << position Y [ pAggregate opCount, pTitle "Number of deaths" ]
                << color [ mName "sex", mScale fmColours, mTitle "" ]
    in
    toVegaLite [ width 180, data, enc [], bar [] ]
```

For simple data editing/shaping, this approach of selecting variables from a spreadsheet or file works reasonably well. Now lets consider that secondary research question and examine the age profile of those who died. We again need to edit the spreadsheet to select those who did not survive, but this time include the `age` column, remove rows that do not have age data, then save as a new CSV file:

![Titanic spreadsheet edit2](https://staff.city.ac.uk/~jwo/datavis2021b/session05/images/titanicSpreadsheetEdit2.jpg)

```elm {l v}
barsByAge : Spec
barsByAge =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/titanic/titanicEdit2.csv" []

        enc =
            encoding
                << position X [ pName "age", pOrdinal, pBin [ biStep 5 ] ]
                << position Y [ pAggregate opCount, pTitle "Number of deaths" ]
                << color [ mName "sex", mScale fmColours, mTitle "" ]
    in
    toVegaLite [ width 300, data, enc [], bar [] ]
```

Again, this works, but the task of editing and saving a CSV every time we want to reshape the data becomes rather tedious and error prone. And looking at the files created, the history of its edits are not stored, so tracking the lineage, or _provenance_, of derived datasets can be difficult or impossible. To do that, we need an alternative approach.

## 3. Literate Data

If Literate Visualization is the process of creating visualization, but with added textual narrative to justify design choices, _literate data_ can be regarded as a specification of data to use combined with a textural narrative justifying any data shaping.

This requires us to specify the data 'inline' with code rather than read it externally. In its simplest form, we can achieve this with the functions [dataFromColumns](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataFromColumns) or [dataFromRows](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataFromRows).

For example,

```elm {l v highlight=[5,6,7]}
inlineExample : Spec
inlineExample =
    let
        data =
            dataFromColumns []
                << dataColumn "a" (strs [ "A", "B", "C", "D", "E", "F", "G" ])
                << dataColumn "b" (nums [ 28, 55, 43, 91, 81, 53, 19 ])

        enc =
            encoding
                << position X [ pName "a" ]
                << position Y [ pName "b", pQuant ]
    in
    toVegaLite [ data [], enc [], bar [] ]
```

Real datasets are likely to be much larger and so in litvis, we can store the data directly in code blocks in their own `.md` file. To assist with the process, we can import the [tidy package](https://package.elm-lang.org/packages/gicentre/tidy/latest/) I've written specifically for shaping tabular data with Elm.

The package contains a couple of useful data import functions allowing us to convert CSV and JSON files into _tables_ ready for shaping and output to elm-vegalite. For example, the simple case above could be represented as a literate data markdown file as follows:

### `simpleData.md`

````txt
---
id: litvis

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

## Simple data example

You might provide a description of the data source / provenance here.

And a tabular summary here:

```elm {m}
showTable : List String
showTable =
    tableSummary 4 myDataTable
```

## Data Shaping

Details of any data shaping with an explanation could go here.

<!-- The raw data, imported with Tidy's fromCSV function: -->

```elm {l=hidden}
myDataTable : Table
myDataTable =
    """a,b
A,28
B,55
C,43
D,91
E,81
F,53
G,19"""
        |> fromCSV
```
````

It contains the normal frontmatter we have seen in our litvis documents, but additionally importing the `gicentre/tidy` package. The raw data are provided at the bottom of the file enclosed in multi-line quotation marks (a pair of triple `"""`) and then converted into a [Table](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#Table) with the tidy function [fromCSV](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#fromCSV) (note the use of the piping operator `|>` to avoid using brackets, as considered in last week's coding gym).

The tidy package provides a useful function [tableSummary](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#tableSummary) for displaying the first _n_ lines of a table in the formatted view (4 in the example above). This is especially useful way of explaining / checking more sophisticated data shaping operations. `tableSummary` creates a list of strings (i.e. text) that is in markdown, formatted as a table. Litvis allows you to format the markdown produced by an elm function by using the `{m}` header in the code block instead of the more usual `{l}`, `{v}` or `{r}`. This `m` (for 'markdown') header is particularly useful when showing tables in a formatted document.

We can keep the visualization design process separate from the literate data process by creating a separate document that _follows_ the data document, just as we did when creating branching narratives:

### `simpleVisualization.md`

````txt
---
id: litvis
follows: simpleData
---

@import "../css/datavis.less"

## Simple Visualization

```elm {l v}
initialDesign : Spec
initialDesign =
    let
        data =
            dataFromColumns []
                << dataColumn "category" (strColumn "a" myDataTable |> strs)
                << dataColumn "value" (numColumn "b" myDataTable |> nums)

        enc =
            encoding
                << position X [ pName "category" ]
                << position Y [ pName "value", pQuant ]
    in
    toVegaLite [ data [], enc [], bar [] ]
```
````

The data are transferred from the `Table` into elm-vegalite with the Tidy functions [strColumn](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#strColumn) (for extracting a column of strings from a table) and [numColumn](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#numColumn) (for extracting numeric values from a table).

For this simple example, splitting the visualization into two documents and storing the data in a Tidy [Table](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#Table) is probably an over-engineered solution. But for real, more complex, datasets, the benefits should be clearer. Here, for example, are Titanic data and visualization documents:

- [titanicData.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/titanicData.md)
- [titanicVisualization.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/titanicVisualization.md)

{(task|} View `titanicData.md` and `titanicVisualization.md` in litvis to see how the data are presented in the preview window.{|task)}

The [Tidy package](https://package.elm-lang.org/packages/gicentre/tidy/latest/) contains many useful functions for shaping data. Some that are used in the titanic example include:

- [filterColumns](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#filterColumns). This allows you to create a new table by selecting subset of columns from an existing table. Which columns are selected depends on a function you provide that, given a column name should return either `True` or `False` depending on whether it should be selected. For example:

  ```elm
  filterColumns (\c -> c == "pclass" || c == "age") table
  ```

  is saying, select the columns from `table` where the column name is either `pclass` or `age`. For more examples of creating Boolean expressions like this, see this session's coding gym.

- [filterRows](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#filterRows) is similar except that the function we need to provide tests each cell in a given column and only selects rows in which this cell test is true. For example:

  ```elm
  filterRows "survived" (\v -> v == "0")
  ```

  is saying, select all rows from the table where the value in column `survived` in the row is equal to 0.

Together, these functions provide the ability to select almost any arbitrary subset of a table determined by the names of the table columns or the values in table cells.

{(infobox|}It is good practice when creating literate data documents to identify any data shaping, such as filtering rows and columns, so that should you (or someone else) wish to verify the integrity of the data, it is clear how the data have been created/shaped before being visualized.{|infobox)}

### Importing JSON Data

The examples we have considered so far have the source data already in tabular format. This makes it easy to generate .csv files and `Tables` from which we can extract the columns for visualization. However, some data, and especially web-based _APIs_, will be in hierarchical JSON format.

To create a table from a JSON file we build it up, column at a time, extracting the relevant field from the json file. For example:

- [titanicJsonData.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/titanicJsonData.md)

As with the .CSV version, we store the data file itself between multi-line quotes (`"""`), but this time we have named the entire string `json` so we can access it multiple times with [insertColumnFromJson](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#insertColumnFromJson):

```elm
let
    json = """...""" -- Full JSON file would be here
in
empty
    |> insertColumnFromJson "pclass" [] json
    |> insertColumnFromJson "survived" [] json
    |> insertColumnFromJson "sex" [] json
    |> insertColumnFromJson "age" [] json
```

Now we no longer need to filter out all the other columns we don't need. We still need to filter the rows though, so we only have records where `survived==0` and `age` is not blank:

```elm
deathsTable : Table
deathsTable =
    fullTable
        |> filterRows "survived" (\v -> v == "0")
        |> filterRows "age" (\v -> v /= "")
        -- Don't need this column once we've filtered rows
        |> filterColumns (\c -> c /= "survived")
```

## 4. Tidy Data

You may be wondering why the package for creating tables is called `Tidy` (Data Science students may already realise this as the concept of 'tidy data' should be familiar to you).

'Tidy data' is an approach to organising data into tables such that columns always refer to _variables_ and rows always refer to _observations_. As a consequence, individual columns and individual rows are independent of one another and can be placed in any order without changing the semantics of the data.

Keeping data tidy makes them _much_ easier to visualize, process and share. The principles of Tidy data organisation were first proposed by [Wickham, 2014](#7-recommended-reading-and-listening) and have subsequently become the basis for the [tidyverse](https://www.tidyverse.org) approach to using the statistics package [R](https://www.r-project.org). The concept is probably best illustrated by showing how 'messy' tables can be transformed into a 'tidy' ones.

### Drug treatment

Imagine a small clinical trial where several patients are given two different courses of treatment after which some outcome is measured. We might represent the table as follows (from Table 1 in [Wickham, 2014](#7-recommended-reading-and-listening)):

| person       | treatmentA | treatmentB |
| ------------ | ---------- | ---------- |
| John Smith   |            | 2          |
| Jane Doe     | 16         | 11         |
| Mary Johnson | 3          | 1          |

This table organisation might be appropriate for tabular display. But equally, we might have arranged the same data differently:

| treatment  | John Smith | Jane Doe | Mary Johnson |
| ---------- | ---------- | -------- | ------------ |
| treatmentA |            | 16       | 3            |
| treatmentB | 2          | 11       | 1            |

Both are "correct" in the sense they capture the data unambiguously, but each would require a different process of data extraction in order to analyse or visualize the data. This is largely because those outcome results (represented by the numbers 16, 3, 2, 11 and 1) are spread across both rows and columns, so any variations in table arrangement require variations in how the values are extracted for visualization.

Instead we can arrange the table into a tidy form that requires just one, and one only, procedure for extracting the data values:

^^^elm{m=(tableSummary -1 tidyTreatment)}^^^

Each row is an observation containing all the necessary data associated with it. As such it doesn't matter in what order the rows are represented or extracted. Each column contains just one variable, which we might encode with a visualization channel, again independently of the order of columns in the table.

The process of arranging table values into the 'tall-thin' tidy table is referred to as _gathering_ (and also _folding_ and _pivot_long_ in some other software), with the Tidy package providing a function to do this programmatically:

```elm {l}
tidyTreatment : Table
tidyTreatment =
    """Person,treatmentA,treatmentB
       John Smith, , 2
       Jane Doe, 16, 11
       Mary Johnson, 3, 1"""
        |> fromCSV
        |> gather "treatment" "result" [ ( "treatmentA", "a" ), ( "treatmentB", "b" ) ]
```

^^^elm{m=(tableSummary -1 tidyTreatment)}^^^

The Tidy [gather](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#gather) function takes four parameters. The first two are the names you will give the two new columns to be created containing the variable categories (corresponding to column names in the original messy table) and variable values (corresponding to the values that are spread across rows and columns in the messy table). The third parameter is a list of _tuples_ that pair messy column headings with the name you wish to give them in the new category column. The final parameter is the name of the table to transform.

### Tuberculosis cases

For a more realistic example, consider this table of tuberculosis prevalence across the countries of the world, subdivided into age and gender categories (from Table 9 in [Wickham, 2014](#7-recommended-reading-and-listening)). The column headings `m014`, `m1524`, `f014` etc. refer to male/female (`m` and `f`) age cohorts (0-14, 15-24 etc.) and the numbers in each cell, the numbers of people diagnosed with TB in those cohorts for each listed country.

^^^elm{m=(tableSummary 8 tbTable)}^^^

Its compact format is useful for data entry and direct display as a table, but is problematic if we wish to create visualizations, say, of the male and female age distributions of TB from this table. If we were to create a bar chart, we would need to position the bars by age cohort, their height by the number of TB cases and colour by the numbers of male and female cases. Yet the values we need are spread across multiple rows and columns so it is not clear which to encode (remember each channel encodes a single column of data). In short, the data are messy.

Consider how many distinct variables we have (which could each theoretically be encoded with a separate channel):

- Country
- Year of observation
- Number of TB cases
- Age cohort
- Gender

A tidy table would represent each of these 5 variables in a table of 5 columns:

^^^elm{m=(tableSummary 12 tbTidy)}^^^

Again, we can use [gather](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#gather) to rearrange the original messy table into the tidy one, additionally filtering out any rows with no `cases` values.

```elm {l}
tbGathered : Table
tbGathered =
    tbTable
        |> gather "cohort"
            "cases"
            ([ "m014", "m1524", "m2534", "m3544", "m4554", "m5564", "m65", "f014", "f1524", "f2534", "f3544", "f4554", "f5564", "f65" ]
                |> List.map (\s -> ( s, s ))
            )
        |> filterRows "cases" ((/=) "")
```

^^^elm{m=(tableSummary 4 tbGathered)}^^^

Gathering places the cases into a single column as we would like, but we still have a problem in that the single `cohort` column contains both gender and age combined. To make the table tidy we need to ensure each column contains only a single variable, so we need to split `cohort` into separate `age` and `gender` columns. We can do this with another Tidy function – [bisect](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#bisect).

```elm {l}
tbTidy : Table
tbTidy =
    tbGathered
        |> bisect "cohort"
            (\s -> ( String.left 1 s, String.dropLeft 1 s ))
            ( "gender", "age" )
```

^^^elm{m=(tableSummary 4 tbTidy)}^^^

Here we use Elm's string functions [String.left](https://package.elm-lang.org/packages/elm/core/latest/String#left) and [String.dropLeft](https://package.elm-lang.org/packages/elm/core/latest/String#dropLeft) to extract the first character from the string (`m` or `f`) and the remaining characters (e.g. `1524`) and place them in separate `gender` and `age` columns.

Now we are in a position to visualize data from the table:

```elm {l v}
tbDist : Spec
tbDist =
    let
        data =
            dataFromColumns []
                << dataColumn "cases" (numColumn "cases" tbTidy |> nums)
                << dataColumn "age" (strColumn "age" tbTidy |> strs)
                << dataColumn "gender" (strColumn "gender" tbTidy |> strs)

        genderColours =
            categoricalDomainMap
                [ ( "f", "rgb(180,90,90)" )
                , ( "m", "rgb(100,130,170)" )
                ]

        enc =
            encoding
                << position X [ pName "age" ]
                << position Y [ pName "cases", pAggregate opSum ]
                << color [ mName "gender", mScale genderColours, mLegend [] ]
    in
    toVegaLite [ width 200, data [], enc [], bar [] ]
```

### Joining Tables

One of the (many) advantages of dealing with tidy data tables is that it makes the task of joining tables together much simpler.

Suppose we have a table representing the main ingredient of some selected food products:

```elm {l=hidden}
t1 : Table
t1 =
    """food,ingredient
    pizza,wheat
    burger,beef
    latte,coffee"""
        |> fromCSV


t2 : Table
t2 =
    """ingredient,emissions
        beef,60
        lamb,24
        cheese,21
        chocolate,19
        coffee,17
        milk,3
        sugar,3
        vegetable oil,3
        wheat,1.4"""
        |> fromCSV
```

^^^elm {m=(tableSummary -1 t1)}^^^

And additionally, suppose we have a data table showing [greenhouse gas emissions across the supply chain](https://ourworldindata.org/food-choice-vs-eating-local) for some common ingredients:

^^^elm {m=(tableSummary -1 t2)}^^^

We could join the two tables together with a [left join](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#leftJoin). That is, we create a new table based on the values of the first (left) table that are also found in the second (right):

```elm {l}
t3 : Table
t3 =
    leftJoin ( t1, "ingredient" ) ( t2, "ingredient" )
```

^^^elm {m=(tableSummary -1 t3)}^^^

This joined table could form the input into our visualization specification, extracting the table's column values with [numColumn](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#numColumn) and [strColumn](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#strColumn) as part of a [dataFromColumns](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataFromColumns) expression.

For a more realistic example, consider the process of linking tables that contain data and geographic location as represented by these literate data and literate visualization documents:

- [badTeethData.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/badTeethData.md)
- [badTeethVisualization.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/badTeethVisualization.md)

As a final JSON example, consider a more complex JSON file, typical of those generated by web-based _data feed APIs_ and a process of literate data shaping that includes filtering, tidying and table joining:

- [tflBikeData.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/tflBikeData.md)
- [tflDamagedBikes.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/tflDamagedBikes.md)

{(task|}

Make sure you view the TfL example in litvis to see the results. Find where filtering of rows and columns have been used to extract the data and where column insertion is used join columns together in a table.

{|task)}

{(infobox|}

As an aside, you find it useful (and fun) to practice your functional shaping skills with the [Cube composer game](https://david-peter.de/cube-composer/). This may be especially familiar to those taking IN3043 Functional Programming.

{|infobox)}

## 5. Transformations in Vega-Lite

Creating tidy tables and documenting the process of data shaping is an efficient and transparent way of organising data and I encourage you to use this approach where possible. However sometimes it is not possible or practical to keep local copies of a dataset. For example, when data are generated live via a web API. In those cases, we are required to reference the data with [dataFromUrl](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataFromUrl).

If we don't have a local copy of the data, can we perform data shaping? The answer is yes, although to a more limited extent than via Elm and Tidy. Instead, we rely on Vega-Lite's own data transformation functions.

As a simple example, consider the Titanic dataset again, this time read from a URL. We can filter rows of the table using elm-vegalite's [transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#transform) and [filter](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#filter) functions:

```elm {v l highlight=[7,8,9,20]}
histo : Spec
histo =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/titanic/titanic.json"

        trans =
            transform
                << filter (fiExpr "datum.survived == 0")

        enc =
            encoding
                << position X
                    [ pName "pclass"
                    , pAxis [ axTitle "Passenger class", axLabelAngle 0, axTicks False ]
                    ]
                << position Y [ pAggregate opCount ]
                << color [ mName "sex", mScale fmColours, mLegend [ leTitle "" ] ]
    in
    toVegaLite [ width 180, data [], trans [], enc [], bar [] ]
```

The filter function takes an [fiExpr](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#fiExpr) expression that specifies the conditions required to keep the data. To refer to any data value, we prefix the name of the data field (i.e. column) with `datum.` in the expression.

Here is the same approach, applied to show the distribution of the age and gender of survivors:

```elm {l v highlight=[9]}
survivorsByAge : Spec
survivorsByAge =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/titanic/titanic.json"

        trans =
            transform
                << filter (fiExpr "datum.survived == 1 && datum.age > 0")

        enc =
            encoding
                << position X [ pName "age", pOrdinal, pBin [ biStep 5 ] ]
                << position Y [ pAggregate opCount ]
                << color [ mName "sex", mScale fmColours, mLegend [ leTitle "" ] ]
    in
    toVegaLite [ width 300, data [], trans [], enc [], bar [] ]
```

Note that the expression combines two Boolean expressions with `&&` (and) – one to select survivors and another to select only those records with an age recorded (which by definition will have an age greater than zero).

We've only just scratched the surface of possible Vega-Lite transform functions that can help with deriving, editing and shaping data from within Vega-Lite. For some other examples, see the Vega-Lite and elm-vegalite documentation:

| Description                                                                                                                              | Vega-Lite transform                                                                             | elm-vegalite function                                                                                                                                                                               |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Create new columns based on values of existing ones                                                                                      | [calculate](https://vega.github.io/vega-lite/docs/calculate.html)                               | [calculateAs](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#calculateAs)                                                                                              |
| _Gather_ data values into two new fields (like [Tidy gather](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#gather))    | [fold](https://vega.github.io/vega-lite/docs/fold.html)                                         | [fold](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#fold)                                                                                                            |
| _Spread_ data values into columns (like [Tidy spread](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#spread))           | [pivot](https://vega.github.io/vega-lite/docs/pivot.html)                                       | [pivot](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pivot)                                                                                                          |
| Perform a relational join of two tables (like [Tidy leftJoin](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#leftJoin)) | [lookup](https://vega.github.io/vega-lite/docs/lookup.html)                                     | [lookup](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookup)                                                                                                        |
| Split strings into multiple columns                                                                                                      | calculate <[string expression](https://vega.github.io/vega/docs/expressions/#string-functions)> | calculateAs <[string expression](https://vega.github.io/vega/docs/expressions/#string-functions)>                                                                                                   |
| Split strings into multiple rows                                                                                                         | [flatten](https://vega.github.io/vega-lite/docs/flatten.html)                                   | [flatten](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#flatten) / [flattenAs](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#flattenAs) |
| Interpolate missing values                                                                                                               | [impute](https://vega.github.io/vega-lite/docs/impute.html)                                     | [impute](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#impute)                                                                                                        |
| Sample a subset from some data values                                                                                                    | [sample](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#impute)    | [sample](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#sample)                                                                                                        |
| Derive new values from adjacent ones in a data column                                                                                    | [window](https://vega.github.io/vega-lite/docs/window.html)                                     | [window](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#window)                                                                                                        |

## 6. Conclusions

While this session has concentrated on shaping and extracting data tables rather than visual design, the two processes are strongly linked. As Wilkinson's Grammar of Graphics system describes, they are both part of the same set of transformations that you perform when turning data into something visual.

Being able to shape data into the form suitable for encoding with channels is an important skill, and one that will greatly increase the range of data sources that you can make use of in your own visualization designs.

The approach you take to shaping data will depend on what needs to be shaped, how you have access to the data, and your own expertise in programmatic data shaping. I would strongly encourage you to adopt the process of _literate data_ where you can, providing a transparent account of how you shape the data sources that form the basis of your visual designs.

## 7. Recommended Reading and Listening

[![The Sinking of the Titanic](https://staff.city.ac.uk/~jwo/datavis2021b/session05/images/bryarsAlbum.png) The music playing at the start of the live session.](<https://en.wikipedia.org/wiki/The_Sinking_of_the_Titanic_(Bryars)>)

**Tidyverse** (2019) [R Packages for Data Science](https://www.tidyverse.org) _(Some additional context for tidy data, in this case using the R statistics package)_

**Wickham, H.** (2014) [Tidy Data](https://www.jstatsoft.org/index.php/jss/article/view/v059i10/v59i10.pdf), _Journal of Statistical Software_, 59(10).

**Wilkinson, L.** (2010) [The Grammar of Graphics: Wiley interdisciplinary reviews](https://go.exlibris.link/2pPxZX51). _Computational Statistics_, 2010 (2), pp.673–677.

## 8. Practical Exercises

{(task|} Please complete the following exercises and push your answers to your github repo. {|task)}

### 1. Tidying Data Tables

Which of the following tables are in 'tidy' format? For those that are not, provide a tidy version of the table (you can do this manually by adding markdown tables to this document – there's no need to use elm code to do so.)

#### (a) UK General Election Results 2019

| Party                     | percentVote | numMPs |
| :------------------------ | ----------- | ------ |
| Conservative              | 43.6        | 365    |
| Labour                    | 32.2        | 202    |
| Scottish National Party   | 3.9         | 48     |
| Liberal Democrats         | 11.6        | 11     |
| Democratic Unionist Party | 0.8         | 8      |

#### (b) Tokyo 2021 Paralympic Medal Table

| Rank | Country | NumGold | NumSilver | NumBronze |
| ---- | :------ | ------- | --------- | --------- |
| 1    | China   | 96      | 60        | 51        |
| 2    | GB      | 41      | 38        | 45        |
| 3    | US      | 37      | 36        | 31        |
| 4    | RPC     | 36      | 33        | 49        |

(RPC = Russia Paralympic Committee)

#### (c) Mexico weather station temperature readings

| id      | date       | maxTemperature | minTemperature |
| ------- | ---------- | -------------- | -------------- |
| MX17004 | 2010-01-30 | 27.8           | 14.5           |
| MX17004 | 2010-02-02 | 27.3           | 14.4           |
| MX17004 | 2010-02-03 | 24.1           | 14.4           |
| MX17004 | 2010-02-11 | 29.7           | 13.4           |
| MX17004 | 2010-02-23 | 29.9           | 10.7           |
| MX17004 | 2010-03-05 | 32.1           | 14.2           |
| MX17004 | 2010-03-10 | 34.5           | 16.8           |
| MX17004 | 2010-03-16 | 31.1           | 17.6           |
| MX17004 | 2010-04-27 | 36.3           | 16.7           |
| MX17004 | 2010-05-27 | 33.2           | 18.2           |

#### (d) UK General Election Results 2017 and 2019

| Party                     | percentVote2017 | numMPs2017 | percentVote2019 | numMPs2019 |
| ------------------------- | --------------- | ---------- | --------------- | ---------- |
| Conservative              | 42.3            | 317        | 43.6            | 365        |
| Labour                    | 40.0            | 262        | 32.2            | 202        |
| Scottish National Party   | 3.0             | 35         | 3.9             | 48         |
| Liberal Democrats         | 7.4             | 12         | 11.6            | 11         |
| Democratic Unionist Party | 0.9             | 10         | 0.8             | 8          |

### 2. Tidying tables programmatically

For those tables in the previous question that are not in tidy format, create functions to create and tidy them using Tidy's [fromCSV](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#fromCSV) and [gather](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#gather) functions. Confirm you have tidied the tables correctly by displaying the tables using [tableSummary](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#tableSummary).

### 3. Data Challenge 1: Global Development Literate Data and Visualization

Using [badTeethData.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/badTeethData.md) and [badTeethVisualization.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/badTeethVisualization.md) as templates, try changing the data source from bad teeth to some other measure of global development. You can find a large selection of data at [Gapminder](https://www.gapminder.org/data/).

### 4. Data Challenge 2: Titanic Survival

Show something interesting with the Titanic dataset! Can you incorporate some of the principles of [data humanism](http://giorgialupi.com/data-humanism-my-manifesto-for-a-new-data-wold) into your design?

---

_Check your progress._

- [ ] I can manipulate data sources externally such as in a spreadsheet or (for data science students) with Python.
- [ ] I can create a separate literate data document that contains data and a description of how it has been shaped, then `follow` it to create a separate literate visualization document.
- [ ] Use the [Tidy package](https://package.elm-lang.org/packages/gicentre/tidy) to import CSV text, _gather_, _join_ and display tables.
- [ ] Use elm-vegalite's [transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#transform) and [filter](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#filter) functions to select data from within a Vega-Lite specification.

---

```elm {l=hidden}
tbTable : Table
tbTable =
    """country,year,m014,m1524,m2534,m3544,m4554,m5564,m65,f014,f1524,f2534,f3544,f4554,f5564,f65
AD,2000,0,0,1,0,0,0,0,0,0,0,0,0,0,0
AE,2000,2,4,4,6,5,12,10,3,16,1,3,0,0,4
AF,2000,52,228,183,149,129,94,80,93,414,565,339,205,99,36
AG,2000,0,0,0,0,0,0,1,1,1,1,0,0,0,0
AL,2000,2,19,21,14,24,19,16,3,11,10,8,8,5,11
AM,2000,2,152,130,131,63,26,21,1,24,27,24,8,8,4
AN,2000,0,0,1,2,0,0,0,0,0,1,0,0,1,0
AO,2000,186,999,1003,912,482,312,194,247,1142,1091,844,417,200,120
AR,2000,97,278,594,402,419,368,330,121,544,479,262,230,179,216
AS,2000,0,0,0,0,1,1,0,0,0,0,0,1,0,0
AT,2000,1,17,30,59,42,23,41,1,11,22,12,11,6,22
AU,2000,3,16,35,25,24,19,49,0,15,19,12,15,5,14
AZ,2000,0,9,24,33,42,30,0,0,3,3,6,3,0,0
BA,2000,4,56,82,99,66,58,77,4,30,46,29,29,48,124
BB,2000,0,0,0,2,0,0,0,0,0,1,0,0,0,0
BD,2000,256,3640,5643,5750,4718,3667,2837,495,3029,3238,2247,1315,778,370
BE,2000,3,20,57,39,55,32,56,6,15,15,19,4,13,27
BF,2000,12,91,274,252,133,68,65,7,59,128,101,45,38,14
BG,2000,0,13,16,20,3,9,10,0,11,14,7,3,4,6
BH,2000,0,0,3,2,5,3,4,0,1,2,0,1,1,1
BI,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
BJ,2000,19,277,428,327,213,103,74,36,239,275,149,76,45,25
BM,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
BN,2000,0,6,4,15,5,7,15,0,4,6,9,6,3,4
BO,2000,166,1182,797,518,466,340,366,191,831,588,334,254,192,233
BR,2000,1894,7268,11568,11906,8623,5085,4494,1859,6719,7215,5395,3582,2384,2496
BS,2000,1,2,7,9,4,3,2,2,5,7,8,2,3,1
BT,2000,6,65,41,30,24,12,2,7,57,34,31,23,3,2
BW,2000,25,185,605,488,267,135,96,37,335,469,262,98,57,36
BY,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
BZ,2000,2,5,7,2,6,3,5,0,2,1,2,4,1,4
CA,2000,5,34,45,46,41,32,79,4,33,40,30,25,12,66
CD,2000,485,4048,5833,4151,2549,1295,602,718,4422,5146,3309,1724,855,351
CF,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
CG,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
CH,2000,0,5,18,10,7,5,8,1,9,12,8,2,1,6
CI,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
CK,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
CL,2000,6,81,160,198,150,132,126,10,66,96,70,54,58,83
CM,2000,41,518,842,584,284,130,75,63,368,530,293,139,60,33
CN,2000,1131,19111,29399,25206,25593,21429,21771,1420,14536,18496,12377,9899,7102,6296
CO,2000,246,763,1030,963,743,610,746,194,587,758,523,381,304,510
CR,2000,14,31,53,62,39,28,49,13,21,33,24,20,23,24
CU,2000,0,71,167,90,74,55,75,2,9,22,26,22,23,39
CV,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
CY,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
CZ,2000,0,7,31,52,89,61,59,0,15,13,9,10,7,57
DE,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
DJ,2000,17,302,347,139,67,60,42,12,147,156,47,31,17,10
DK,2000,5,10,20,24,16,11,14,5,16,15,14,6,7,8
DO,2000,73,410,481,344,173,125,113,65,317,325,212,115,79,75
DZ,2000,59,927,1516,610,491,234,299,36,1005,1293,746,314,208,312
EC,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
EE,2000,0,6,31,53,56,35,15,0,9,11,14,11,4,10
EG,2000,21,641,827,667,476,307,158,55,457,343,257,211,112,48
ER,2000,9,70,75,57,32,25,20,10,100,87,71,21,12,8
ES,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
ET,2000,915,5095,5187,3082,1495,610,397,1037,4699,4424,2105,976,366,122
FI,2000,0,3,8,22,19,28,53,0,1,5,3,4,6,49
FJ,2000,0,8,6,13,5,4,2,0,7,5,7,1,4,0
FM,2000,0,2,0,1,0,0,1,4,3,1,1,0,1,1
FR,2000,10,136,248,247,211,125,244,18,108,127,89,46,43,155
GB,2000,8,86,130,96,87,75,138,9,95,114,60,31,31,67
GD,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
GE,2000,4,76,111,113,63,45,28,1,49,37,33,17,10,5
GH,2000,73,550,1266,1115,811,495,426,74,456,791,566,338,179,176
GN,2000,39,551,860,570,282,203,103,66,314,446,245,114,82,45
GR,2000,1,10,22,32,24,19,46,0,2,9,10,5,6,25
GT,2000,36,220,236,216,177,112,140,41,199,167,175,135,87,111
GU,2000,2,1,6,6,9,6,9,0,3,1,2,5,2,2
GW,2000,2,52,92,80,64,39,19,4,30,46,47,24,15,12
GY,2000,4,20,19,14,7,6,9,1,11,8,7,5,5,3
HK,2000,4,78,102,160,211,236,578,5,65,115,86,44,45,211
HN,2000,30,123,371,246,277,214,43,25,21,269,258,270,160,38
HR,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
HT,2000,67,836,898,613,350,147,118,96,914,857,513,275,132,71
HU,2000,0,8,24,85,104,58,27,1,7,17,19,22,10,30
ID,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
IE,2000,0,10,7,7,6,4,12,0,13,8,13,6,7,15
IL,2000,0,16,28,17,24,10,31,2,11,15,7,3,7,25
IN,2000,1588,20963,31090,30829,24230,15308,8534,2250,14495,17287,11768,7516,4594,2697
IQ,2000,21,627,317,297,205,135,101,37,338,241,136,134,103,87
IR,2000,29,438,467,387,295,344,642,77,593,410,322,320,407,647
IS,2000,0,0,0,0,0,0,0,0,0,1,0,0,0,0
IT,2000,12,63,96,75,58,54,112,6,38,58,33,13,19,39
JM,2000,0,6,13,13,15,6,5,1,8,8,7,2,5,1
JO,2000,0,8,16,13,9,14,2,0,8,9,1,2,2,5
JP,2000,2,246,572,676,1494,1509,3816,5,222,464,213,292,384,1958
KE,2000,264,3739,6653,3548,1630,630,414,416,3916,4363,1874,831,347,148
KG,2000,4,128,227,205,115,52,46,6,128,146,100,41,30,29
KH,2000,26,519,1323,1618,1456,1373,1058,38,457,1157,1649,1798,1459,892
KI,2000,2,9,3,3,3,8,2,2,5,6,3,4,1,3
KM,2000,0,18,7,14,9,3,4,1,9,6,12,1,2,1
KN,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
KP,2000,293,928,1508,2927,2519,1167,651,167,683,1121,2004,1524,591,357
KR,2000,19,821,1085,988,853,731,901,25,546,544,393,220,295,795
KW,2000,0,10,44,32,21,11,5,1,11,24,12,5,3,1
KY,2000,0,0,3,1,0,1,0,0,0,0,0,0,0,0
KZ,2000,36,1057,1409,1379,923,439,218,84,999,1079,599,275,202,204
LA,2000,7,92,128,166,201,177,176,10,59,95,131,122,91,71
LB,2000,5,16,28,20,15,17,14,4,31,26,9,7,4,6
LC,2000,0,0,0,1,0,1,2,0,1,0,1,0,1,0
LK,2000,25,266,459,695,793,484,360,23,312,264,176,202,144,113
LR,2000,12,133,196,127,52,17,26,21,140,149,88,28,16,16
LS,2000,8,165,458,517,395,198,76,11,222,336,195,83,36,29
LT,2000,1,38,97,145,155,74,68,0,20,37,39,32,22,48
LU,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
LV,2000,0,53,106,124,111,64,34,2,25,41,27,28,7,15
LY,2000,5,101,239,86,36,29,32,6,43,35,24,24,16,22
MA,2000,99,2061,2423,1705,855,485,595,170,1530,1121,672,398,406,352
MC,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
MD,2000,2,52,31,36,13,13,6,1,16,32,45,23,14,6
MG,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
MH,2000,3,5,4,1,3,5,3,7,7,3,0,2,2,0
MK,2000,5,8,14,20,19,20,14,1,15,14,17,5,5,10
ML,2000,23,206,430,396,297,235,144,14,174,232,152,106,75,43
MM,2000,88,1459,2636,2781,2161,1235,836,72,1040,1592,1397,987,592,378
MN,2000,6,181,260,171,68,38,23,32,200,213,113,41,26,17
MO,2000,0,10,8,25,22,9,17,0,10,4,6,6,3,13
MP,2000,1,4,8,9,9,3,2,0,10,17,7,3,1,1
MR,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
MS,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
MT,2000,0,1,0,1,1,0,1,0,0,0,0,0,0,0
MU,2000,2,6,9,18,19,14,8,1,5,8,8,6,7,4
MV,2000,0,9,10,2,5,5,3,0,11,4,5,4,5,2
MW,2000,50,653,1476,1113,585,245,114,66,1038,1481,831,401,148,64
MX,2000,214,1079,1387,1162,1235,972,1126,176,663,828,698,832,595,709
MY,2000,32,694,1138,1177,908,814,891,41,464,564,424,367,356,286
MZ,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
0,2000,18,269,874,665,300,147,81,16,352,654,348,161,76,52
NC,2000,1,1,3,4,2,3,4,1,8,1,1,3,2,4
NE,2000,29,270,174,441,252,151,78,31,123,206,168,151,63,9
NG,2000,157,2173,3164,1836,1091,566,463,239,2934,2434,1110,676,344,231
NI,2000,18,194,174,147,108,64,90,34,188,173,98,76,46,61
NL,2000,0,34,63,41,25,10,21,4,29,22,16,9,5,10
NO,2000,0,1,9,3,6,2,4,1,3,1,0,0,2,5
NP,2000,170,1904,1763,1713,1491,1294,772,176,1267,1078,833,575,419,228
NR,2000,0,0,0,0,1,0,0,0,0,0,0,1,1,0
NU,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
NZ,2000,0,6,5,6,8,10,7,1,6,6,5,0,4,10
OM,2000,1,8,9,11,12,9,11,2,17,5,7,5,11,6
PA,2000,3,44,78,61,37,27,26,6,43,34,35,19,12,16
PE,2000,552,5290,2875,1546,1041,801,796,633,3686,2472,1156,609,499,624
PF,2000,1,3,3,4,4,4,3,1,4,1,0,1,0,0
PG,2000,8,87,70,30,21,12,5,6,77,45,21,15,5,1
PH,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
PK,2000,55,498,387,256,232,153,130,130,591,416,274,163,103,56
PL,2000,1,99,303,812,782,361,434,1,99,158,211,170,82,421
PR,2000,0,1,4,19,9,10,14,1,4,5,3,7,1,3
PS,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
PT,2000,8,147,375,349,208,140,140,5,114,154,87,41,25,64
PY,2000,16,112,103,105,86,80,71,12,69,86,41,41,30,46
QA,2000,0,7,19,9,7,2,1,0,0,4,3,1,0,0
RO,2000,46,832,1508,1799,1684,916,533,53,701,766,484,341,207,321
RU,2000,1,295,526,596,402,151,54,1,43,73,74,38,31,44
RW,2000,155,466,974,824,393,129,56,105,396,473,309,109,52,14
SA,2000,0,131,268,213,158,86,107,28,172,182,79,51,50,70
SB,2000,3,13,4,8,8,10,6,8,15,13,7,7,5,2
SC,2000,0,0,2,4,1,1,0,0,0,1,0,1,1,0
SD,2000,785,1028,1511,1351,1119,638,677,817,925,1134,905,771,327,323
SE,2000,0,9,10,12,11,4,25,1,9,8,10,2,2,15
SG,2000,1,8,9,34,51,26,64,1,9,8,7,9,5,16
SI,2000,0,3,11,36,22,14,17,0,3,9,3,4,3,20
SK,2000,2,6,15,31,50,16,32,0,5,9,7,5,4,54
SL,2000,18,287,486,361,190,113,47,27,249,298,225,92,49,30
SM,2000,0,0,0,0,0,0,1,0,0,0,0,0,0,0
SN,2000,60,772,1297,857,470,279,189,77,521,540,376,217,107,61
SO,2000,113,740,724,408,254,195,142,85,354,319,219,110,72,41
SR,2000,1,6,6,3,2,0,4,2,3,6,3,0,1,1
ST,2000,1,5,11,4,7,3,10,3,7,15,5,7,4,15
SV,2000,13,99,124,114,92,62,107,28,81,76,63,63,39,47
SY,2000,8,359,289,125,86,76,55,23,195,101,53,46,38,28
SZ,2000,11,130,352,249,138,37,17,10,198,298,62,62,24,5
TC,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
TG,2000,4,101,168,144,109,48,39,13,107,124,50,36,24,15
TH,2000,27,859,2570,2380,2117,1908,2213,32,624,1035,780,873,1016,1321
TJ,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
TK,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
TM,2000,16,103,185,144,127,31,21,19,73,140,76,31,34,17
TN,2000,16,139,208,156,109,65,101,7,68,59,43,21,21,58
TO,2000,0,2,1,1,0,1,5,0,1,1,1,0,1,1
TR,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
TT,2000,0,7,18,27,17,7,7,0,5,7,9,5,2,4
TV,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
TZ,2000,200,2357,4836,3430,2022,1202,834,257,2106,3426,1738,868,494,269
UA,2000,21,693,1552,2385,2007,1062,532,41,487,590,447,298,218,405
UG,2000,283,1511,3497,2479,1279,607,395,400,1649,2782,1510,671,316,163
US,2000,6,365,602,906,904,577,738,14,246,376,349,253,152,396
UY,2000,0,36,48,45,41,30,34,2,28,22,21,13,12,16
UZ,2000,6,351,749,510,346,213,107,11,261,547,288,213,112,111
VC,2000,0,1,0,4,2,0,1,1,0,0,0,0,0,0
VE,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
VG,2000,0,0,0,0,0,1,0,0,0,0,0,0,0,0
VN,2000,51,2367,6147,8209,6713,5150,7712,64,1334,2320,2754,2594,2847,4907
VU,2000,2,7,5,1,10,5,2,5,3,15,7,3,3,1
WS,2000,0,3,1,1,1,2,1,0,2,1,1,0,0,0
YE,2000,110,789,689,493,314,255,127,161,799,627,517,345,247,92
YU,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0
ZA,2000,116,723,1999,2135,1146,435,212,122,1283,1716,933,423,167,80
ZM,2000,349,2175,2610,3045,435,261,174,150,932,1118,1305,186,112,75
ZW,2000,0,0,0,0,0,0,0,0,0,0,0,0,0,0"""
        |> fromCSV
```
