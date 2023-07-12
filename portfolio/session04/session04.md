---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---
SSS
@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 4: Representing Data with Visual Variables

## Table of Contents

1.  [Introduction](#1-introduction)
2.  [Matching Visual Variables with Data and Tasks](#2-matching-visual-variables-with-data-and-tasks)
3.  [Perception of Visual Variables](#3-perception-of-visual-variables)
4.  [Conclusions](#4-conclusions)
5.  [Recommending Reading](#5-recommended-reading)
6.  [Practical Exercises](#6-practical-exercises)

{(aim|}

This session is designed to help you use a wider range of visual encodings in your designs by providing a set of guiding principles and properties that relate design choice to properties of data and the nature of the visualization task.

By the end of this session you should be able to:

- Draw upon Vega-Lite's full range of marks and channels in your visualization specifications.
- Select the most appropriate channel encoding for a data variable of a given measurement type.
- Assess the degree to which a channel supports tasks that are _quantitative_, _orderable_, _associative_ and _selective_.
- Optimise your design choices to support _detection_, _assembly_ and _estimation_.
- Apply perceptual scaling of symbol sizes if/when you think it appropriate.

{|aim)}

---

## 1. Introduction

In this session we will extend the theme introduced last week of altering the appearance of marks through data channels. But rather than just channels that affect colour, we will consider a wider range of data-driven channels and place them in a more general framework first identified by the French cartographer-designer Jacques Bertin.

As a reminder, we can think of a data visualization design as the specification of graphic _marks_ whose appearance is controlled though a series of _channels_ such as colour and position, in response to the values of variables in a dataset. These are terms used by [Munzner](#-recommended-reading) as well as Vega-Lite and several other data visualization environments. Good data visualization design is concerned with choosing the marks and the channels that influence them in a way that best conveys the properties of the data you wish to depict.

In Vega-Lite, so far we have focussed on marks of type [bar](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#bar), [circle](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#circle) and [line](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#line) and encoding channels of type [position X / position Y](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#position) and [color](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#color). There are others though, most of which are summarised below:

### Vega-Lite marks

| Mark                                                                                               | Description                                                                |
| -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [arc](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#arc)             | circular arc with an angular and areal extent, e.g. pie chart segments     |
| [area](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#area)           | continuous area with two bounded edges                                     |
| [bar](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#bar)             | rectangle, usually with a common baseline and customisable length          |
| [circle](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#circle)       | solid circular point mark                                                  |
| [geoshape](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#geoshape)   | geographic point, line or polygon such as country outline                  |
| [image](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#image)         | a rectangular raster image with a customisable width and height            |
| [line](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#line)           | line joining consecutive point locations                                   |
| [point](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#point)         | a single point feature that can have a customised shape                    |
| [rect](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#rect)           | a rectangle with customisable width and height                             |
| [rule](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#rule)           | a single straight line                                                     |
| [square](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#square)       | a square with customisable side length                                     |
| [textMark](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#textMark)   | some text                                                                  |
| [tick](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#tick)           | a short straight line with customisable position but not length            |
| [trail](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#trail)         | like a line, but with customisable width that can vary along its length    |
|                                                                                                    |                                                                            |
| [boxplot](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#boxplot)     | _compound mark_ for summarising a distribution with a 'box' and 'whiskers' |
| [errorband](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#errorband) | _compound mark_ for showing an areal range around a line                   |
| [errorbar](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#errorbar)   | _compound mark_ for showing a linear range around a point or line          |

### Vega-Lite encoding channels

Data-driven modifications to a mark's properties.

| Channel                                                                                                    | Description                                                    |
| ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| [angle](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#angle)                 | the angular direction (orientation) in which a mark is aligned |
| [color](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#color)                 | a mark's boundary and interior colour                          |
| [detail](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#detail)               | separate marks for different elements of a dataset             |
| [fill](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#fill)                   | the interior colour of a mark                                  |
| [fillOpacity](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#fillOpacity)     | the opacity of a mark's interior                               |
| [hyperlink](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#hyperlink)         | clickable URL destination of a mark                            |
| [order](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#order)                 | the order marks are displayed                                  |
| [position](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#position)           | the position of a mark's location or bounding edges            |
| [shape](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#shape)                 | the shape of a mark point symbol                               |
| [size](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#size)                   | the area, width or height of a mark                            |
| [stroke](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#stroke)               | the colour of a mark's boundary                                |
| [strokeDash](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#strokeDash)       | the dashed style of a line                                     |
| [strokeOpacity](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#strokeOpacity) | the opacity of a mark's boundary                               |
| [strokeWidth](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#strokeWidth)     | the width of a mark's boundary                                 |
| [text](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#text)                   | the text content of a mark                                     |
| [tooltip](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#tooltip)             | the text content of a tooltip associated with a mark           |

The idea that a visualization design can be broken down into these separable properties can be traced back to the highly influential work of [Jacques Bertin](https://en.wikipedia.org/wiki/Jacques_Bertin) in his [Semiology of Graphics](#references).

![Jacques Bertin at the giCentre](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/bertin.jpg)

## 2. Matching Visual Variables with Data and Tasks

Of particular value is Bertin's concept of the _retinal variable_ (more commonly referred to as the _visual variable_). Since his original work in 1967 there have been a number of suggested additions to the visual variables we can consider as designers. The figure below is based on the visual variables suggested by [MacEachren (1994)](#references) which are widely used today (although also contested by many).

![visual variables](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/visualVariables.png)

Note also, that several of them are not explicitly available in Vega-Lite as encoding channels (e.g. texture and focus), but it is useful at least to consider them as potentially data-modifiable properties of a design.

{(infobox|}

Wondering how texture may be used as a visual variable? Sometimes with wit, as in this map of the most common roofing materials in regions of France:

![roof tiles texture](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/roofTiles.jpg)

_Source: Pitte, J-R. (2003) Histoire du paysage français: De la Préhistoire à nos jours, p.163._

{|infobox)}

{(task|}

Remembering that data types can be classified into `Nominal`, `Ordinal` and `Quant`, which visual variables do you think are best suited to each of these measurement types? Complete the table below:

|     Visual variable | Suitable measurement types |
| ------------------: | -------------------------- |
|        _colour hue_ | Nominal                    |
| _colour saturation_ | Ordinal, Quant             |
|  _colour lightness_ | Ordinal                    |
|       _orientation_ | Nominal                    |
|             _shape_ | Nominal                    | 
|           _texture_ | Nominal                    |    -- Arrangement
|       _arrangement_ | Nominal                    |
|              _size_ | Ordinal, Quant will be hard to es             |
|             _focus_ | Ordinal, Quant             |
|          _location_ | Nominal                    |

{|task)}

Bertin identified four primary properties of visual variables, which can guide the choice of which to use for any given task. A visual variable is

**orderable** if changes in this variable can be used to place marks in some order;

**quantitative** if a numerical reading is obtainable from changes in this visual variable;

**selective** if a change in its properties alone is sufficient to isolate a mark from all others in a group of marks;

**associative** if marks that are alike in all other ways can be grouped according to change in a the visual variable or **dissociative** if differences in the visual variable prevent us from grouping marks together.

This is a useful set of properties because it allows us to relate our visualization design to higher level _tasks_ that we might wish our visualization to support. For example, if one of the tasks is to allow people to quickly place data items into distinct groups though visual inspection, we would need to encode our data with _associative_ visual variables / channels and avoid _dissociative_ encodings that would interfere with that task. Similarly, if we need people to be able to make visual assessments of relative magnitude of data values, we would need to encode with visual variables / channels that have _quantitative_ properties.

As an example, consider this scatterplot comparing the power, efficiency and country of origin of a set of cars:

```elm {l v }
associative : Spec
associative =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json" []

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << shape [ mName "Origin" ]
    in
    toVegaLite
        [ width 400
        , height 400
        , data
        , enc []
        , point [ maFilled True, maSize 200, maOpacity 0.3 ]
        ]
```

Shape symbols are generated by setting the mark to a [point](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#point) and adding a [shape](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#shape) encoding channel.

Despite a high degree of overlap, the use of shape to symbolise the three countries of origin is _associative_. That is, we are able visually to group the triangles together and associate them as being part of the same group of similar data. Note also that position is playing a strong associative role here too as we identify spatially grouped clusters, especially the group of US vehicles with high power / low fuel efficiency.

To see the role of dissociative encodings, consider this variation of the scatterplot where we additionally encode the name of each car with hue and the weight of each car with symbol size:

```elm {l v}
dissociative : Spec
dissociative =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json" []

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << shape [ mName "Origin" ]
                << size [ mName "Weight_in_lbs", mQuant ]
                << color [ mName "Name", mLegend [] ]
    in
    toVegaLite
        [ width 400
        , height 400
        , data
        , enc []
        , point [ maFilled True, maSize 200, maOpacity 0.5 ]
        ]
```

Now, our ability to group shapes is diminished as hue and size interfere with our ability to group by shape.

Note that most visual variables / channels that have an associative effect will also have a dissociative effect depending on what they encode. For example, if we _double encode_ country of origin with both shape and hue, they reinforce their associative properties despite the fact we are still encoding size with another data variable:

```elm {l v}
doubleEncoding : Spec
doubleEncoding =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json" []

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << shape [ mName "Origin" ]
                << color [ mName "Origin" ]
                << size [ mName "Weight_in_lbs", mQuant
                ]
    in
    toVegaLite
        [ width 400
        , height 400
        , data
        , enc []
        , point [ maFilled True, maSize 200, maOpacity 0.5 ]
        ]
```

{(infobox|}**Double encoding** (or triple encoding etc.) is the act of encoding some data variable with more than one channel simultaneously. It can have the advantage of reinforcing a particular property conveyed by the visualization (e.g. country of origin in the example above. Disadvantages include unnecessary complication of the visual design and therefore requiring extra cognitive effort to decode; and 'using up' channels that might otherwise have been available for encoding other data variables.{|infobox)}

{(task|}

Complete the table below by giving ratings of 1 (weak), 2 (moderate) or 3 (strong) for each of the properties of each visual variable.

(Selective - easy to pick out diffrence, associative - connection and similarities, dis)

|     Visual variable | quantitative | orderable | selective | associative | dissociative |
| ------------------: | :----------: | :-------: | :-------: | :---------: | :----------: |
|        _colour hue_ |      1       |     1     |     3     |      3      |      3       |
| _colour saturation_ |      3       |     3     |     3     |      1       |      3       |
|  _colour lightness_ |      3       |     3     |    1      |      3       |      3       |
|       _orientation_ |      1       |     1     |    3      |      2      |     2        |
|             _shape_ |      1       |     1     |    3      |      2      |     2        |
|           _texture_ |      1       |     1     |    3      |      2      |     2        |
|       _arrangement_ |      3       |     2     |    3      |      1      |     1        |
|              _size_ |      3       |     2     |    3      |      3      |     3        |
|             _focus_ |      1       |     2     |    3      |      1      |     3        |
|          _location_ |      1       |     1     |    3      |      3      |     3        |

{|task)}

### Expressiveness and effectiveness

[Munzner (2015)](#5-recommended-reading) approaches this idea from a slightly different perspective by considering the _expressiveness_ of different classes of channels and their _effectiveness_ at conveying the properties of the data they encode.

![effectiveness / expressiveness](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/effectiveness.png)
_Source: Munzner (2015) p.94_

These properties can be useful in providing guidance in choosing which to vary when representing different aspects of your data. You should think about what kind of question you intend your visualization to be able to answer. And then ensure you symbolise it with visual variables that are capable of providing those answers.

For example, if you want the reader of your visualization to be able to group items together such as tweets on a similar theme, you should consider using categorically expressive visual variables with good associative properties such as location, shape or colour hue. If on the other hand you want readers to be able to read the magnitude of something such as the size of annual salary bonuses paid to a particular profession, you would need to use visual variables with good quantitative effectiveness such as position on a common scale.

{(infobox|}Design usually involves trade-offs. It is often about prioritising which tasks are most important and matching the representations and encodings that are most effective to the tasks that are most important, recognising this may constrain the encoding options for less important tasks.{|infobox)}

### Customising Shape Encoding

You may have noticed that when encoding a data variable with shape, Vega-Lite allocates a set of default shapes to categories (circle, square and triangle in the car examples above). Just like encoding with colour, you can override these defaults by specifying your own shape encodings explicitly. The built-in shapes include: `circle`, `cross`, `diamond`, `square`, `triangle-up`, `triangle-down`, `triangle-left` and `triangle-right`.

You can specify a set of shapes to encode with in a similar way to that for categorical colours, using [categoricalDomainMap](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#categoricalDomainMap). Here is an example:

```elm {l v highlight=[7,8,9,10,11,12,18]}
customOriginShapes : Spec
customOriginShapes =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json" []

        myShapes =
            categoricalDomainMap
                [ ( "Europe", "cross" )
                , ( "Japan", "circle" )
                , ( "USA", "triangle-down" )
                ]

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << shape [ mName "Origin", mScale myShapes ]
    in
    toVegaLite
        [ width 400
        , height 400
        , data
        , enc []
        , point [ maFilled True, maSize 200, maOpacity 0.5 ]
        ]
```

If the pre-existing set of shapes is not sufficient, you can customise further by specifying [an SVG path](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths) that describes the geometry of the custom shape. For example, here we customise the USA symbol with our own path representing a right-pointing triangle.

```elm {l v highlight=[7,8,9,10,11,12,18]}
customOriginPath : Spec
customOriginPath =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/cars.json" []

        myShapes =
            categoricalDomainMap
                [ ( "Europe", "cross" )
                , ( "Japan", "circle" )
                , ( "USA", "m0 -1l0.866 0.5v1l-0.866 0.5l-0.866 -0.5v-1" )
                ]

        enc =
            encoding
                << position X [ pName "Horsepower", pQuant ]
                << position Y [ pName "Miles_per_Gallon", pQuant ]
                << shape [ mName "Origin", mScale myShapes ]
    in
    toVegaLite
        [ width 400
        , height 400
        , data
        , enc []
        , point [ maFilled True, maSize 200, maOpacity 0.5 ]
        ]
```

### Orientation

To use Bertin's _orientation_ visual variable we can encode a data value with Vega-Lite's [angle](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#angle) channel. This is especially useful when the data being encoded are 'circular' – that is, when values 'wrap around' across some value such as hours of the day (23:59 -> 00:00) or angles (359 degrees -> 0 degrees).

Here we use angle to show wind direction across NW Europe. Each wedge shape _glyph_ is sized according to wind speed and orientated according to the wind direction.

```elm {v l}
windVectorField : Spec
windVectorField =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/windvectors.csv" []

        enc =
            encoding
                << position Longitude [ pName "longitude" ]
                << position Latitude [ pName "latitude" ]
                << angle
                    [ mName "dir"
                    , mQuant

                    -- Make glyphs point in correct direction
                    , mScale [ scDomain (doNums [ 0, 360 ]), scRange (raNums [ 180, 540 ]) ]
                    ]
                << size [ mName "speed", mQuant ]
    in
    toVegaLite
        [ width 600
        , height 760
        , data
        , enc []
        , point [ maShape symWedge, maFilled True ]
        ]
```

Note that because the data are geographic (you might be able to make out the coastline of the British Isles and northern France), we encode position not with `X` and `Y` but `Longitude` and `Latitude`. More on this in Session 8 when we consider geographic mapping.

## 3. Perception of Visual Variables

When selecting visual variables in your visualization design, you should not only consider the data types you are using, but also the way in which those variables are perceived. Munzner’s effectiveness ordering of channels is in part based on experimental work that examined our ability to make judgements from graphical symbolisation. This approach to evaluation of design alternatives provides a more robust and informed basis for some design choices.

Cleveland (1993) considered the perception and comprehension of graphic elements and identified three perceptual activities required to make sense of a data graphic:

- **Detection** - the element of the graphic must be easily discernible.
- **Assembly** - the process of identifying patterned regularities and structures within the graphical elements of the visualization.
- **Estimation** - the process of making comparisons of the magnitudes of data items from the visual elements used.

### Detection

Ware (2021) usefully distinguishes between two forms of cognitive process used for detection, assembly and estimation - those of _attentive processing_ and _preattentive processing_. Attentive processing is the sequential conscious effort we make when attempting to understand a visual scene. In contrast, preattentive processing happens unconsciously and in parallel. Preattentive processing is the type of visual cognition that allows us to understand something of a visual scene 'at a glance' without conscious effort. This can be used in your visualization designs to reduce the cognitive load required of people to make sense of your graphics and extract information from them.

_In class this week we will try a few experiments to explore this distinction between attentive and preattentive processing of visual scenes._

The relation between unconscious preattention and conscious attention is aligned with _System 1_ and _System 2_ thinking discussed by Daniel Kahneman in his book [Thinking, Fast and Slow](https://en.wikipedia.org/wiki/Thinking,_Fast_and_Slow). While such a model is disputed by some, it does appear clear that the degree to which preattentive cognition takes place can vary according to the number of 'distractors', experience and assembly. See, for example, the work of [Haroz and Whitney (2012)](#references) who attempted to determine experimentally the effect of distractors and complexity on our ability to detect patterns and symbols.

### Assembly

![colon / colony](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/colony.png)

Our ability to assemble the visual field into patterns (that hopefully have meaning in a well-designed data visualization) is dependent on a complex set of interactions between perception and cognition. Our understanding of the assembly process has greatly benefited from the early 20th century _Gestalt school of psychology_ that attempted to formalise the we we perceive and assemble patterns in the visual field.

As an example, consider the gestalt principle of **proximity**, that states that perceptually we tend to group together things that are close to one another. Even small differences in proximity can affect how we assemble patterns:

{(task|}

What associations are implied by these arrangements of dots?

![proximity](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/gestaltProx.png)

_Adapted from Ware (2021) p.186._



{|task)}

{(annotation|}Your thoughts here.{|annotation)}

The gestalt principle of **continuity** suggests we are more likely to construct patterns that are smooth and continuous than ones that contain abrupt changes in direction. We tend to assemble the figure below-left as two lines with continuity (centre) rather than as two lines with discontinuities (right).

![continuity](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/gestaltCont.png)
_Adapted from Ware (2021) p.188._

For data visualization especially, an important gestalt principle is that of **connectedness**, that states connecting graphical objects with (simple) lines significantly enhances our ability to group objects, even if the proximity principle might suggest otherwise:

![connectivity](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/gestaltConnect.png)

[Ware (2021)](#references) pp.185-106 summarises the key principles and [Meeks (2017)](#5-recommended-reading) provides an accessible account of how the core gestalt principles can be used to inform good data visualization design.

### Estimation

For estimation of magnitude, it is useful to remind ourselves that our perceived magnitude of various visual variables is not always accurate or consistent, nor are we equally good at estimating magnitude for different types of shapes.

{(task|}

How much larger is the first circle than the second?

![circle area estimation](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/estimationCircles.png)

How much larger is the first rectangle than the second?

![bar area estimation](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/estimationBars.png)

Which of the two pairs of symbols did you find easier to compare? And why might this be so?

{|task)}

{(annotation|}Your thoughts here.{|annotation)}

We have already seen examples of colour contrast interfering with our ability to judge grey-level. In another example of perception interfering with an apparent objective truth, it is also the case that we do not always perceive area or volume in direct proportion to the actual area or volume of graphical symbol. The relationship between _stimulus_ (e.g. symbol size, or grey level) and _response_ (e.g. our perception of symbol size or grey level) forms the basis of [psychophysics](https://en.wikipedia.org/wiki/Psychophysics) which tries understand what is often a complex non-linear relationship.

The figure below summarises this perceptual non-linearity for perception of symbol size:

![non-linear area estimation](https://staff.city.ac.uk/~jwo/datavis2021b/session04/images/areaEstimation.png)

When estimating the length of lines, generally, the actual drawn length matches our perceived length of it (45 degree line on the left-hand chart). However, for areal features such as circles and squares, as symbol area increases, we tend to slightly under-estimate its true area relative to a smaller one of the same shape. As a result if we wish to produce a perceptually accurate symbolisation, we may choose to increase the ratio between larger and smaller symbols (second set of circles) beyond a simple linear relationship with area (first set of circles) in order to 'undo' our perceptual misjudgement.

[Stevens](#5-recommended-reading) suggested a power function describes most stimulus-response relations

$R = kS^n$

and the experimental work of [Flannery](https://www.utpjournals.press/doi/abs/10.3138/J647-1776-745H-3667) suggested that for circle size perception, $k=1$ and

$R = S^{0.87}$

in other words $\text{perceived circle area} = a^{0.87}$ where $a$ is the true circle area (noting that if we were able to judge area accurately that exponent would be 1 rather than 0.87).

It should be noted that there is significant variation in experimental findings as well as variation between different individuals' ability to make accurate area estimations. For example [Cleveland et al, 1982](#references) found that the exponent was approximately 1 (i.e. accurate area estimation) when many circles were placed in a single representation, as they might on a [proportional symbol map](https://www.axismaps.com/guide/univariate/proportional-symbols). In contrast ([Stevens and Guirao, 1963](#references)) found the exponent to be 0.7 (i.e. a significant underestimation of larger circle sizes). For a more informal discussion of this issue, see [perceptual scaling of map symbols](https://makingmaps.net/2007/08/28/perceptual-scaling-of-map-symbols) by John Krygier.

## Specifying a Flannery Correction in Vega-Lite

We can change the way data values are scaled to symbol area by specifying an [mScale](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mScale), in a similar way to when we specified a custom colour and custom shape. This time the scaling can be applied to the size of the symbol rather than colour value. By specifying a [scPow](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scPow) scale and some [scExponent](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scExponent) we can provide non-linear scalings of circle area. Here is a simple test to see the effect of different exponent values that also sets the maximum size of the largest circle explicitly to 50000 square units by specifying a [scRange](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scRange):

```elm {l}
circleScaling : Float -> Spec
circleScaling scaling =
    let
        data =
            dataFromColumns []
                << dataColumn "dataValue" (nums (List.map toFloat (List.range 1 10)))

        enc =
            encoding
                << size
                    [ mName "dataValue"
                    , mScale
                        [ scRange (raNums [ 10, 50000 ])
                        , scType scPow

                        -- Reciprocal because we to scale stimulus, not response
                        , scExponent (1 / scaling)
                        ]
                    , mLegend []
                    ]
    in
    toVegaLite [ data [], enc [], point [] ]
```

Notice that this is an example of a parameterised function (as we covered in the coding gym last week), where we need to provide a value for `scaling` when we call the function. Below we call the function three times providing a different value for the parameter each time. Litvis allows us to use the more compact 'triple-hat' syntax, in place of the normal 'triple-backtick' style, making it easier to embed in a markdown table :

| exponent of 1 (default; no correction) | exponent of 0.87 (Flannery correction) | exponent of 0.5 (diameter judgement) |
| -------------------------------------- | -------------------------------------- | ------------------------------------ |
| ^^^elm {v=(circleScaling 1)}^^^        | ^^^elm {v=(circleScaling 0.86)}^^^     | ^^^elm {v=(circleScaling 0.5)}^^^    |

To see an example with real data, below is "bubble chart" that sizes circles in proportion to country population, using the default scaling:

```elm {v l highlight=[16,17,18]}
logBubble : Spec
logBubble =
    let
        data =
            dataFromUrl "https://vega.github.io/vega-lite/data/gapminder-health-income.csv"

        enc =
            encoding
                << position X
                    [ pName "income", pScale [ scType scLog ], pAxis [ axGrid False ] ]
                << position Y
                    [ pName "health", pQuant, pScale [ scZero False ], pAxis [ axGrid False ] ]
                << size
                    [ mName "population"
                    , mScale
                        [ scRange (raNums [ 0, 5000 ])
                        , scType scPow -- Area proportional to datum raised to power of some exponent
                        , scExponent 1.0 -- Default value (i.e. area directly proportional to datum)
                        ]
                    ]
    in
    toVegaLite [ width 500, height 350, data [], enc [], circle [] ]
```

{(task|}Try changing the value of `scExponent` above from `1.0` (default) to `0.87` (Flannery) and then `0.7` (Stevens). What effect does this have on the chart? Does it help in magnitude estimation tasks? {|task)}

{(task|}Is scaling a circle area by raising it to the power other than 1 'cheating'? Which is more honest, to ensure the graphical size is consistently proportional to the magnitude of the data it represents, or to scale it so it is perceived as proportional to magnitude?{|task)}

{(annotation|}Your thoughts here.{|annotation)}

## 4. Conclusions

As we progress through the module, I have introduced additional marks and encoding channels so you have a wider choice of design options in your data visualization. We are no longer restricted to simple bar charts and scatterplots.

With this wider choice comes the risk of _design paralysis_ – you become overwhelmed with choices and don't know where to start making design decisions. We have already seen how sketching and branching narratives can help in navigating this increasingly large design space. In this session we have considered another approach – the identification of the properties each of those choices has, and related those properties to both the data we are visualizing and the perceptual and cognitive processes of detection, assembly and estimation. This helps us by more systematic in answering the question _"how well do my design choices help in achieving the task my data visualization is supporting?"_. Other theories of perception, such as the Gestalt principles become increasingly useful as our visualizations become more complex. In future weeks, as we consider other ways to enrich and assemble more sophisticated visual designs, remember these core principles and properties to help make sensible design choices that meet the objectives of your visualizations.

## 5. Recommended Reading

To become good at data visualization you will need to read beyond the lecture notes. 'Recommended reading' points you to accessible sources of further information related to the content of each session. At the very minimum, please read the relevant chapter of either Kirk (2019) or Munzner (2015).

**Kirk, A.** (2019) Chapter 6: _Data Representation_, pp.135-201 in [Data Visualisation: A Handbook for Data Driven Design, 2nd Edition](https://go.exlibris.link/rwjXvTCv), Sage.

**Munzner, T.** (2015) Chapter 5: _Marks and Channels_, pp.94-115 in [Visualization Analysis and Design](https://go.exlibris.link/9jMy6fQG), CRC Press

**Meeks, E.** (2017) [Gestalt principles for data visualization](https://medium.com/@Elijah_Meeks/gestalt-principles-for-data-visualization-59f18f20bd40). _Medium, March 2017_

**Roth, R.** (2017) [Visual Variables](https://www.researchgate.net/profile/Robert_Roth8/publication/317266613_Visual_Variables/links/5a66f7e24585158bca55366c/Visual-Variables.pdf) in _The International Encyclopedia of Geography: People, The Earth, Environment, and Technology_. Wiley ISBN: 978-0-470-65963-2

### References

**Bertin, J.** (2010) [The Semiology of Graphics](https://go.exlibris.link/Pcb5W5J3). ESRI Press, ISBN 1589482611 (translation of the original 1967 _La Semiologie Graphique_)

**Carpendale, M.S.T.** (2003) [Considering visual variables as a basis for information visualisation](https://prism.ucalgary.ca/handle/1880/45758), _University of Calgary Computer Science Technical Report_ #2001-693-16

**Cleveland, W., Harris, C. and McGill. R.** (1982) [Judgments of circle sizes on statistical maps](https://www.jstor.org/stable/pdf/2287708.pdf). _Journal of the American Statistical Association_, 77.379 pp.541-547.

**Cleveland, W.** (1993) _The Elements of Graphing Data_, Hobart Press, ISBN 0963488411

**Flannery, J.** (1971) The relative effectiveness of some common graduated point symbols in the presentation of quantitative data. _Cartographica: 8(2) pp.96–109._

**Haroz, S., and Whitney, D.** (2012) [How capacity limits of attention influence information visualization effectiveness](https://whitneylab.berkeley.edu/PDFs/Haroz_Whitney_2012_InfoVis.pdf). _IEEE Transactions on Visualization and Computer Graphics_, 18(12) pp.2402-2410.

**MacEachren, A.** (1994) _Some Truth With Maps: A Primer on Symbolization and Design_, Association of American Geographers, ISBN 0892912146

**Stevens, S. and Guirao, M.** (1963) [Subjective scaling of length and area and the matching of length to loudness and brightness](https://hz7fp2gl5k.search.serialssolutions.com/log?L=HZ7FP2GL5K&D=EPA&J=JOURNOFEXPEPSY&P=EJP&PT=Redirector&H=55252a0fab&U=https%3A%2F%2Fsearch.ebscohost.com%2Fdirect.asp%3Fdb%3Dpdh%26jid%3DXGE%26scope%3Dsite%26authtype%3Dip%2Cshib%26custid%3Ds1089299), _Journal of Experimental Psychology_ 66(2) pp.177-186.

**Ware, C.** (2021) [Information Visualization: Perception for Design](https://go.exlibris.link/MqVhLW9Y), London: Morgan- Kaufmann

## 6. Practical Exercises

{(task|} Please complete the following exercises before next week. {|task)}

### 1. Matching visual variables to data type and task property

Make sure you have completed the visual variable table tasks in the lecture notes above. If you are having trouble completing either of the tables, you may find [Roth (2017)](#5-recommended-reading) provides helpful guidance.

Discuss your choices with others in the lab class to see where you agree / disagree.

### 2. Graphical summary of channel properties

Consider the table of visual variables and the ratings you have given each of the 5 properties. In the example below, all ratings are 1, but when you have completed the question above, you should have values in the table between 1 and 3.


|     Visual variable | quantitative | orderable | selective | associative | dissociative |
| ------------------: | :----------: | :-------: | :-------: | :---------: | :----------: |
|        _colour hue_ |      1       |     1     |     3     |      3      |      3       |
| _colour saturation_ |      3       |     3     |     3     |      1       |      3       |
|  _colour lightness_ |      3       |     3     |    1      |      3       |      3       |
|       _orientation_ |      1       |     1     |    3      |      2      |     2        |
|             _shape_ |      1       |     1     |    3      |      2      |     2        |
|           _texture_ |      1       |     1     |    3      |      2      |     2        |
|       _arrangement_ |      3       |     2     |    3      |      1      |     1        |
|              _size_ |      3       |     2     |    3      |      3      |     3        |
|             _focus_ |      1       |     2     |    3      |      1      |     3        |
|          _location_ |      1       |     1     |    3      |      3      |     3        |

The following function will create a table of data that represents the table (again illustrated with ratings of 1 for all table cells).

```elm {l}
vvTable =
    let
        table =
            """visualVariable,    q, o, s, a, d
               colour hue,        1, 1, 3, 3, 3
               colour saturation, 3, 3, 3, 1, 1
               colour lightness,  3, 3, 1, 3, 3
               orientation,       1, 1, 3, 2, 2
               shape,             1, 1, 2, 3, 3
               texture,           1, 1, 2, 3, 3
               arrangement,       3, 2, 3, 1, 1
               size,              3, 2, 3, 3, 3
               focus,             1, 1, 3, 1, 3
               location,          1, 1, 3, 2, 2"""
                |> fromCSV
                |> gather "property"
                    "rating"
                    [ ( "q", "quantitative" )
                    , ( "o", "orderable" )
                    , ( "s", "selectable" )
                    , ( "a", "associative" )
                    , ( "d", "dissociative" )
                    ]
    in
    dataFromColumns []
        << dataColumn "visualVariable" (table |> strColumn "visualVariable" |> strs)
        << dataColumn "property" (table |> strColumn "property" |> strs)
        << dataColumn "rating" (table |> numColumn "rating" |> nums)
```

Update the '1's in this function to reflect your own ratings. Then see if you can produce a graphical summary of the table by encoding the ratings with one or more channels that you think provides a descriptive graphical summary.

You can use the following as a start point:

```elm {l v}
summary : Spec
summary =
    let
        enc =
            encoding
                << position Y [ pName "visualVariable" ]
                << position X [ pName "property" ]
                << color [ mName "Dark2", mLegend [] ]
    in
    toVegaLite [ width 400, height 400, vvTable [], enc [], point [] ]
```
]
### 3. Design Challenge: Exploring data-channel design options

The [Gapminder](https://www.gapminder.org/data/) dataset at [vega.github.io/vega-lite/data/gapminder-health-income.csv](https://vega.github.io/vega-lite/data/gapminder-health-income.csv) shows the relationship between health, income and population numbers for the countries of the world.

| country             | income | health | population |
| ------------------- | ------ | ------ | ---------- |
| Afghanistan         | 1925   | 57.63  | 32526562   |
| Albania             | 10620  | 76     | 2896679    |
| Algeria             | 13434  | 76.5   | 39666519   |
| Andorra             | 46577  | 84.1   | 70473      |
| Angola              | 7615   | 61     | 25021974   |
| Antigua and Barbuda | 21049  | 75.2   | 91818      |
| Argentina           | 17344  | 76.2   | 43416755   |
| :                   | :      | :      | :          |

Create one or more litvis documents that explore some design possibilities for showing these data. In particular focus on exploring the use of different channels (e.g. position, colour, shape, size) to encode different data variables (country, income, health, population). You should come up with several different encodings and provide a brief evaluation of which ones work most effectively.

Some issues to consider:

- Are the best designs consistent with the guidance given for associating channels with data type (nominal, quant etc.) and task property (associative, orderable etc.)?
- Does double-encoding help or hinder interpretation?
- What are the advantages / disadvantages of encoding all variables in a single chart compared to separating them in multiple charts?
- Does perceptual (Flannery) scaling help in interpretation?

---

_Check your progress._

- [ ] I can use all of Vega-Lite's _marks_ (e.g. line, trail, textMark, point, circle, bar etc.) in my specifications.
- [ ] I can select the most appropriate channel encoding for a data variable of a given measurement type.
- [ ] I can evaluate the degree to which a channel supports tasks that are _quantitative_, _orderable_, _associative_, _dissociative_ and _selective_.
- [ ] I can make design choices that support _detection_, _assembly_ and _estimation_ tasks.
- [ ] I can apply perceptual scaling of symbol sizes and know when it is appropriate to apply it.
