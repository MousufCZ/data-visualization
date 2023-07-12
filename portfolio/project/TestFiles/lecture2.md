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


## Expenditure data set

```elm {l}
dataExp =
    dataFromUrl "expenditure1.csv"
```



### Bar 1

#### How much does the UK Gov spend in 2021 compared to 2009 and how does it compare to the total annual expenditure in the UK from private and public funding source.

This was my fav so far:

```elm {l=hidden v}
barL2 : Spec
barL2 =    
    let
        enc =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y [ pName "AnnualExpenditure", pOrdinal ]
                << color [ mName "UK Gov", mQuant ]

    in
    toVegaLite [ width 540, dataExp [], enc [], bar [] ]
```

Tufte: 

```elm {l=hidden v}
barTufte1 : Spec
barTufte1 =    
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
        enc =
            encoding
                << position X [ pName "Year", pTemporal, pAxis [] ]
                << position Y [ pName "AnnualExpenditure", pOrdinal, pAxis [], 
                        pScale[ scZero True ] ]
                << color [ mName "UK Gov", mQuant ]

    in
    toVegaLite [ width 540, cfg[], dataExp [], enc [], bar [] ]
```


---


Tufte, Adding label, this is my fav: 

```elm {l=hidden v}
annualUKExpenditure : Spec
annualUKExpenditure =    
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis 
                        [ axcoTicks True
                        , axcoDomain False
                        , axcoLabelAngle 0 
                        , axcoLabelPadding 0 -- 
                        ])
        enc =
            encoding
                << position X [ pName "Year"
                    , pTemporal
                    , pAxis [axTitle "United Kingdom's annual expenditure over the years"]
                    ]
                << position Y [ pName "AnnualExpenditure"
                    , pOrdinal
                    , pAxis [axTitle "£"]
                    , pScale[ scZero True ] 
                    ]
                << color [ mName "UK Gov"
                    , mQuant
                    ]

    in
    toVegaLite [ width 450
                , height (450 / goldenRatio)
                , cfg[]
                , dataExp []
                , enc []
                , bar [] ]
```

I am going to be using this particular visualisation.

---





{(insights|}

---
"annualUKExpenditure" is a simple bar chart that compares the annual expenditure variable relationship with investment made by the UK government from 2009 through to 2020. 

This specific data variable relationship has allowed me to interpret and understand the UK R&D landscape and understand if there is an importance to persue the rest of my researh questions.

As a result of doing so, I can infer that the UK annual expenditure has spent considerably more in 2020 than 2009 and the pattern combined with available online reports suggests that the United Kingdom's research and development landscape is an important issue to focus on. The UK Gov is the largest funding source of the annual expenditure and their recent heavy investment up to 2020 shows the UK aims achieve it's investment in R&D to 2.4% of GDP by 2027.

Having a deeper look to manke an interpretation of the data visualisation we can identify that from 2013 to 2020, there has been a consistant growth every year on annual expenditure from private and public funding sources. However between 2009 and 2011, there is an inconsistant annual expenditure investment patterns as well as UK Gov public investment patterns.

I should mention that there is very little data available before 2009, which points at the lack of importance placed on UK's research and development and the UK Gov has spent very little in the first 3 years. This points at new research and development policy. 

I did a quick research following this finding and discovered "UK's 2010 to 2015 government policy: research and development". With in this policy, there was to be the creation of GrothAccelertor, UK Innoavtion Investment Fund, UK Space Agency, greater funding in National Academies, greater investments made in 7 research councils to invest in R&D, there were:

- Arts and Humanities Research Council (AHRC) 
- Biotechnology and Biological Sciences Research Council (BBSRC) 
- Engineering and Physical Sciences Research Council (EPSRC) 
- Economic and Social Research Council (ESRC) 
- Medical Research Council (MRC) 
- Natural Environment Research Council (NERC) 
- Science and Technology Facilities Council (STFC)

More importantly, there was the creation of University Enterprise Zones and innovation centers which brings together knowledge, skills, technical resources and capital. ([2010 to 2015 government policy: research and development](#references)) This further confirms the requirement of my research, something I would not have been able to verify if not for this visualisation.

The UK Gov found great result from investing in R&D outline in the report of 2015, ([The UK Innovation Survey 2015 Main Report](#references)) and from 2013 and 2015 heavier investment were by the UK Gov on top of the private funding. 

At this point, it is important to explore the UK GDP economic growth to explain the pattern we see above of the UK Gov investment. Although the investment in annual expenditure sees a consistant growth from other founding sources, the UK Gov funding becomes is lower from 2015 to 2018. (See investmentsource)

I believe this maybe due to fact that UK GDP shrunk by 1.6% from 2014 to 2018, this also explain why the alternative funding source's investment grew considerably during this period. ([UK GDP growth](#references))

So far we have just been counting different ways the annual expenditure works however visualizing this alone is not enough. Next we will try to find and show other meaningful patterns in our data sources. ([Jonathan Corum](#references))


{|insights)}

---

{(designJustification|}

---

#### Design choice:

We need to visualise the data of UK Research and development, it's relationship with annual expenditure and investment made by the UK Government in the most simplest format. This is the best way to answer my first research question, "Is United Kingdom's research and development landscape an important issue to focus on?".

I believe I have got this visualisation correct, as it alone answered the requirement of inspiring young academics into research and development sector. 

By making my design choice simple for relationship comparison has allowed me to make the data more natural for the human mind to comprehend and therefore makes it easier to identify specific trends, patterns, and outliers within it. This made the process of asking new questions easy, in addition to this, it allowed me to research questions I otherwise would not have thought to ask. ([Why Data Viz is important][#references])

Whilst coming to these new questions, I did look at the structure and leading ways our brains creates bias, an abstract example is seeing faces in the clouds. I found it very difficult to filter this process however, understanding this exists has allowed me to be aware of it and try to overcome this bias as I progress through my research questions and build experience in the future.


##### Axis

This visualizations is designed to compare variences and I will ensure to make all my charts to have consistent approach to how I use my axes. 

I decided to focus on this as I hope it will allow users to accurately compare the data across each visualization. ***(I.e. The cause is how far the kid sits from the TV - the x variable. The effect is how much eyesight she loses - the y variable.)XXXXXXX***

As a general rule, I will apply the independent" variable on the x-axis and the "dependent" variable on the y-axis and applying the following recommended guidence: 
- Be consistent in how you label tick marks.
- Use consistent axes for data visualizations that are being compared.
- Add the unit of measurement to the axis title (i.e. %, £).
- Reduce clutter by not labeling every tick mark.
- Use different size tick marks (like a ruler) when not labeling each tick mark, i.e. use a longer tick mark for every 5th mark. I am not sure how to do this using code yet.
([Axes guidence](#references))


#### Storytelling

To enhance my approach to data visualisation, I settling on a creative approach to storytelling. As I have not done this before I experimented with the CLUE model which looks at every process involved in deliveringthe current state of narrative. ([Unveiling storytelling and visualization of data](#references)) 

In my experimentation, I dound the data axis were very easy to understand with out any axis title. I took advantage of this by removing the "Year" label and implemented a simple sentence which anotates what is happening.

I found this to be an interactive and an effective approach and I intend on seeing how else I can develop this going forward.

#### Human in the loop
For human understanding, approximations highlight a weakness in our approach: not every question can be answered by analytic means. However, we believe that these approximations are sufficient to aid effective usage. These guidelines could be more closely examined (and made more precise) through greater user study and data gathering.

I would recommend further reading on Research and development and UK's total expenditure as a whole before looking at the visualisation. This is is to give extra knowledge to my target audience and allow them to be able to compare this data.

I also added a question before the visualisation to help guide the audience 

I initially used squares for design choice however I found the bars to be better. This was to help me better determin relationship between the variables. This gives graphically much more.

-- toVegaLite [ width 540, lec1data [], enc [], **square []** ]


#### Does design choice answer the question I am trying to answer.

This simple design choice shows that United Kingdom's research and development landscape is an important issue to focus on. UK Gov is the largest  and decidding to use these design choices made 

{|designJustification)}

---



#### Second research question

##### How can we harness excitement about this vision, and inspire a whole new generation of scientists, researchers, technicians, engineers, and innovators to the correct industry?



The following data visualisations shows the relationsip of Scientists and engineer recruitment through the years.


Tufte suggests that because horizontal text is easier to read than vertical, we should endeavour to layout text horizontally where possible. In our case, Vega-Lite has (sensibly) arranged bar labels vertically as the chart is not wide enough to fit the text any other way.

By enlarging the chart, we can make sufficient room. How wide and tall should we make it? Tufte suggests that creating charts with an aspect ratio (width divided by height) of the golden ratio makes graphics easy to interpret and pleasing to view:

##### Tufte's minimalist design approach using the principle of Visual-Data Correspondence :


```elm {l}
goldenRatio : Float
goldenRatio =
    1.618
```

```elm {l=hidden v}
recruitment1 : Spec
recruitment1 =    
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis [ axcoTicks True, axcoDomain False, axcoLabelAngle 0 ])
        enc =
            encoding
                << position X [ pName "Year"
                    , pTemporal
                    , pAxis [axTitle "R&D Employment"] ]
                << position Y [ pName "Scientists and engineers"
                    , pQuant
                    , pAxis [axTitle "Scientists and engineer"] ]

    in
    toVegaLite [ width 450
                , height (450 / goldenRatio)
                , cfg[]
                , dataExp []
                , enc []
                , bar [] 
                ]
```


I found this data looks better when using line:

```elm {l=hidden v}
recruitment2 : Spec
recruitment2 =    
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
                    , pAxis [axTitle "R&D Employment"] 
                    --, pScale [ scPaddingInner 0.5 ]
                    ]
                << position Y [ pName "Scientists and engineers"
                    , pQuant
                    , pAxis [axTitle "Scientists and engineer"] 
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


This gives a false image of the data:

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


```elm {l=hidden v}
recruitmentmaFill : Spec
recruitmentmaFill =    
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
                , line [ maFillOpacity 0.5, maFill "#aab" ]
                ]
```

{(designJustification|}

---
The objective of the design is to clearly show the employemnt growth of scientist and engineers and build a story from it.

I have designed this in this way for the reason of simplicity based on Tufte's minimalist design approach using the principle of Visual-Data Correspondence design branch. 

As a result of doing so, I now understand this about the ...... DATAxxxx

Therefore it was a success/not as greater proportion of ink is devoted to data and is a better visualisation. By removing the chart junk and keeping the integrity of the data, the minimalist approach allows us to see a more complex message in my story I am trying to tell. Removing any destraction, confusion and room for hullucination.

Considering objectivity, I have discovered xxxx otherwise, I woudn't have made  xxxx if I had not made this design....



For my final design, I have choosen this design, this does the sane thing, just like here and it's simillar/not to the other designs. 
[academic & impartial evidence to support my design choice and also justify data choice] 

"

What are my human vs computation process

{|designJustification)}