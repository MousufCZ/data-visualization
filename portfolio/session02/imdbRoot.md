---
id: litvis

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

```elm {l=hidden}
import VegaLite exposing (..)
```

# Internet Movie Database

Here is the data source for creating visualizations based on the Internet Movie Database (IMDB):

```elm {l}
data : Data
data =
    dataFromUrl "https://vega.github.io/vega-lite/data/movies.json" []
```

