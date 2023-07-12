---
id: litvis

follows: democracyData
---

@import "../css/datavis.less"

# Mapping Global Democracy Data

```elm {l}
data =
    dataFromColumns []
        << dataColumn "dem2010" (numColumn "2010" democracyGeoTable |> nums)
        << dataColumn "dem2011" (numColumn "2011" democracyGeoTable |> nums)
        << dataColumn "lon" (numColumn "longitude" democracyGeoTable |> nums)
        << dataColumn "lat" (numColumn "latitude" democracyGeoTable |> nums)
        << dataColumn "CountryName" (strColumn "CountryName" democracyGeoTable |> strs)
```

Let's just first look at the non-geographical spread of democracy index values:

```elm {l}
histogram : String -> Spec
histogram year =
    let
        enc =
            encoding
                << position X [ pName year, pBin [ biMaxBins 21 ] ]
                << position Y [ pAggregate opCount ]
                << color [ mName year, mQuant, mScale [ scScheme "blueorange" [] ] ]
    in
    toVegaLite [ width 400, data [], enc [], bar [] ]
```

^^^elm {v=(histogram "dem2010")}^^^
^^^elm {v=(histogram "dem2011")}^^^

We can provide a simple map of the data by plotting circles at their country location, colouring them by their democracy score.

```elm {l}
circleMap : String -> Spec
circleMap year =
    let
        enc =
            encoding
                << position Latitude [ pName "lat" ]
                << position Longitude [ pName "lon" ]
                << color [ mName year, mQuant, mScale [ scScheme "blueorange" [] ] ]
    in
    toVegaLite [ width 700, height 300, data [], enc [], circle [ maSize 180 ] ]
```

^^^elm {v=(circleMap "dem2010") interactive}^^^
^^^elm {v=(circleMap "dem2011") interactive}^^^

It is hard to see the differences between the two years, so let's calculate the change in score directly and plot that.

```elm {l}
trans =
    transform
        << calculateAs "datum.dem2011 - datum.dem2010" "diff"
```

We can see the distribution of changes between 2010 and 2011 by viewing the histogram. Because the vast majority of countries have not changed democracy score, we use a square root scaling of frequencies to emphasise change categories that have fewer countries within them:

```elm {l v}
changeHistogram : Spec
changeHistogram =
    let
        enc =
            encoding
                << position X [ pName "diff", pBin [] ]
                << position Y [ pAggregate opCount, pScale [ scType scSqrt ] ]
                << color
                    [ mName "diff"
                    , mQuant
                    , mScale [ scScheme "pinkyellowgreen" [], scDomain (doMid 0) ]
                    ]
    in
    toVegaLite [ width 400, data [], trans [], enc [], bar [] ]
```

And we can map the changes like we did for Bad Teeth, using a diverging colour scheme. The design is not ideal (hard to use colour to determine small changes in score), but it provides a basis for further design modifications.

```elm {l v interactive}
changeMap : Spec
changeMap =
    let
        enc =
            encoding
                << position Latitude [ pName "lat" ]
                << position Longitude [ pName "lon" ]
                << color
                    [ mName "diff"
                    , mQuant
                    , mScale [ scScheme "pinkyellowgreen" [], scDomain (doMid 0) ]
                    ]
                << tooltips
                    [ [ tName "CountryName" ]
                    , [ tName "diff" ]
                    ]
    in
    toVegaLite [ width 700, height 300, data [], trans [], enc [], circle [ maSize 180 ] ]
```