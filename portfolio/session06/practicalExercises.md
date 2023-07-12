---
id: litvis

narrative-schemas:
  - ../../narrative-schemas/teaching.yml
    - ../narrative-schemas/tdia.yml

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

# Session 6: Practical Exercises

{(task|}

### 2. Direct and Indirect Interaction

Consider direct (interacting with marks) and indirect (interacting via buttons, sliders, drop-down lists etc.) interaction and list the advantages and disadvantages of each by amending / adding to the two tables below. After initially filling the tables, discuss with a others in the Zoom lab session or via Slack, adding any additional advantages/disadvantages that arise from the discussion.

#### 2.1 Direct Interaction

| Advantages                           | Disadvantages                                                         |
| :----------------------------------- | :-------------------------------------------------------------------- |
| Zooming and panning more intuitive   | Difficult to select densely positioned marks                          |
| User able to select specific data    | Extra feature if the data doesn't need interaction                    |
| Easy to make scpecific comparison    | User may come to WRONG comparison through unintended selection (bias) |
| User is in control of the data story | Less credible                                                         |
| Shortens data understanding time     |                                                                       |
| Much more engaging                   |

#### 2.2 Indirect Interaction

| Advantages                                          | Disadvantages                                                          |
| :-------------------------------------------------- | :--------------------------------------------------------------------- |
| Can select non position-encoded data ranges         | It can be less interactive with the user                               |
| Select what we want the viewer to learn from data   | Only reveal patterns and trends directly corrolated with selected data |
| Clear data that we want to highlight (scaling)      | Less                                                                   |
| Use multiple graphs to compare                      |                                                                        |
| Use of various tools to interact with data (sensors |                                                                        |

#### 3.1 Graph

##### Getting the graph to work

Adding `interactive` to any visualization code block header and adding [maTooltip ttEncoding](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#ttEncoding).

(

#### Sharing Litvis documents with wider audience

dataFromUrl "https://github.com/MousufCZ/data-viz-data-repo/blob/main/westMidsCrimesShort.tsv"

)

```elm {l}
crimeIntro : Data
crimeIntro =

    dataFromUrl "https://gicentre.github.io/data/westMidlands/westMidsCrimesShort.tsv" []
```

```elm {l v interactive highlight=13}
crimeIntro1 : Spec
crimeIntro1 =
    let
        enc =
            encoding
                << position X [ pName "month", pTemporal ]
                << position Y [ pName "reportedCrimes", pQuant ]
    in
    toVegaLite
        [ width 640
        , crimeIntro
        , enc []
        , point [ maTooltip ttEncoding ]
        ]
```

##### What is displayed in a tooltip explicitly

```elm {l v interactive highlight=13}
crimeTTDisplay : Spec
crimeTTDisplay =
    let
        enc =
            encoding
                << position X [ pName "month", pTemporal ]
                << position Y [ pName "reportedCrimes", pQuant ]
                                << tooltips
                    [ [ tName "Month", tTemporal, tFormat "%B %Y" ]
                    , [ tName "crimeType", tTitle "Crime Type" ]
                    ]
    in
    toVegaLite
        [ width 640
        , crimeIntro
        , enc []
        , point [ ]
        ]
```

#### Vega-Lite Parameters

##### Non-interactive Vega-Lite parameter

Note in line 17 we set point size with maNumExpr that tells Vega-Lite it should evaluate the circleSize parameter as numeric expression and provide that number to maSize.

```elm {l v interactive highlight=[4-6,15,17]}
parameter : Spec
parameter =
    let
        ps =
            params
                << param "circleSize" [ paValue (num 10) ]

        enc =
            encoding
                << position X [ pName "month", pTemporal ]
                << position Y [ pName "reportedCrimes", pQuant ]
    in
    toVegaLite
        [ crimeIntro
        , ps []
        , enc []
        , point [ maNumExpr "circleSize" maSize ]
        ]
```

```elm {l v interactive highlight=[4-5,15]}
parameterExample2 : Spec
parameterExample2 =
    let
        ps =
            params
                << param "circleSize" [ paValue (num 10), paBind (ipRange [ inMax 50 ]) ]

        enc =
            encoding
                << position X [ pName "month", pTemporal ]
                << position Y [ pName "reportedCrimes", pQuant ]
    in
    toVegaLite
        [ crimeIntro
        , ps []
        , enc []
        , point [ maNumExpr "circleSize" maSize ]
        ]
```

#### Interval selection to be added to a scatterplot

```elm {l v interactive highlight=[4-6,13-16,22]}
cycleHiresInterval : Spec
cycleHiresInterval =
    let
        ps =
            params
                << param "myBrush" [ paSelect seInterval [] ]

        enc =
            encoding
                << position X [ pName "month", pTemporal ]
                << position Y [ pName "reportedCrimes", pQuant ]
                << color
                    [ mCondition (prParamEmpty "myBrush")
                        [ mStr "green" ]
                        [ mStr "lightgrey" ]
                    ]
    in
    toVegaLite
        [ width 400
        , height 400
        , crimeIntro
        , ps []
        , enc []
        , point []
        ]
```

#### Crimedata2

##### Line Interaction

```elm {l highlight=[21-24,26-29]}
crimeData =
    dataFromUrl "https://gicentre.github.io/data/westMidlands/westMidsCrimesAggregated.tsv" []


crimeColours =
    categoricalDomainMap
        [ ( "Anti-social behaviour", "rgb(59,118,175)" )
        , ( "Burglary", "rgb(81,157,62)" )
        , ( "Criminal damage and arson", "rgb(141,106,184)" )
        , ( "Drugs", "rgb(239,133,55)" )
        , ( "Robbery", "rgb(132,88,78)" )
        , ( "Vehicle crime", "rgb(213,126,190)" )
        ]


encHighlight =
    encoding
        << position X [ pName "month", pTemporal, pAxis [ axTitle "", axGrid False ] ]
        << position Y [ pName "reportedCrimes", pQuant, pAxis [ axGrid False ] ]
        << color
            [ mCondition (prParamEmpty "mySelection")
                [ mName "crimeType", mScale crimeColours ]
                [ mStr "black" ]
            ]
        << opacity
            [ mCondition (prParamEmpty "mySelection")
                [ mNum 1 ]
                [ mNum 0.1 ]
            ]
```

```elm {l v interactive }
lineSelection : Spec
lineSelection =
    let
        ps =
            params
                << param "mySelection" [ paSelect sePoint [] ]
    in
    toVegaLite
        [ width 540
        , crimeData -- Data specified in code block above
        , ps [] -- The selection parameters
        , encHighlight [] -- Encoding specified in code block above
        , line [ maInterpolate miMonotone ]
        ]
```

##### Point selection individually

```
elm {l v interactive}
individualPointSelection : Spec
individualPointSelection =
    let
        ps =
            params
                << param "mySelection" [ paSelect sePoint [] ]
    in
    toVegaLite
        [ width 540
        , crimeData -- Data specified in separate code block
        , ps []
        , encHighlight [] -- Encoding specified in separate code block
        , circle []
        ]
```

{|task)}

# Example of Shneiderman's 7 Task-Domain Information Actions

{(overview|}

{|overview)}

{(zoom|}

{|zoom)}

{(filter|}

{|filter)}

{(dod|}

{|dod)}

{(relate|}

{|relate)}

{(history|}

While it could be useful to provide a history / undo facility, providing a literate visualization document does allow the user to annotate the document with their own observations that can act as an _aide-mémoire_ for interaction choices. Currently there is no technical facility to allow interaction choices to be logged automatically.

{|history)}

{(extract|}
There is the possibility to extract data subsets using the [Tidy data package](https://package.elm-lang.org/packages/gicentre/tidy/latest/). Currently there is no mechanism in Litvis for auto-extracting datasets based on user interaction.
{|extract)}
