---
id: litvis

narrative-schemas:
  - ../../narrative-schemas/project.yml

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

#### Second research question

##### How can we harness excitement about this vision, and inspire a whole new generation of scientists, researchers, technicians, engineers, and innovators to the correct industry? 



The following data visualisations shows the relationsip of Scientists and engineer recruitment through the years.


Tufte suggests that because horizontal text is easier to read than vertical, we should endeavour to layout text horizontally where possible. In our case, Vega-Lite has (sensibly) arranged bar labels vertically as the chart is not wide enough to fit the text any other way.

By enlarging the chart, we can make sufficient room. How wide and tall should we make it? Tufte suggests that creating charts with an aspect ratio (width divided by height) of the golden ratio makes graphics easy to interpret and pleasing to view:

##### Tufte's minimalist design approach using the principle of Visual-Data Correspondence :


## Expenditure data set

```elm {l=hidden}
dataExp =
    dataFromUrl "expenditure1.csv"
```





Colour and shapes

Approximately 1 in 12 men of northern European descent have a red-green colour vision deficiency making it hard to distinguish between many shades of green from shade of red (if you have trouble seeing the number above, or you see a '21', may have a red-green colour deficiency). It is much lower in women (c. 0.5%) and for people of non-European descent.

Beware therefore of designing data visualizations where it is important to be able to distinguish reds from greens in order to uncover patterns in the data. Sites such as [Coblis](https://www.color-blindness.com/coblis-color-blindness-simulator/) can be used to check how your data visualization may be perceived by those with various colour vision deficiencies. There are also various colour palette combinations that are generally less vulnerable to vision deficiencies (as we shall see below).

```elm {l=hidden}
goldenRatio : Float
goldenRatio =
    1.618
```


I like the one below as it breaks down growth in 

```elm {l=hidden v}
recruitmentpBin : Spec
recruitmentpBin =    
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis 
                        [ axcoTicks True
                        , axcoDomain False
                        , axcoLabelAngle 0 
                        , axcoLabelPadding -0 -- 
                        ])
        enc =
            encoding
                << position X [ pName "Year"
                    , pTemporal
                    , pAxis [axTitle "R&D Employment Over The years"] 
                    --, pScale [ scPaddingInner 0.5 ]
                    ]
                << position Y [ pName "Scientists and engineers"
                    , pQuant
                    , pAxis [axTitle "Scientists and engineer"] 
                    , pBin[]
                    ]

    in
    toVegaLite [ width 450
                , height (450 / goldenRatio)
                , cfg[]
                , dataExp []
                , enc []
                , line [ maFillOpacity 0.5 ]
                ]
```




```elm {v l=hidden}
recruitmentArea : Spec
recruitmentArea =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])


        enc =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y
                    [ pName "Scientists and engineers"
                    , pQuant
                    , pStack stCenter -- Stacked from the centre not bottom.
                    , pAxis []
                    ]
                << detail [ dName "AnnualExpenditure" ]
                << color [ mName "UK Gov" ]
    in
    toVegaLite
        [ width 600
        , height 300
        , cfg []
        , dataExp []
        , enc []
        , area
            [ maLine (lmMarker []) -- Add lines around each area 'stream'
            , maInterpolate miMonotone -- Monotone interpolation gives curved lines
            ]
        ]
```


```elm {v l=hidden interactive}
dynamicAnnotation : Spec
dynamicAnnotation =
    let
        ps =
            params
                << param "label"
                    [ paSelect sePoint
                        [ seNearest True, seOn "mouseover", seEncodings [ chX ] ]
                    ]

        enc1 =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y [ pName "Scientists and engineers", pQuant ]
                << color [ mName "AnnualExpenditure", mTitle "" ]

        spec1 =
            asSpec
                [ enc1 []
                , layer
                    [ asSpec [ line [] ]
                    , asSpec [ ps [], enc1_2 [], point [] ]
                    ]
                ]

        enc1_2 =
            encoding
                << opacity [ mCondition (prParamEmpty "label") [ mNum 1 ] [ mNum 0 ] ]

        spec2 =
            asSpec [ trans2 [], layer [ spec2_1, spec2_2 ] ]

        trans2 =
            transform << filter (fiSelectionEmpty "label")

        spec2_1 =
            asSpec [ enc2_1 [], rule [ maColor "gray" ] ]

        enc2_1 =
            encoding << position X [ pName "Year", pTemporal ]

        spec2_2 =
            asSpec [ enc2_2 [], textMark [ maAlign haLeft, maDx 5, maDy -5 ] ]

        enc2_2 =
            encoding
                << position X [ pName "Year", pTemporal, pTitle "" ]
                << position Y [ pName "Technicians and laboratory assistants", pQuant ]
                << text [ tName "AnnualExpenditure", tQuant ]
                << color [ mName "UK Gov" ]
    in
    toVegaLite [ width 540, height 300, dataExp [], layer [ spec1, spec2 ] ]
```


{(designJustification|}

---
The objective of the design is to clearly show the employemnt growth of scientist and engineers and build a story from it.

I have designed this in this way for the reason of simplicity based on Tufte's minimalist design approach using the principle of Visual-Data Correspondence design branch. I heavily focused on the effective use of colour and visual variable to answer my research question.

As a result of doing so, I now understand this about the ...... DATAxxxx

Therefore it was a success/not as greater proportion of ink is devoted to data and is a better visualisation. By removing the chart junk and keeping the integrity of the data, the minimalist approach allows us to see a more complex message in my story I am trying to tell. Removing any destraction, confusion and room for hullucination.

Considering objectivity, I have discovered xxxx otherwise, I woudn't have made  xxxx if I had not made this design....



For my final design, I have choosen this design, this does the sane thing, just like here and it's simillar/not to the other designs. 
[academic & impartial evidence to support my design choice and also justify data choice] 

"

What are my human vs computation process

{|designJustification)}






#### Second research question

##### How can we harness excitement about this vision, and inspire a whole new generation of scientists, researchers, technicians, engineers, and innovators to the correct industry?

The following data visualisations branch starts with understanding which industries are leading the way for research and development.




____
# Design choice 


I took advantage of Tufte's minimalistic suggestion and looking at data-ink ration. Focusing on the dimensions of beauty, clarity, effectiveness, and simplicity. ([Minimalism in Data Visualization: Perceptions of Beauty, Clarity, Effectiveness, and Simplicity](#reference))  I have also I focused on the effective use of colour and visual variable to answer my research question.






By enlarging the chart, we can make sufficient room. How wide and tall should we make it? Tufte suggests that creating charts with an aspect ratio (width divided by height) of the golden ratio makes graphics easy to interpret and pleasing to view:



Through analysing all experimenting, use of less minimalism, striking colour and selecive visual variable, I believe for bar charts and scatterplots a story is better told giving a fuller information. This design slection is contrary to the belief of Edward Tufte.

Despite my current finding, I look forward to experimenting with more advanced data visuation and Edward Tufte's approach.

Having focused on multiple discipline of data visualization with in academic research and industry practice, for my final design, I have choosen to take a humanism approach. Exploring where art this does the same if not a better undertaking of closing the gap between human and data interpretation. I am hoping this will also justify my choice of data.

----

 just like here and it's simillar/not to the other designs. 
[academic & impartial evidence to support my design choice and also justify data choice] 