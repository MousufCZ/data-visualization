---
elm:
    dependencies:
        gicentre/elm-vegalite: latest
---

```elm {l=hidden}
import VegaLite exposing (..)
```


```elm {l v}
myBarchart : Spec
myBarchart =    
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
                << color [ mName "AvHireTime", mQuant ]


    in
    toVegaLite [ width 640, data [], enc [], bar [] ]
```