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

```elm {l=hidden}
dataTrend : Data
dataTrend =
    dataFromUrl "topIndustry1.csv" []
```

Line:

```elm {l=hidden v}
lineL1 : Spec
lineL1 =    
    let
        enc =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y [ pName "Pharmaceuticals", pQuant ]

    in
    toVegaLite [ width 540, dataTrend, enc [], line [] ]
```

l6-bar:

```elm {l=hidden v}
bar1 : Spec
bar1 =    
    let
        enc =
            encoding
                << position X [ pName "Year", pTemporal ]
                << position Y [ pName "Pharmaceuticals", pOrdinal ]
    in
    toVegaLite
        [ width 640
        , dataTrend
        , enc []
        , bar [ maTooltip ttEncoding ]
        ]
```




```elm {v l}
donut : Spec
donut =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        enc =
            encoding
                << position Theta [ pName "Year", pTemporal ]
                << color [ mName "Pharmaceuticals" ]
    in
    toVegaLite [ cfg [], dataTrend, enc [], arc [ maInnerRadius 50 ] ]
```


```elm {v l}
strip : Spec
strip =
    let
        enc =
            encoding
                << position X [ pName "Pharmaceuticals", pQuant ]
    in
    toVegaLite [ dataTrend, enc [], tick [] ]
```

## Multiple strip plots

```elm {v l}
tickDistributions : Spec
tickDistributions =
    let
        enc =
            encoding
                << position X [ pName "Year", pTemporal, pScale [ scZero False ] ]
                << position Y [ pName "Pharmaceuticals", pOrdinal ]
    in
    toVegaLite [ dataTrend, enc [], tick [] ]
```

---

## Simple scatterplot

Positioning two quantitative variables on orthogonal axes and representing the position with a point mark generates a scatterplot.

```elm {v l}
scatter1 : Spec
scatter1 =
    let
        enc =
            encoding
                << position X [ pName "Year", pTemporal, pScale [ scZero False ] ]
                << position Y [ pName "Pharmaceuticals", pOrdinal, pScale [ scZero False ] ]
    in
    toVegaLite [ dataTrend, enc [], point [] ]
```


---

```elm {v l=hidden}
binnedScatter : Spec
binnedScatter =
    let
        enc =
            encoding
                << position X [ pName "Year", pTemporal, pBin [ biMaxBins 10 ] ]
                << position Y [ pName "Pharmaceuticals", pOrdinal, pBin [ biMaxBins 10 ] ]
                << size [ mAggregate opCount, mQuant ]
    in
    toVegaLite [ dataTrend, enc [], circle [] ]
```


```elm {v l=hidden}
regressionCoefficients : Spec
regressionCoefficients =
    let

        transReg =
            transform
                << regression "Year" "Pharmaceuticals" [ rgMethod rgLinear ]

        transCoef =
            transform
                << regression "Year" "Pharmaceuticals" [ rgMethod rgLinear, rgParams True ]
                << calculateAs "'y = '+format(datum.coef[0],'.2f')+' + '+format(datum.coef[1],'.2f')+'x'" "coef"
                << calculateAs "'R² = '+format(datum.rSquared, '.2f')" "rSq"

        encScatter =
            encoding
                << position X [ pName "Pharmaceuticals", pOrdinal ]
                << position Y [ pName "Year", pTemporal ]

        encCoef =
            encoding
                << text [ tName "coef" ]

        encRSq =
            encoding
                << text [ tName "rSq" ]

        specScatter =
            asSpec [ encScatter [], point [ maFilled True, maOpacity 0.3 ] ]

        specReg =
            asSpec [ transReg [], encScatter [], line [ maColor "firebrick" ] ]

        specCoef =
            asSpec
                [ transCoef []
                , encCoef []
                , textMark [ maX 4, maY 4, maAlign haLeft, maBaseline vaTop, maColor "firebrick" ]
                ]

        specRSq =
            asSpec
                [ transCoef []
                , encRSq []
                , textMark [ maX 4, maY 20, maAlign haLeft, maBaseline vaTop, maColor "firebrick" ]
                ]
    in
    toVegaLite [ width 300, height 300, dataTrend, layer [ specScatter, specReg, specCoef, specRSq ] ]
```




```elm {v l}
scatterWithPolynomial : Spec
scatterWithPolynomial =
    let
        trans =
            transform
                << regression "Year"
                    "Pharmaceuticals"
                    [ rgMethod rgPoly, rgOrder 3 ]

        enc =
            encoding
                << position X [ pName "Pharmaceuticals", pOrdinal ]
                << position Y [ pName "Year", pTemporal ]

        pointSpec =
            asSpec [ point [ maFilled True, maOpacity 0.3 ] ]

        regSpec =
            asSpec [ trans [], line [ maColor "firebrick" ] ]
    in
    toVegaLite [ width 300, height 300, dataTrend, enc [], layer [ pointSpec, regSpec ] ]
```