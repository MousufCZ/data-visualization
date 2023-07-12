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
            dataFromUrl "https://gicentre.github.io/data/euPolls.json"

        enc =
            encoding
                << position Y [ pName "Percent", pTemporal ]
                << position X [ pName "Answer", pOrdinal ]
                << color [mName "Percent", mTemporal]


    in
    toVegaLite [ width 640, data [], enc [], bar [] ]
```


