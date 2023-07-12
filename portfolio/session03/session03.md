---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 3: Representing Data with Colour

## Table of Contents

1.  [Introduction](#1-introduction)
2.  [Describing and Perceiving Colour](#2-describing-and-perceiving-colour)
3.  [Encoding Data with Colour](#3-encoding-data-with-colour)
4.  [Conclusions](#4-conclusions)
5.  [Recommending Reading](#5-recommended-reading)
6.  [Practical Exercises](#6-practical-exercises)

{(aim|}

This session is designed to show how colour may be incorporated into the data visualization process. It introduces some of the important colour models and palettes and their representation in litvis/Vega-Lite. Guidance is given on what colour models and palettes are most appropriate for given data types.

By the end of this session you should be able to:

- Specify colours explicitly using decimal and hex RGB triplets, HSL triplets, and named colours.
- Consider variations in perception of colour when designing your own visualizations, including accommodating those with colour vision deficiencies.
- Match data measurement type to an appropriate colour scheme.
- Avoid common colour design problems such as rainbow and red-green colour schemes.

{|aim)}

---

## 1. Introduction

We use colour in the representation of information so ubiquitously that it is easy to forget the design decisions required to use it effectively. Colour is a complex topic that would merit an entire module by itself. This session will introduce some of the ways we can describe colour, how it may be most effectively associated with different data types and how to specify data to colour encodings with litvis / Vega-Lite.

Before we look in detail at how we describe and use colour, let’s consider an example. [Baby Name Voyager](http://www.babynamewizard.com/voyager) allows you to explore the popularity of names given to babies in the US over the last 140 years.

![Baby name voyager example](https://staff.city.ac.uk/~jwo/datavis2021b/session03/images/babyNames.png?q=0)

{(task|} Explore the baby name voyager and see if you can find any interesting patterns. In what ways has colour been used to make finding these patterns easier? Would you use colour in any different ways if you were to create a similar baby-name explorer? {|task)}

{(annotation|} Add your answers here. {|annotation)}

### An Aside: Streamgraphs

The Baby Name Voyager is a simplified example of a 'streamgraph' - a technique proposed by [Lee Byron and Martin Wattenberg in 2008](http://leebyron.com/streamgraph/stackedgraphs_byron_wattenberg.pdf). It is a variation of a stacked bar chart that tries to maintain continuity of the 'bars'. While that continuity helps to trace subgroups across the chart, it does suffer from the problem of different vertical orderings having a significant effect on the appearance of the chart as the 'shape' of each stream is in part determined by the shapes of the other streams in the stack.

Creating streamgraphs with Vega-Lite / litvis is quite straightforward: Simply use the [area](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#area) mark to show a variable faceted by some categorical variable (in the baby name voyager, the variable is name frequency, faceted by name and shown over time). Here is an example based on module attendance from a previous year where this module was attended by undergraduate and postgraduate students:

```elm {v l}
attendance : Spec
attendance =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])

        data =
            dataFromUrl "https://gicentre.github.io/data/attendance.csv"
                -- Force session values to be treated as numbers not text
                [ parse [ ( "session", foNum ) ] ]

        enc =
            encoding
                << position X [ pName "session" ]
                << position Y
                    [ pName "attendance"
                    , pQuant
                    , pStack stCenter -- Stacked from the centre not bottom.
                    , pAxis []
                    ]
                << detail [ dName "id" ]
                << color [ mName "cohort" ]
    in
    toVegaLite
        [ width 600
        , height 300
        , cfg []
        , data
        , enc []
        , area
            [ maLine (lmMarker []) -- Add lines around each area 'stream'
            , maInterpolate miMonotone -- Monotone interpolation gives curved lines
            ]
        ]
```

## 2. Describing and Perceiving Colour

If we wish to control the way in which we use colour in our data visualizations we need a precise way to describe the colours we use. While from a computational point of view, this is necessary, there is some interesting research that suggests that [colour perception may be linked to gender](http://www.datapointed.net/2010/09/men-women-color-names/) as well as the language we use to describe colour. For commentary on this phenomenon, see the [Eager Eyes blog](https://eagereyes.org/blog/2011/you-only-see-colors-you-can-name) by Robert Kosara and this article by [Caldwell-Harris (2019)](https://www.scientificamerican.com/article/our-language-affects-what-we-see)

[![Colour vision in babies](https://staff.city.ac.uk/~jwo/datavis2021b/session03/images/baby.jpg)](https://staff.city.ac.uk/~jwo/datavis2021b/session03/images/Colour-Do-you-see-what-I-see-The-Himba-tribe.mov)
_Source: [BBC Horizon "Do you see what I see" (2011)](https://www.bbc.co.uk/news/science-environment-14421303)_

[.](https://staff.city.ac.uk/~jwo/datavis2021b/session03/sketches/rainbow/javascript/index.html)

Rather than subjective terms like 'dark red', 'pale orange' etc., in most computing environments where specifying colours is required, it is possible to describe colours with a set of RGB (red, green, blue) numbers. Scaling each number between 0-255 in decimal or `#00` to `#ff` in hexadecimal, we can describe every colour the computer is a capable of generating, known as its [colour gamut](https://en.wikipedia.org/wiki/Gamut). The RGB system is common in computing because it maps well onto the way in which display screens generate coloured images.

Litvis allows colours to be specified this way by using the common CSS (cascading style sheet) convention of [rgb(XXX,XXX,XXX)](https://www.w3schools.com/colors/colors_rgb.asp) for a decimal representation or [#XXXXXX](https://www.w3schools.com/colors/colors_hexadecimal.asp) for compact hexadecimal representation, or by simply naming the colour from a [set of named colours](https://www.w3schools.com/colors/colors_names.asp).

Below is a simple specification to display a coloured dot.

```elm {l v highlight=13}
colouredCircle : Spec
colouredCircle =
    let
        data =
            dataFromColumns []
                << dataColumn "value" (nums [ 1 ])
    in
    toVegaLite
        [ data []
        , circle
            [ maOpacity 1
            , maSize 9000
            , maColor "rgb(200,50,20)"
            ]
        ]
```

{(task|}Try changing the values in brackets after `rgb` to see the effect on the coloured circle. Try changing the colour specification to use hex values instead of decimal (e.g. `maColor "#dc3214"`). Finally, try with a named colour from [this list](https://www.w3schools.com/colors/colors_names.asp) and check it works. {|task)}

One of the problems with the RGB system is that it may not be apparent without some practice, which numbers specify which colours. For example, while it may be obvious that to get a bright red colour we might use `rgb(255,0,0)` in decimal or `#ff0000` in hex, what triplet is required for a dark brown or a pastel mauve? One way of conceiving of all possible reproducible colours is to think of a _colour space_. So for example, if we construct 3 chart axes at right angles representing the degree of red, green and blue components of a colour, we can plot an RGB colour cube:

![RGB cube](https://staff.city.ac.uk/~jwo/datavis2021b/session03/images/rgbCube.png)

This gives us some sense of how the three components can be combined to give us the colour we want (e.g. the cube shows that yellow is made up of high red and green components but a low blue component). Unfortunately many potential colours are not visible in a single view of the cube as they appear 'round the back' (e.g. browns), or 'in the middle' (e.g. greys). And worse still, the colours are not evenly spaced around the cube in terms of our ability to detect and discriminate between them. So while conceptually the RGB space has some simplicity, it does not map well to the way we conceive or perceive colour.

{(task|}Use this [online colour selector](https://www.w3schools.com/colors/colors_picker.asp) to find the RGB values of a dark brown and a pastel mauve. What colour corresponds to the RGB triplet (104,38,100)? {|task)}

{(infobox|} Brown is an unusual colour. When described using RGB values it is 'dark yellow'. [Ware (2021, p.117)](#references) points out that unlike red, blue or green, to identify brown in a scene, we also need a 'reference white' for it to be perceived. He suggests that there is no such thing as a brown light in a darkened room (in contrast to, for example, a red, orange or green light). The idea that one colour can only exist with reference to another is an idea we will consider later in this session.{|infobox)}

### Perceptual Colour Spaces

We can make the specification of colours easier by using alternative spaces that map more closely to the way we perceive colour. One approximation is [HSL space](https://www.w3schools.com/colors/colors_hsl.asp) (Hue - Saturation - Lightness)

![HSL cylinder](https://staff.city.ac.uk/~jwo/datavis2021b/session03/images/hslCylinder.jpg)

**Hue** corresponds to what we might loosely describe as "colour" in everyday language. It is what distinguishes blue from red, or purple from orange. Hue is usually represented as an angle between 0 and 360 degrees recognising that the sequence of colours 'wrap round' (the reds at the left of the hue strip in the selector are similar to the reds at the right).

**Saturation** (closely related to chroma) describes how vivid a colour appears. It is typically expressed as a percentage with low values giving a greyer washed out appearance and higher values giving richer more vivid colours.

**Lightness** (closely related to brightness, value or intensity) is as we might expect, what distinguishes dark from light shades and is typically expressed as a value between 0% (dark) and 100% (light).

{(task|}Try changing the `maColor` line in the `colouredCircle` specification above to use different HSL colours. For example `maColor "hsl(185, 70%, 40%)"` should yield an aqua-marine colour. If you have a particular colour in mind, which do you find easier, finding its RGB value or HSL value?{|task)}

{(infobox|}

When choosing colours for _categorical_ data (that is `nominal` or `ordinal` data that are in discrete categories rather than continuous measurements), HSL (or the related [HCL](https://en.wikipedia.org/wiki/HCL_color_space) colour space), can be useful for choosing contrasting (but not too contrasting) colours that reflect the data properties. [This blog](https://blog.datawrapper.de/beautifulcolors) by Lisa Charlotte Rost provides a very useful set of guidelines.

{|infobox)}

There are other colour spaces that more closely map to how we perceive colour, notably the [CIELab](https://en.wikipedia.org/wiki/CIELAB_color_space) and [CIELuv](https://en.wikipedia.org/wiki/CIELUV) spaces where distance within the space attempts to be directly proportional to perceptual similarity. While direct specification in these colour spaces is not possible in CSS or litvis, they have been used to construct a number of perceptual colour schemes that we will consider below.

### 2.1 Perceiving Colour

As we have the ability to specify colours as RGB or HSL triplets, can we can now fully control the way our data are mapped to colours in our visualizations? Well, not quite, because as we shall see, colour _generation_ does not always correspond to colour _perception_. To give you an idea of the role of colour perception, consider the chart below: Two alternating lines are overlaid on a bar chart whose first two values are 0 and the second two are 1. The colour of the lines is specified with the triplet `rgb(104,38,100)` throughout, yet its colour appears to change depending on the background colour. This apparent change is known as _simultaneous colour contrast_.

```elm {v}
colourContrast : Spec
colourContrast =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        lineData =
            dataFromColumns []
                << dataColumn "x" (nums [ 0, 1, 2, 3, 4, 0, 1, 2, 3, 4 ])
                << dataColumn "y" (nums [ 0, 1, 0, 1, 0, 1, 0, 1, 0, 1 ])
                << dataColumn "id" (nums [ 1, 1, 1, 1, 1, 2, 2, 2, 2, 2 ])

        bgData =
            dataFromColumns []
                << dataColumn "x" (nums [ 0, 1, 2, 3 ])
                << dataColumn "y" (nums [ 0, 0, 1, 1 ])

        bgColours =
            categoricalDomainMap
                [ ( "0", "rgb(255,255,255)" )
                , ( "1", "rgb(255,255,255)" )
                , ( "2", "rgb(150,0,0)" )
                , ( "3", "rgb(0,0,100)" )
                ]

        bgEnc =
            encoding
                << position X [ pName "x", pAxis [] ]
                << position Y [ pName "y", pQuant, pAxis [] ]
                << color [ mName "x", mScale bgColours, mLegend [] ]

        bgSpec =
            asSpec [ bgData [], bgEnc [], bar [] ]

        lineEnc =
            encoding
                << position X [ pName "x", pQuant, pAxis [] ]
                << position Y [ pName "y", pQuant ]
                << detail [ dName "id" ]

        lineSpec =
            asSpec [ lineData [], lineEnc [], line [ maStroke "rgb(104, 38, 100)", maStrokeWidth 6 ] ]
    in
    toVegaLite
        [ cfg [], width 600, height 200, layer [ bgSpec, lineSpec ] ]
```

Colour contrast is an important aspect to consider when constructing data visualizations. Colours of items are not perceived in isolation, but in comparison with surrounding colours. You need to consider the wider context when thinking about how to incorporate colour into your data representation. We will see further examples of this in the classroom lecture.

### Colour Blindness

We have already seen how language (and therefore culture) may influence different people perceive colours. Physiology also plays a role, especially when associated with various types of colour vision deficiencies.

{(task|} Can you see a number in the image below? If so, what is it? {|task)}

![Ishihara test](https://staff.city.ac.uk/~jwo/datavis2021b/session03/images/ishihara9.jpg)
[_Source_](https://en.wikipedia.org/wiki/Color_blindness)

Approximately 1 in 12 men of northern European descent have a red-green colour vision deficiency making it hard to distinguish between many shades of green from shade of red (if you have trouble seeing the number above, or you see a '21', may have a red-green colour deficiency). It is much lower in women (c. 0.5%) and for people of non-European descent.

Beware therefore of designing data visualizations where it is important to be able to distinguish reds from greens in order to uncover patterns in the data. Sites such as [Coblis](https://www.color-blindness.com/coblis-color-blindness-simulator/) can be used to check how your data visualization may be perceived by those with various colour vision deficiencies. There are also various colour palette combinations that are generally less vulnerable to vision deficiencies (as we shall see below).

## 3. Encoding Data with Colour

In many, if not all cases, a visualization specification in litvis / Vega-Lite will not need to specify colours individually. Instead, by default, Vega-Lite will allocate a colour palette according to the type of data you specify should be encoded with colour. In particular whether the data _measurement type_ is _nominal_, _ordinal_, _quantitative_ or _temporal_.

The underlying principle that governs the default allocation is that **the visual properties of a data visualization should reflect the properties of the data you are depicting.** You can think of this as an example of following the principle of _visual-data correspondence_ discussed last week when we considered visualization algebra.

So if the data are in discrete categories that have no order to them (_nominal_), the colours allocated to each category should be discrete, distinguishable and with no obvious order to them. If the data are in discrete categories, but have some natural order (_ordinal_) the colours should reflect that order and categorisation. If the data are continuous, ordered measurements or counts (_quantitative_), the colour allocation should reflect that ordered continuity.

To explore this, imagine we wish to show a set of numbers

```txt
28, 55, 43, 91, 81, 53, 19, 87, 52
```

which we can, for convenience, declare in its own function (note also that we are declaring the data values programmatically rather than loading from a URL using the functions [dataFromColumns](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataFromColumns) and [dataColumn](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataColumn) ):

```elm {l}
exampleData =
    dataFromColumns []
        << dataColumn "category" (strs [ "A", "B", "C", "D", "E", "F", "G", "H", "I" ])
        << dataColumn "value" (nums [ 28, 55, 43, 91, 81, 53, 19, 87, 52 ])
```

Without context, those numbers could be nominal (e.g. personnel IDs); ordinal (e.g. European cities ranked by standard of living – 19th, 28th, 43rd etc.); or quantitative (e.g. daily pay in Euros). Good datavis design would attempt to reflect those qualities, including through their colour encoding. Here for example is a simple chart that uses the default colour scheme for nominal data:

```elm {v l highlight=[7,8]}
colouredBars : Spec
colouredBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pNominal, pTitle "categorical value" ]
                << color [ mName "value", mNominal ]
    in
    toVegaLite [ exampleData [], enc [], bar [] ]
```

{(task|} Observe the effect of changing the measurement type in the `position Y` and `color` lines above to `Ordinal`, `Quant` and back to `Nominal`. How does the colour allocation and legend display reflect the type of data? Are there any problems with the choice of default colours? {|task)}

In many cases you may wish to reflect the characteristics of the data in a similar way to the default colour schemes, but actually choose a different set of colours. Vega-Lite (and Vega) offers a [wide range of built-in colour schemes](https://vega.github.io/vega/docs/schemes/) to choose from. To use a non-default colour scheme we need to specify a [scale](https://vega.github.io/vega-lite/docs/scale.html) – that is, a rule that determines how data values themselves should be translated into colours. This is achieved with the mark scale [mScale](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mScale) function. The name you provide to [scScheme](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scScheme) should correspond to one of the [built-in colour scheme names](https://vega.github.io/vega/docs/schemes/).

Here, for example is how you can create a series of 'reds' rather than the default 'blues' scheme for quantitative (`mQuant`) data:

```elm {l v highlight=11}
redBars : Spec
redBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pQuant ]
                << color
                    [ mName "value"
                    , mQuant
                    , mScale [ scScheme "reds" [] ]
                    ]
    in
    toVegaLite [ exampleData [], enc [], bar [] ]
```

Here is how you would use the [dark2](https://vega.github.io/vega/docs/schemes/#dark2) colour scheme for encoding nominal (which is the default if not specified) data:

```elm {l v}
catBars : Spec
catBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pTitle "categorical value" ]
                << color [ mName "value", mScale [ scScheme "dark2" [] ] ]
    in
    toVegaLite [ exampleData [], enc [], bar [] ]
```

Many of these colour schemes have been developed to take into account perceptual issues such as perceptual uniformity, colour contrast, colour blindness etc. so it is worth considering using them in your own designs. Here for example is the sequential 9-category 'yellow-orange-red' scheme developed as part of the [color-brewer](http://colorbrewer2.org/#type=sequential&scheme=YlOrRd&n=9) system that attempts to make uniform perceptual changes between adjacent categories (see legend):

```elm {l v}
orangeBars : Spec
orangeBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pTitle "categorical value" ]
                << color [ mName "value", mScale [ scScheme "yelloworangered" [] ] ]
    in
    toVegaLite [ exampleData [], enc [], bar [] ]
```

### Diverging Colour Scales

One important data characteristic that may not be captured simply by distinguishing nominal, ordinal and quantitative data, is whether the data are diverging or not. Diverging colour schemes reflect some difference from a central value that may be either greater or less than that central value. The further a data value is from that central point, the more dominant the colour. This is especially useful when comparing data values with some mean or standard measurement.

For example, suppose we wished to see how different each data item is from zero, we could use a diverging scheme such as [red-blue](https://vega.github.io/vega/docs/schemes/#redblue) (which is also colour blind safe):

```elm {l}
divergingData =
    dataFromColumns []
        << dataColumn "category" (strs [ "A", "B", "C", "D", "E", "F", "G", "H", "I" ])
        << dataColumn "value" (nums [ -28.6, -1.6, -13.6, 34.4, 24.4, -3.6, -57.6, 30.4, -4.6 ])
```

```elm {l v highlight=11}
divergingBars : Spec
divergingBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pQuant ]
                << color
                    [ mName "value"
                    , mQuant
                    , mScale [ scScheme "redblue" [] ]
                    ]
    in
    toVegaLite [ divergingData [], enc [], bar [] ]
```

Note that the central 'white' value isn't quite located at 0 (closer to -10 in the legend). This is because the minimum and maximum values are not the same distance from 0 but the scheme allocation will attempt to use the full range of colours stretched between the minimum and maximum value. We can correct this by setting midpoint of the _domain_ (that is, the data being encoded) to 0 with [doMid](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#doMid). Note how after modification, all upward pointing bars are blue and downward bars are red):

```elm {l v highlight=[13]}
divergingSymBars : Spec
divergingSymBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pQuant ]
                << color
                    [ mName "value"
                    , mQuant
                    , mScale
                        [ scScheme "redblue" []
                        , scDomain (doMid 0)
                        ]
                    ]
    in
    toVegaLite [ divergingData [], enc [], bar [] ]
```

### Custom Colour Schemes

Occasionally you may wish to use a colour scheme that is not included in the [Vega colour scheme set](https://vega.github.io/vega/docs/schemes/). This can be achieved by explicitly specifying the colours you wish to use using named colours, RGB or HSL triplets. For continuous schemes with two end-point colours, you can provide a [domainRangeMap](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#domainRangeMap) that specifies the colour associated with the minimum data (domain) value and another corresponding to the maximum (domain) value. Intermediate data values are interpolated between these two extremes.

For example, here is a custom colour mapping between an upper and low limit (note how we set these two limits to be +- 57.6 so it is symmetrical about zero):

```elm {l v highlight=12-14}
customQuantBars : Spec
customQuantBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pQuant ]
                << color
                    [ mName "value"
                    , mQuant
                    , mScale
                        (domainRangeMap
                            ( -57.6, "hsl(220,80%,50%)" )
                            ( 57.6, "hsl(40,80%,50%)" )
                        )
                    ]
    in
    toVegaLite [ divergingData [], enc [], bar [] ]
```

Finally, if the data are categorical we can use the equivalent function [categoricalDomainMap](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#categoricalDomainMap) to provide the mapping from each discrete data value to the colour you wish to encode it with:

```elm {l v highlight=11}
customBars : Spec
customBars =
    let
        enc =
            encoding
                << position X [ pName "category", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "value", pTitle "categorical value" ]
                << color
                    [ mName "value"
                    , mScale
                        (categoricalDomainMap
                            [ ( "28", "hsl(0,70%,50%)" )
                            , ( "55", "hsl(30,70%,50%)" )
                            , ( "43", "hsl(60,70%,50%)" )
                            , ( "91", "hsl(90,70%,50%)" )
                            , ( "81", "hsl(170,70%,50%)" )
                            , ( "53", "hsl(210,70%,50%)" )
                            , ( "19", "hsl(240,70%,50%)" )
                            , ( "87", "hsl(270,70%,50%)" )
                            , ( "52", "hsl(300,70%,50%)" )
                            ]
                        )
                    ]
    in
    toVegaLite [ exampleData [], enc [], bar [] ]
```

{(infobox|}

## Resources for Generating Colour Schemes

Vega-Lite's built-in [named colour schemes](https://vega.github.io/vega/docs/schemes/#accent) provide a good range of possibilities that suit many designs. But there also many other sources for generating colours and providing inspiration for your own schemes. Here are a few to get you started:

- [Fabio Crameri's Scientific Colour Maps](http://www.fabiocrameri.ch/colourmaps.php) for perceptually scaled and colour blind-safe schemes.
- [i want hue](https://medialab.github.io/iwanthue/) for generating categorical colour schemes including colour blind-safe schemes.
- [Leonardo](https://medium.com/@NateBaldwin/leonardo-an-open-source-contrast-based-color-generator-92d61b6521d2) for generating higher contrast colour schemes within a fixed range.
- [Colormind](http://colormind.io): Palettes generated via AI trained on images/movies.
- [Adobe Color Chooser](https://color.adobe.com/create) for exploring colour schemes in various colour spaces.
- [Color Oracle](https://colororacle.org) Simulate various forms of colour vision deficiencies for any image.
- [DataWrapper Categorical Colour Guide](https://blog.datawrapper.de/beautifulcolors) Good advice on selecting appropriate categorical (nominal) colours.

If you have any favourite colour-generating sites, please do share them on [Slack](https://dataviscity.slack.com).

{|infobox)}

### Direct Colour Encoding

The examples of colour encoding above can be considered _indirect_ in that we specify either the type of data we have and let Vega-Lite choose a colour mapping for us, or we specify some colour rules that Vega-Lite then applies to the data it encodes.

Alternatively, we can use _direct_ colouring if our dataset contains the colour specifications as data items. For example, suppose we have a dataset of the percentage of [surveyed respondents' favourite colours](https://www.hotdesign.com/marketing/whats-your-favorite-color):

```elm {l v highlight=[6-7,18]}
favouriteColourExample : Spec
favouriteColourExample =
    let
        data =
            dataFromColumns []
                << dataColumn "favouriteColour"
                    (strs [ "blue", "green", "purple", "red", "orange", "black", "yellow", "grey" ])
                << dataColumn "percent" (nums [ 27, 18, 13, 13, 7.4, 7.3, 5.5, 3.3 ])

        enc =
            encoding
                << position X
                    [ pName "favouriteColour"
                    , pSort [ soByChannel chY, soDescending ]
                    , pAxis []
                    ]
                << position Y [ pName "percent", pQuant ]
                << color [ mName "favouriteColour", mScale [] ]
    in
    toVegaLite [ title "Favourite colours" [], width 200, data [], enc [], bar [] ]
```

By setting `mScale` to be an empty list, we are specifying that there is to be no scaling of data value to colour value. In other words, we want the data value (in the `favouriteColour`column) to be used directly to specify the colour. This is possible because in this example, the data items themselves are valid CSS colour values (`blue`, `green`, `purple` etc.). Note also that we sort the position of columns using [pSort](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pSort) specifying that the order should be determined by the y-channel (percent in this case) descending from highest to lowest.

#### Generating a colour data field (advanced)

If our data do not include direct colour values but we have reason to map colour directly, we can generate new data fields that store colour values. This uses techniques we will consider later in the module (lookup transforms and data calculations), but we consider them briefly here.

Suppose we wished to use colour to show votes for one of two political parties parties (the red party `A` and the blue party `B`). And suppose additionally we would like to use colour lightness to show how much party support there is for each party (so higher numbers of votes result in a darker shade of either red for party A or blue for party B). In other words, we combine two data fields (party and percent) to create a single colour using one field to determine colour hue and the other to determine colour lightness.

We do this by creating a data [transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#transform) that calculates the maximum % vote in any region and then specifies an HSL data value for each region based on its associated party and vote values.

```elm {l v highlight=[10-19]}
colourCombineExample : Spec
colourCombineExample =
    let
        data =
            dataFromColumns []
                << dataColumn "region" (strs [ "region1", "region2", "region3", "region4" ])
                << dataColumn "party" (strs [ "A", "B", "B", "A" ])
                << dataColumn "vote" (nums [ 28, 75, 21, 48 ])

        colourLookup =
            dataFromColumns []
                << dataColumn "category" (strs [ "A", "B" ])
                << dataColumn "hue" (strs [ "0", "215" ])

        trans =
            transform
                << joinAggregate [ opAs opMax "vote" "maxVote" ] []
                << lookup "party" (colourLookup []) "category" (luFields [ "hue" ])
                << calculateAs "'hsl('+datum.hue+',100%,'+(100 - 70*datum.vote/datum.maxVote)+'%)'" "colour"

        enc =
            encoding
                << position X [ pName "region", pAxis [ axTitle "", axLabelAngle 0 ] ]
                << position Y [ pName "vote", pQuant, pTitle "% vote of winning party" ]
                << color [ mName "colour", mScale [], mLegend [] ]
    in
    toVegaLite [ width 200, height 130, data [], trans [], enc [], bar [] ]
```

Don't worry if you don't understand the code yet – the important message is that it is sometimes desirable, and possible, to generate colour data values directly from a dataset.

## 4. Conclusions

Colour is a complex topic, drawing on elements of physiology, vision science and sociology as well as the more obvious domains of design.

This session has considered how to design the selection of colours in your data visualization to match the types of data you wish to represent. We will see this principle applied again and again in datavis design, where you attempt to convey the character of the data you are showing with by the character of its visual expression.

We have seen how _marks_ such as bars and circles can be modified by encoding them with the colour _channel_. A good starting point in this encoding is simply to specify the data measurement type (e.g. [mNominal](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mNominal), [mOrdinal](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mOrdinal), [mQuant](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mQuant)) and let Vega-Lite find an appropriate default colour encoding. But as you become more experienced you are likely to want to provide your own custom colour encodings, either from an empirically derived colour scheme or by explicitly setting your own colours.

## 5. Recommended Reading

**Kirk, A.** (2019) Chapter 9: _Colour_, pp.249-276 in [Data Visualisation: A Handbook for Data Driven Design, 2nd Edition](https://go.exlibris.link/rwjXvTCv), Sage.

**Munzner, T.** (2015) Chapter 10: _Color and other channels_, pp.218-235 in [Visualization Analysis and Design](https://go.exlibris.link/9jMy6fQG), CRC Press

**Kosara, R.** (2013) [How The Rainbow Color Map Misleads](http://eagereyes.org/basics/rainbow-color-map)

**Rost, L-C.** (2018) [Your friendly guide to colors in data visualization](https://blog.datawrapper.de/colorguide/). _Datawrapper blog_

**Stone, M.** (2006) [Choosing Colors for Data Visualization](https://www.perceptualedge.com/articles/b-eye/choosing_colors.pdf), _Perceptual Edge_.

**Ware, C.** (2021), Ch.4 Colour, pp.95-141 in [Information Visualization: Perception for Design](https://go.exlibris.link/MqVhLW9Y), London: Morgan- Kaufmann

### References

**Brewer, C., Hatchard, G. and Harrower, M.** (2003) [ColorBrewer in print: A catalog of color schemes for maps.](http://tinyurl.com/yb6zcldg) _Cartography and Geographic Information Science_, 30(1), pp.5-32

**Byron, L. and Wattenberg, M.** (2008) [Stacked graphs – geometry and aesthetics.](http://leebyron.com/streamgraph/stackedgraphs_byron_wattenberg.pdf) _IEEE Transactions on Visualization and Computer Graphics_, 14(6), pp.1245 - 1252.

**Caldwell-Harris, C.** (2019) [Our language affects what we see](https://www.scientificamerican.com/article/our-language-affects-what-we-see/). _Scientific American, January 2019_

## 6. Practical Exercises

{(task|} Please complete the following exercises before next week. They should be added to your session03 folder of your portfolio. Remember to stage, commit and push your answers to github so I can see them. {|task)}

### 1. Measurement Types and Colour Schemes

![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg)

Which measurement types (e.g. `Nominal`, `Ordinal`, `Quant`, `Temporal`) and which [colour schemes](https://vega.github.io/vega/docs/schemes/) (e.g. `category20`, `purples` etc.) would you specify for the following data. Briefly add your reasoning to the table below and then copy it to your exercises document in your coursework portfolio repo:

| Data                                                                                           | Measurement type | Colour Scheme | Justification |
| ---------------------------------------------------------------------------------------------- | ---------------- | ------------- | ------------- |
| 1. Student cohorts in an attendance streamgraph                                                |                  |               |               |
| 2. FTSE 100 indices over the last 12 months                                                    |                  |               |               |
| 3. Your assessment marks for all completed modules                                             |                  |               |               |
| 4. Political party of elected MPs                                                              |                  |               |               |
| 5. Responses to a 5-point [Likert](https://en.wikipedia.org/wiki/Likert_scale) survey question |                  |               |               |

### 2. Modifying the attendance streamgraph

Modify the attendance streamgraph provided near the top of these lecture notes so that it uses the colour scheme you've recommended in the question above. If your recommendation was the existing scheme, see if you can find a better scheme and briefly say how it is an improvement.

### 3. Global Temperature Data Challenge

Below is a simple line chart showing [global temperature anomalies](https://datahub.io/core/global-temp). That is, the difference in mean global temperature compared to the 1950-1980 average.

Modify the chart to make use of colour to emphasise the trends over time and possible seasonal cycles (the data are provided monthly since 1900). You will probably want to change the [line](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#line) mark to a [bar](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#bar) mark to help do this.

```elm {l v}
linechart : Spec
linechart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/temperatureAnomalies.json"
                [ parse [ ( "Anomaly", foNum ) ] ]

        enc =
            encoding
                << position X [ pName "Date", pTemporal ]
                << position Y [ pName "Anomaly", pQuant ]
    in
    toVegaLite [ width 600, data, line [], enc [] ]
```

Briefly add some text to your litvis document justifying your colour design choices.

### 4. Design Challenge

![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg)

One way of developing your design skills is deliberately to constrain your design choices and see if you can come up with creative solutions under those constraints. Using this idea of [constrained creativity](https://www.fastcompany.com/3067925/how-constraints-force-your-brain-to-be-more-creative), try to come up with a design for showing gender and name frequency over time (as shown in [Baby Name Voyager](http://www.babynamewizard.com/voyager) at the start of this lecture) that does not use colour (i.e. use only black or white in your design). For inspiration you may wish to have a look at the finalists in the [MonoCarto competition results](https://somethingaboutmaps.wordpress.com/monocarto-2019-winners/).

Sketch out your design and include a photo of it in your `practicalExercises.md` portfolio document. As a reminder, you can include an image in a litvis document like this:

```markdown
![image caption](myImagefile.jpg)
```

---

_Check your progress._

- [ ] I can specify colours explicitly using decimal and hex RGB triplets.
- [ ] I can specify colours using HSL triplets.
- [ ] I can specify colours using explicitly named colours.
- [ ] I can identify _nominal_, _ordinal_, _quantitative_ and _temporal_ measurement types.
- [ ] I can match measurement type to an appropriate colour scheme.
- [ ] I can specify colour encodings that accommodate the most common forms of colour vision deficiencies.
