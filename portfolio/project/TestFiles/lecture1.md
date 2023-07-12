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

# Lecture 1
## Total Expenditure UK 

### Line

This is a start:

```elm {l=hidden v}
lineL1 : Spec
lineL1 =    
    let
        lec1data =
            dataFromUrl "expenditure1.csv"
             

        enc =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y [ pName "AnnualExpenditure", pQuant ]

    in
    toVegaLite [ width 540, lec1data [], enc [], line [] ]
```



### Bar 1

This is too simple: 

```elm {l=hidden v}
barL1 : Spec
barL1 =    
    let
        lec1data =
            dataFromUrl "expenditure1.csv"
             

        enc =
            encoding
                << position Y [ pName "Year", pTemporal ]
                << position X [ pName "AnnualExpenditure", pOrdinal ]
                << color [ mName "UK Gov", mQuant ]

    in
    toVegaLite [ width 540, lec1data [], enc [], bar [] ]
```



### Bar 2

#### How much does the UK Gov spend in 2021 compared to 2009 and how does it compare to the total annual expenditure in the UK from private and public funding source.

This is my fav so far:

```elm {l=hidden v}
barL2 : Spec
barL2 =    
    let
        lec1data =
            dataFromUrl "expenditure1.csv"
             

        enc =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y [ pName "AnnualExpenditure", pOrdinal ]
                << color [ mName "UK Gov", mQuant ]

    in
    toVegaLite [ width 540, lec1data [], enc [], bar [] ]
```


### Other

This is hard to read:

```elm {l=hidden v}
circle1 : Spec
circle1 =    
    let
        lec1data =
            dataFromUrl "expenditure1.csv"
             

        enc =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y [ pName "AnnualExpenditure", pOrdinal ]
                << color [ mName "UK Gov", mQuant ]

    in
    toVegaLite [ width 540, lec1data [], enc [], circle [] ]
```

This is unreadable:

```elm {l=hidden v}
circle2 : Spec
circle2 =    
    let
        lec1data =
            dataFromUrl "expenditure1.csv"
             

        enc =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y [ pName "AnnualExpenditure", pOrdinal ]
                << color [ mName "UK Gov" ]

    in
    toVegaLite [ width 540, lec1data [], enc [], bar [] ]
```

```elm {l=hidden v}
area1 : Spec
area1 =    
    let
        lec1data =
            dataFromUrl "expenditure1.csv"
             

        enc =
            encoding
                << position Y [ pName "Year", pTemporal ]
                << position X [ pName "AnnualExpenditure", pOrdinal ]
                << color [ mName "UK Gov", mQuant ]

    in
    toVegaLite [ width 540, lec1data [], enc [], area [] ]
```


{(insights|}

---
These are simple charts that looks the annual expenditure relationship with UK Gov investment during 2009 to 2020. 

From this comparison, we see that the UK annual expenditure has spent considerably more in 2020 than 2009. 

Making comparison of the data visualisation, we can identify that from 2013 to 2020, there has been a consistant growth every year on annual expenditure from private and public UK Gov funding source. However between 2009 and 2011, there is an inconsistant annual expenditure investment patterns as well as UK Gov public investment patterns.

I should mention that there is very little data available before 2009, which points at the lack of importance placed on UK's research and development and the UK Gov has spent very little in the first 3 years. This points at new research and development policy. 

I did a quick research following this and found UK's 2010 to 2015 government policy: research and development. With in this poicy, there was to be creation of GrothAccelertor, UK Innoavtion Investment Fund, UK Space Agency, greater funding in National Academies, greater investments made in 7 research councils to invest in R&D
- Arts and Humanities Research Council (AHRC) 
- Biotechnology and Biological Sciences Research Council (BBSRC) 
- Engineering and Physical Sciences Research Council (EPSRC) 
- Economic and Social Research Council (ESRC) 
- Medical Research Council (MRC) 
- Natural Environment Research Council (NERC) 
- Science and Technology Facilities Council (STFC)

More importantly, there was the creation of Uniersity Enterprise Zones and innovation centers to bringing together knowledge, skills, technical resources and capital. ([2010 to 2015 government policy: research and development](#references))

The UK Gov found great result from investing in R&D outline in the report of 2015. ([The UK Innovation Survey 2015 Main Report](#references)),  from 2013 and 2015, further and heavier investment were by the UK Gov on top of the private funding. 

At this point, it is important to explore the UK GDP economic growth to explain the pattern of UK Gov investment. Although the investment in annual expenditure sees a consistant growth from other founding sources, UK Gov funding becomes much lower. 

The UK GDP shrunk by 1.6% from 2014 to 2016, this is why I created a simple data visualisation which looks at the UK R&D funding sources. ([UK GDP growth](#references))

We see that the UK Gov has spent considerably more in 2020 than 2009. 

{|insights)}

---

{(designJustification|}

---

Design choice:

The colours show that the UK Gov has increased it's investment heavily over the year to 2020. The change of X and Y axis allows to better compare, making it much more effective.

These approximations highlight a weakness in our approach: not every question can be answered by these analytic means. However, we believe that these approximations are sufficient to aid effective usage. These guidelines could be more closely examined (and made more precise) through a user study.

I would recommend further reading on Research and development and UK's total expenditure as a whole before looking at the visualisation. This is is to give extra knowledge to my target audience and allow them to be able to compare this data.

I also added a question before the visualisation to help guide the audience 

I initially used squares for design choice however I found the bars to be better. This was to help me better determin relationship between the variables. This gives graphically much more.
-- toVegaLite [ width 540, lec1data [], enc [], **square []** ]



{|designJustification)}
