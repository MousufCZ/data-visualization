---
follows: imdbRoot
---

A default histogram:

```elm {l v}
histogram : Spec
histogram =
    let
        enc =
            encoding
                << position X [ pName "IMDB Rating", pBin [] ]
                << position Y [ ],pAggregate opCount ]
		-- << 
                << color [ mName "IMDB Rating", ],mOrdinal ]
    in
    toVegaLite [ data, enc [], bar [] ]
```