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


---
elm:
    dependencies:
        gicentre/elm-vegalite: latest
---



```elm {l v}
myTestChart : Spec
myTestChart =    
    let
        data =
            dataFromUrl "testData.csv"

        enc =
            encoding
                << position X [ pName "days", pNominal ]
                << position Y [ pName "hours", pTemporal ]
                << url [ hStr "favicon.png" ]


    in
    toVegaLite [ width 640, data [], enc [], image [ maWidth 50, maHeight 25 ]  ]
```


```elm {l v}
jitter2 : Spec
jitter2 =
  let
    data =
      dataFromUrl "https://gicentre.github.io/data/titanic/titanic.json"

    trans =
      transform
        << calculateAs "if (datum.sex=='female', random(), 1.5+random())" "jitter"

    enc =
      encoding
        << position X
          [ pName "jitter"
          , pQuant
          , pAxis
            [ axLabelExpr "if(datum.value <1.5,'Female','Male')"
            , axValues (nums [ 0.5, 2 ])
            , axTitle ""
            , axGrid False
            ]
          ]
        << position Y [ pName "age", pQuant ]
        << color [ mName "survived" ]
  in
  toVegaLite [ width 300, height 250, data [], trans [], enc [], circle [] ]
  ```