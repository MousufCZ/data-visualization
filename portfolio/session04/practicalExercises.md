---
id: litvis

narrative-schemas:
  - ../../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 4: Practical Exercises

{(task|}

Use this document as a place to add your answers to the week's practical exercises.

{|task)}


{(task|}

Remembering that data types can be classified into `Nominal`, `Ordinal` and `Quant`, which visual variables do you think are best suited to each of these measurement types? Complete the table below:

|     Visual variable | Suitable measurement types |
| ------------------: | -------------------------- |
|        _colour hue_ | Nominal                    |
| _colour saturation_ | Ordinal, Quant             |
|  _colour lightness_ | Ordinal                    |
|       _orientation_ | Nominal                    |
|             _shape_ | Nominal                    | 
|           _texture_ | Nominal                    |    -- Arrangement
|       _arrangement_ | Nominal                    |
|              _size_ | Ordinal, Quant will be hard to es             |
|             _focus_ | Ordinal, Quant             |
|          _location_ | Nominal                    |

{|task)}



{(task|}

Complete the table below by giving ratings of 1 (weak), 2 (moderate) or 3 (strong) for each of the properties of each visual variable.

(Selective - easy to pick out diffrence, associative - connection and similarities, dis)

|     Visual variable | quantitative | orderable | selective | associative | dissociative |
| ------------------: | :----------: | :-------: | :-------: | :---------: | :----------: |
|        _colour hue_ |      1       |     1     |     3     |      3      |      3       |
| _colour saturation_ |      3       |     3     |     3     |      1       |      3       |
|  _colour lightness_ |      3       |     3     |    1      |      3       |      3       |
|       _orientation_ |      1       |     1     |    3      |      2      |     2        |
|             _shape_ |      1       |     1     |    3      |      2      |     2        |
|           _texture_ |      1       |     1     |    3      |      2      |     2        |
|       _arrangement_ |      3       |     2     |    3      |      1      |     1        |
|              _size_ |      3       |     2     |    3      |      3      |     3        |
|             _focus_ |      1       |     2     |    3      |      1      |     3        |
|          _location_ |      1       |     1     |    3      |      3      |     3        |

{|task)}



{(task|}

Our ability to assemble the visual field into patterns (that hopefully have meaning in a well-designed data visualization) is dependent on a complex set of interactions between perception and cognition. Our understanding of the assembly process has greatly benefited from the early 20th century Gestalt school of psychology that attempted to formalise the we we perceive and assemble patterns in the visual field.

As an example, consider the gestalt principle of proximity, that states that perceptually we tend to group together things that are close to one another. Even small differences in proximity can affect how we assemble patterns:

What associations are implied by these arrangements of dots?

![proximity](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/gestaltProx.png)

_Adapted from Ware (2021) p.186._



{|task)}

{(annotation|} 

Our knowledge of pattern perception can be distilled into abstract design principles which state how to organize data so that important structures will be perceived. If we can map information structures to readily perceived patterns, then those structures will be more easily interpreted. (Gestalt Laws - A)

Things that are close together are perceptually grouped together.
Fig. 6.2 shows two arrays of dots that illustrate the proximity principle. Only a small change in spacing causes us to change what is perceived from a set of rows, inFig. 6.2(a), to a set of columns.

In Fig. 6.2(c), the existence of two groups is
perceptually inescapable. But proximity is not the only factor in predicting perceived
groups.

In Fig. 6.3, the dot labeled x is perceived to be part of cluster a rather than cluster b, even though it is as close to the other points in cluster b as they are to each other. 

Slocum (1983) called this the spatial concentration principle; we perceptually group regions of similar element density. The application of the proximity law in display design is straightforward.

In addition to the perceptual organization benefit, there is also a perceptual efficiency to using proximity. Because we more readily pick up information close to the fovea, less time and effort will be spent in neural processing and eye movements if related information is spatially grouped. (Ware 2021, pp.185-186)


Spatial proximity is a powerful cue for perceptual organization. A matrix of dots is perceived as rows on the left (a) and columns on the right (b). In (c) we perceive two groups of dots because of proximity relationships.

The principle of spatial concentration. The dot labeled x is perceived as part of cluster a rather than cluster b.

Ware, C. (2021) Information Visualization: Perception for Design, London: Morgan- Kaufmann

{|annotation)}


{(task|}

How much larger is the first circle than the second?

![circle area estimation](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/estimationCircles.png)

How much larger is the first rectangle than the second?

![bar area estimation](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/estimationBars.png)

Which of the two pairs of symbols did you find easier to compare? And why might this be so?

{|task)}

{(annotation|}

The circle is 4.5x diffrence. My estimation was 5. The class ranged between 3 - 1.0.

The rectangle was far easier to compare ssize visually. 

It is easier to compare the rectangle than circle shapes, it is also faster to estimate. This is becuase people are not good at estimating magnitude for different types of shapes.

Using shapes, we can let the audience to conclude the over-estimate, under-estimate and the correct estimate.

{|annotation)}

{(task|}
```elm {l}
vvTable =
    let
        table =
            """visualVariable,    q, o, s, a, d
               colour hue,        1, 1, 3, 3, 3
               colour saturation, 3, 3, 3, 1, 1
               colour lightness,  3, 3, 1, 3, 3
               orientation,       1, 1, 3, 2, 2
               shape,             1, 1, 2, 3, 3
               texture,           1, 1, 2, 3, 3
               arrangement,       3, 2, 3, 1, 1
               size,              3, 2, 3, 3, 3
               focus,             1, 1, 3, 1, 3
               location,          1, 1, 3, 2, 2"""
                |> fromCSV
                |> gather "property"
                    "rating"
                    [ ( "q", "quantitative" )
                    , ( "o", "orderable" )
                    , ( "s", "selectable" )
                    , ( "a", "associative" )
                    , ( "d", "dissociative" )
                    ]
    in
    dataFromColumns []
        << dataColumn "visualVariable" (table |> strColumn "visualVariable" |> strs)
        << dataColumn "property" (table |> strColumn "property" |> strs)
        << dataColumn "rating" (table |> numColumn "rating" |> nums)
```


Update the '1's in this function to reflect your own ratings. Then see if you can produce a graphical summary of the table by encoding the ratings with one or more channels that you think provides a descriptive graphical summary.

You can use the following as a start point:

```elm {l v}
summary : Spec
summary =
    let
        enc =
            encoding
                << position Y [ pName "visualVariable" ]
                << position X [ pName "property" ]
                << color [ mName "Dark2", mLegend [] ]
    in
    toVegaLite [ width 400, height 400, vvTable [], enc [], point [] ]
```
]

{|task)}


{(task|}
### 3. Design Challenge: Exploring data-channel design options

The [Gapminder](https://www.gapminder.org/data/) dataset at [vega.github.io/vega-lite/data/gapminder-health-income.csv](https://vega.github.io/vega-lite/data/gapminder-health-income.csv) shows the relationship between health, income and population numbers for the countries of the world.

| country             | income | health | population |
| ------------------- | ------ | ------ | ---------- |
| Afghanistan         | 1925   | 57.63  | 32526562   |
| Albania             | 10620  | 76     | 2896679    |
| Algeria             | 13434  | 76.5   | 39666519   |
| Andorra             | 46577  | 84.1   | 70473      |
| Angola              | 7615   | 61     | 25021974   |
| Antigua and Barbuda | 21049  | 75.2   | 91818      |
| Argentina           | 17344  | 76.2   | 43416755   |
| :                   | :      | :      | :          |

Create one or more litvis documents that explore some design possibilities for showing these data. In particular focus on exploring the use of different channels (e.g. position, colour, shape, size) to encode different data variables (country, income, health, population). You should come up with several different encodings and provide a brief evaluation of which ones work most effectively.

Some issues to consider:

- Are the best designs consistent with the guidance given for associating channels with data type (nominal, quant etc.) and task property (associative, orderable etc.)?
- Does double-encoding help or hinder interpretation?
- What are the advantages / disadvantages of encoding all variables in a single chart compared to separating them in multiple charts?
- Does perceptual (Flannery) scaling help in interpretation?

---

Country -   Nominal -   String  -   188 -   Ordered alphabetically
Income  -   Quant   -   int
Health  -   Ordinal -   float
Population  Quant   -   int

---

#### Part 1
Lecture 1 exercise

```elm {l v}
myBarchart1 : Spec
myBarchart1 =    
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/gapminder-health-income.csv"

        enc =
            encoding
                << position X [ pName "health", pOrdinal ]
                << position Y [ pName "income", pQuant ]
                << color [ mName "population", mQuant ]


    in
    toVegaLite [ width 640, data [], enc [], bar [] ]
```

---


#### Part 2
Lecture 2 - Line
Health and Income

```elm {l v}
myLineChart : Spec
myLineChart =
    let 
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/gapminder-health-income.csv"

        enc =  
            encoding
                << position X [ pName "health", pOrdinal ]
                << position Y [ pName "income", pQuant ]


    in
    toVegaLite [ width 640, data [], enc [], line[] ]
```      


Population and income

```elm {l v}
myLineChart1 : Spec
myLineChart1 =
    let 
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/gapminder-health-income.csv"

        enc =  
            encoding
                << position Y [ pName "country", pNominal ]
                << position X [ pName "income", pQuant ]


    in
    toVegaLite [ width 640, data [], enc [], line[] ]
```

#### Setting exercise data (elm class?)


```elm {l}
exrcData =
    dataFromUrl "https://vega.github.io/vega-lite/data/gapminder-health-income.csv"
```

```elm {l v}
line1 : Spec
line1 =
    let 
        enc =  
            encoding
                << position X [ pName "health", pOrdinal, pAxis [] ]
                << position Y [ pName "income", pQuant ]
    in
    toVegaLite [ width 640, exrcData [], enc [], line[] ]
``` 

#### Tufte
#### 1. Remove all non-data ink

Let's consider how we might apply some of Tufte's principles of information design to a default visualization produced by Vega-Lite.

Following Tufte's principle of maximising the data-ink ratio, we should ask ourselves, _can we increase the proportion of "ink" used to show the data?_

One approach is to start by removing _all_ non-data ink and then consider adding back only those elements necessary in the context of the visualization.

Add pAxis to remove lable


```elm {l v}
tufteChart : Spec
tufteChart =
    let 
        enc =  
            encoding
                << position X [ pName "health", pOrdinal, pAxis [] ]
                << position Y [ pName "income", pQuant ]
    in
    toVegaLite [ width 640, exrcData [], enc [], line[] ]
``` 



#### I dont understand CFG

Additionally, we add a configure option that allows us to modify the style of the chart. Here, coView states we wish to configure the general view of the chart, stating that it should have no thin grey border (try removing line 6 to see what effect it has).


##Why is CFG not working when I remove the cfg line?

```elm {l v}
cfgWhatIsCfg : Spec
cfgWhatIsCfg =
    let 
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])


        enc =  
            encoding
                << position X [ pName "health", pOrdinal, pAxis [] ]
                << position Y [ pName "income", pQuant, pAxis [] ]
    in
    toVegaLite [ width 640, exrcData [], enc [], bar[] ] -- I removed cfg []
``` 


#### 2. Add bar labels
While we can see the trend in bar heights, too much of the context has been lost, making it hard to know which income is represented by each bar. So we can add the labels back in again (but no need for the tick marks or baseline):

## CFG working

```elm {l v siding highlight=[7,11]}
labelAxis : Spec
labelAxis =
    let 
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])

        enc =  
            encoding
                << position X [ pName "health", pOrdinal, pAxis [] ]
                << position Y [ pName "income", pQuant, pAxis [axTitle "Insert 'Income' Here"] ]
    in
    toVegaLite [ width 640, cfg[], exrcData [], enc [], bar[] ] 
``` 


#### 3. Add bar height labels

See lecture 2 notes, due to labling diffrence, I cannot add this.

```elm {l siding highlight=[15]}
tufteChart3 : Spec
tufteChart3 =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])

        enc =
            encoding
                << position X [ pName "ageGroup", pAxis [ axTitle "" ] ]
                << position Y
                    [ pName "leave"
                    , pQuant
                    , pAxis [ axTitle "", axValues (nums [ 0.5 ]), axFormat "%" ]
```



#### 4. Improving the readability of the chart

Tufte suggests that because horizontal text is easier to read than vertical, we should endeavour to layout text horizontally where possible. In our case, Vega-Lite has (sensibly) arranged bar labels vertically as the chart is not wide enough to fit the text any other way.

By enlarging the chart, we can make sufficient room. How wide and tall should we make it? Tufte suggests that creating charts with an aspect ratio (width divided by height) of the [golden ratio](https://en.wikipedia.org/wiki/Golden_ratio) makes graphics easy to interpret and pleasing to view:


```elm {l}
goldenRatio : Float
goldenRatio =
    1.618
```

```elm {v l siding highlight=[8,21,22]}

    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])

        enc =
            encoding
                << position X [ pName "ageGroup", pAxis [ axTitle "" ] ]
                << position Y
                    [ pName "leave"
                    , pQuant
                    , pAxis [ axTitle "", axValues (nums [ 0.5 ]), axFormat "%" ]
                    ]
    in
    toVegaLite
        [ width 450
        , height (450 / goldenRatio)
        , cfg []
        , brexitData []
        , enc []
        , bar []
        ]
```


```elm {l v siding highlight=[8,21,22]}
largeBar : Spec
largeBar =
    let 
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])
        enc =  
            encoding
                << position X [ pName "health", pOrdinal, pAxis [] ]
                << position Y [ pName "income", pQuant, pAxis [axTitle "Insert 'Income' Here"] ]
    in
    toVegaLite [ width 640, height (450 / goldenRatio), cfg[], exrcData [], enc [], bar[] ] 
``` 


This looks pretty ugly, with the bar widths out of proportion with the text, so lets shrink the width of the bars and give them a more muted shade, which will also help to give greater relative prominence to that critical 50% threshold and allow us to move the bar labels to within the bar space:



                << position X
                    [ pName "ageGroup"
                    , pScale [ scPaddingInner 0.5 ]
                    , pAxis [ axTitle "" ]
                    ]
                << position Y
                    [ pName "leave"
                    , pQuant
                    , pAxis [ axTitle "", axValues (nums [ 0.5 ]), axFormat "%" ]
                    ]



```elm {l v siding highlight=[8,21,22]}
betterLargeBar : Spec
betterLargeBar =
    let 
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis [ axcoTicks False
                            , axcoDomain False
                            , axcoLabelAngle 45
                            , axcoLabelPadding 0    -- How to apply this to x axis
                            ]
                    )
        enc =  
            encoding
                << position X 
                    [ pName "health"
                    , pOrdinal
                    , pScale [ scPaddingInner 0.5 ]
                    , pAxis []
                    --, pAggregate opMean 
                    ]
                << position Y 
                    [ pName "income"
                    , pQuant 
                    , pAxis [axTitle "Insert 'Income' Here"] ]
    in
    toVegaLite [ width 640, height (450 / goldenRatio), cfg[], exrcData [], enc [], bar[] ] 
``` 

{|task)}


##Lecture 3 - colour

####Steamgraph

```elm {v}
attendance : Spec
attendance =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])

        data =
            dataFromUrl "https://gicentre.github.io/data/attendance.csv"
                -- Force session values to be treated as numbers not text
                [ parse [ ( "session", foNum ) ] ]  -- specify the nature of data to sumber when runiing

        enc =
            encoding
                << position X [ pName "session" ] --pQuant to number
                << position Y
                    [ pName "attendance"
                    , pQuant
                    --, pStack stCenter -- Stacked from the centre not bottom.
                    , pAxis []
                    ]
                << detail [ dName "id" ] -- look in id column and provide symbol/stripes for each student
                << color [ mName "cohort" ]
    in
    toVegaLite
        [ width 600
        , height 300
        , cfg []
        , data
        , enc []
        , area
            [ maLine (lmMarker []) -- Add lines around each area 'stream'
            , maInterpolate miMonotone -- Monotone interpolation gives curved lines
            ]
        ]
```





```elm {v}
steamgraph : Spec
steamgraph =
    let 
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])
        exrcData1 =
                dataFromUrl "https://vega.github.io/vega-lite/data/gapminder-health-income.csv"
                -- Force session values to be treated as numbers not text
                --[ parse [ ( "", foNum ) ] ]  -- specify the nature of data to sumber when runiing
                
        enc =  
            encoding
                << position X 
                    [ pName "health"
                    --, pOrdinal
                    -- pScale [ scPaddingInner 0.9 ]
                    , pAxis []
 
                    ]
                << position Y 
                    [ pName "income"
                    , pQuant 
                    , pAxis [axTitle "Insert 'Income' Here"] ]

                << detail [ dName "country" ] -- look in id column and provide symbol/stripes for each student
                << color [ mName "population", mOrdinal, mScale [ scScheme "reds" [] ]]
    in
    toVegaLite [ width 640, height (450 / goldenRatio)
               , cfg[]
               , exrcData1 []
               , enc []
               ,  area [ maLine (lmMarker []) -- Add lines around each area 'stream'
               , maInterpolate miMonotone -- Monotone interpolation gives curved lines
               
                       ] 
               ] 
``` 
---

### Colours
Here is how you would use the [dark2](https://vega.github.io/vega/docs/schemes/#dark2) colour scheme for encoding nominal (which is the default if not specified) data:



```elm {l v}
catBars1 : Spec
catBars1 =
    let 
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis [ axcoTicks False
                            , axcoDomain False
                            , axcoLabelAngle 45
                            , axcoLabelPadding 0    -- How to apply this to x axis
                            ]
                    )
        enc =  
            encoding
                << position X 
                    [ pName "health",
                      pAxis [ axLabelAngle 0 ]
                    --, pOrdinal
                    --, pScale [ scPaddingInner 0.5 ]
                    , pAxis []
                    --, pAggregate opMean 
                    ]
                << position Y 
                    [ pName "income"
                    --, pQuant 
                    , pAxis [axTitle "Insert 'Income' Here"] 
                    , pAxis[]]
                
                << color [ mName "population", mScale [ scScheme "dark2" [] ] ]
    in
    toVegaLite [ width 640, height (450 / goldenRatio), cfg[], exrcData [], enc [], bar[] ] 
``` 


### Diverging Colour Scales
One important data characteristic that may not be captured simply by distinguishing nominal, ordinal and quantitative data, is whether the data are diverging or not. Diverging colour schemes reflect some difference from a central value that may be either greater or less than that central value. The further a data value is from that central point, the more dominant the colour. This is especially useful when comparing data values with some mean or standard measurement.


For example, suppose we wished to see how different each data item is from zero, we could use a diverging scheme such as [red-blue](https://vega.github.io/vega/docs/schemes/#redblue) (which is also colour blind safe):


```elm {l}
divergingData =
    dataFromColumns []
        << dataColumn "category" (strs [ "A", "B", "C", "D", "E", "F", "G", "H", "I" ])
        << dataColumn "value" (nums [ -28.6, -1.6, -13.6, 34.4, 24.4, -3.6, -57.6, 30.4, -4.6 ])
```

```elm {l v highlight=11}
divergingBars : Spec
divergingBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pQuant ]
                << color
                    [ mName "value"
                    , mQuant
                    , mScale [ scScheme "redblue" [] ]
                    ]
    in
    toVegaLite [ divergingData [], enc [], bar [] ]
```




```elm {l v}
diverging : Spec
diverging =
    let 
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis [ axcoTicks False
                            , axcoDomain False
                            , axcoLabelAngle 45
                            , axcoLabelPadding 0    -- How to apply this to x axis
                            ]
                    )
        enc =  
            encoding
                << position X 
                    [ pName "health",
                      pAxis [ axLabelAngle 0 ]
                    --, pOrdinal
                    --, pScale [ scPaddingInner 0.5 ]
                    , pAxis []
                    --, pAggregate opMean 
                    ]
                << position Y 
                    [ pName "income"
                    , pQuant 
                    , pAxis [axTitle "Insert 'Income' Here"] 
                    , pAxis[]]
                
                << color [ mName "population", mOrdinal, mScale [ scScheme "redblue" [] ] ]
    in
    toVegaLite [ width 640, height (450 / goldenRatio), cfg[], exrcData [], enc [], point[] ] 
``` 

