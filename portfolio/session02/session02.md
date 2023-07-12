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

# Session 2: Approaches to Design

## Table of Contents

1.  [Introduction](#1-introduction)
2.  [Data Humanism](#2-data-humanism)
3.  [Edward Tufte](#3-edward-tufte)
4.  [Visualization Algebra](#4-visualization-algebra)
5.  [Conclusions](#5-conclusions)
6.  [Recommending Reading](#6-recommended-reading)
7.  [Practical Exercises](#7-practical-exercises)

{(aim|}

This session is designed to widen your appreciation of approaches to visualization design and provide you with some frameworks to support your own design process.

By the end of this session you should be able to:

- Recognise and apply four contrasting approaches to visualization design.
- Distinguish visualization objectives such as _explanation_, _analysis_, _reflection_ and _artistic expression_.
- Create _branching narratives_ that explore alternative design options for a visualization.
- Specify a Vega-Lite visualization that follows some of Tufte's design principles.
- Identify _hallucinators_, _distractors_, _jumblers_ and _misleaders_ in a visualization.

{|aim)}

---

## 1. Introduction

You have a problem in mind that you wish to investigate. You've found a dataset and a platform capable of generating data visualizations. But how do you design your visualization so that it does what you want it to? What kind of design decisions do you have to make and how do you know if they are good decisions?

This session introduces the process of data visualization design, exploring different approaches to that process along with perspectives and models that assist in structuring that process. We will see how Literate Visualization can help you organise your own data visualization design and construction.

To start, in advance of Wednesday's live session, spend 15 minutes reading [this account of the design process](http://style.org/stdp3/) created by Jonathan Corum, the New York Times science graphics editor, and consider the questions following (you can add your thoughts to the annotation space below them).

![See, Think, Design, Produce](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/seeThinkDesignProduce.png)

{(task|}

- What pieces of advice from the talk are most persuasive to you?
- To what extent do you agree with Corum's statement that _"the goal of the design process is to find a clear thought and then show it."_?
- To what extent do you agree with Corum's statement that _"visualization is not counting"_ and that it should only _"find and show **meaningful** patterns"_?

{|task)}

{(annotation|}

Add your answers here.

{|annotation)}

I'd like to draw out three points he makes and show how we can use Literate Visualization to support them.

Firstly, Corum makes the case for _sketching_ as part of the design process, suggesting it is a form of "visual problem solving" and that because sketches are quick to generate, they are not "commitments" to a particular design and allow you to explore different design possibilities easily.

Secondly, he asserts _"let the data lead you to the design"_. In other words, he suggests a visualization design will be tailored to the specific data you are visualizing, not solely the result of some universal design rules.

And thirdly, that _"design is cumulative decision making"_, that it is about applying many small decisions, not just one single imposed "design".

### Branching narratives

To facilitate exploring different design choices, sketching is useful because it is a relatively low-cost way of considering a particular design. Low enough that you can chose different designs and don't feel committed to any one of them. In _litvis_, one way of easily incorporating paper sketches is simply to take a photo of a sketch and include it in your litvis document.

As we saw in the practical exercises from last week, to include an image in markdown you use the following format:

```markdown
![tooltip / missing image text](path/to/image.jpg)
```

where `path/to/image.jpg` is either the location and name of the image file relative to the litvis document, or an absolute URL pointing to the image.

We might, for example, include a sketch of a possible bike hire bar chart (from last week):

![Sketch 1](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/sketch1.jpg?q=0)

As you can see from my example, you do not need to be able to draw well for sketching to be a useful activity!

We might explore a different way of comparing number of journeys with journey time, and sketch that out too. The sketch might identify features you don't yet know whether you want to include:

![Sketch 2](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/sketch2.jpg)

These two sketches give us two starting points for our designs from which we might only eventually select one. But it can be useful to at least consider both and refine each of them before coming to a decision (this is what Corum refers to as "sketching is not a commitment" and "ugly is fine").

In litvis, we can create two parallel "branches" as separate documents that explore each of these alternative designs. We might start with a _root document_ that contains the setup and dataset common to both designs:

#### `rootExample.md`

````markdown
---
id: litvis

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

```elm {l=hidden}
import VegaLite exposing (..)
```

# Bicycle Hire Datavis Designs

Here is the dataset we will use to explore alternative designs:

```elm {l}
data =
    dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"
```
````

Note how we have created a `data` function as a top-level function in its own code block, so that it is available to any other code blocks in our litvis documents.

We could choose to add further code blocks to the document in order to specify the visualization designs, but instead we will create completely separate documents that `follow` the root document:

#### `Design1Example.md`

````markdown
---
follows: rootExample
---

## Design 1: Show as a time series

Aim to show the data as a time series so that we can easily see temporal trends and cycles:

![Time series design sketch](images/sketch1.jpg)

```elm {v}
myBarchart : Spec
myBarchart =
    let
        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
                << color [ mName "AvHireTime", mQuant ]
    in
    toVegaLite [ width 640, data [], enc [], bar [] ]
```
````

#### `Design2Example.md`

````markdown
---
follows: rootExample
---

## Design 2: Show as scatterplot

Aim to emphasise the possible correlation between number and length of journeys,
but still provide some indication of when journeys occur in case this can be
associated with the number-length relationship :

![Scatterplot design sketch](images/sketch2.jpg)

```elm {v}
myScatterplot : Spec
myScatterplot =
    let
        enc =
            encoding
                << position Y [ pName "AvHireTime", pQuant ]
                << position X [ pName "NumberOfHires", pQuant ]
                << color [ mName "Month", mTemp ]
    in
    toVegaLite [ width 500, height 500, data [], enc [], circle [] ]
```
````

Note how both 'following' documents have access to the `data` function we have created in `rootExample.md`. Keeping the designs separate in `designExample1.md` and `designExample2.md` allows us to continue to work with those separate designs independently of each other as we continue to iterate on each design. Working on each in parallel rather than completing one before starting the other can help avoid the [sunk cost fallacy](https://en.wikipedia.org/wiki/Sunk_cost) (or 'plan-continuation bias') where the more effort we put into something, the less likely we are to abandon it, even if we know there are better alternatives.

A willingness to try many alternatives is an important part of exploring the "design space" in datavis, keeping your mind open to a range of approaches for any given data visualization challenge, without initially committing to any one design.

## 2. Data Humanism

Corum's approach to design is not the only way we might conceive of the design process and design goals. A recent movement, proposed by [Giorgia Lupi](http://giorgialupi.com) and exemplified by [RJ Andrews](https://infowetrust.com/inspire/) and others is [data humanism](https://medium.com/@giorgialupi/data-humanism-the-revolution-will-be-visualized-31486a30dbfb). It can be seen as a manifesto for a certain approach to design, summarised by Lupi's call to action:

> We are ready to question the impersonality of a merely technical approach to data, and to begin designing ways to connect numbers to what they really stand for: knowledge, behaviors, people.

It encompasses many of the traits of _feminist data visualization_ proposed by [D'Ignazio and Klein (2020)](https://data-feminism.mitpress.mit.edu), recognising that data visualization is a human process, not just in its interpretation but in its construction, contextualisation and the power structures it embodies.

{(task|}

As an example, consider [Sara Weber's train delay scarf](https://www.theguardian.com/world/2019/jan/14/german-train-delay-scarf-ebay-commute).

![Train Delay Scarf](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/weberScarf.jpg) _Photo: Sara Weber / Twitter_

Is this a data visualization? What elements of it embrace _data humanism_ and _feminist data visualization_?

{|task)}

{(annotation|}

Add your answers here.

{|annotation)}

A common element of data visualization that reflects data humanism is its hand-constructed nature. This both reflects the labour of generating it and frames its interpretation ([Wood et al 2014](#references)).

Probably the most widely known example of data humanism visualization is the postcard correspondence between Giorgia Lupi and Stefanie Posavec in [Dear Data](http://www.dear-data.com/theproject). They spent a year sending each other hand-drawn personal data visualizations every week, each capturing a different aspect of their lives.

![Example Dear Data Postcard](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/dearData.jpg) _Image: Stefanie Posavec / Dear Data_

{(task|}

Can the approach represented by Lupi and Posavec in _Dear Data_ be reconciled with Corum's assertion that "visualization is not counting"? Apart from the subject matter how does personal visualization differ from other forms of data visualization?

{|task)}

{(annotation|}

Add your answers here.

{|annotation)}

For some further inspiration you may wish to have a look at how Stefanie Posavec has been using ideas of personal visualization and data humanism for [health monitoring](https://peoplelikeyou.ac.uk/blog/figuring-out-health-research-data/):

![Personal health monitoring visualization](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/posavecHealth.jpg) _Image: Stefanie Posavec / People Like You_

## 3. Edward Tufte

The approach to design proposed by Edward Tufte in his three books [The Visual Display of Quantitative Information (1983)](#references), _Envisioning Information_ (1990) and _Visual Explanations_ (1997) has had significant impact on data visualization over the last 40 years. Through illustration of hundreds of historical examples, he advocates a _minimalist_ approach to design with an emphasis on _graphical integrity_, _data ink_ and _data density_.

{(infobox|}

Many people confuse _minimalism_ with 'minimal' or 'simple'. It is important to recognise that Tufte is not arguing that datavis should only show simple things (in fact, quite the opposite). What he has argued for is that data graphics should show data in the simplest way possible, and in doing so, allow more complex depictions to be more easily understood.

In previously submitted coursework I have seen students argue _"Following the principles of Tufte, I have chosen just to show just this tiny selection of data"_. Please don't fall into this trap!

{|infobox)}

### Data-Ink Ratio and Chart-Junk

A rule of thumb advocated by Tufte is to maximise the _data-ink ratio_. That is, consider all the "ink" that makes up a data visualization and divide it into ink that directly represents data (e.g. the bars in a bar chart), and the ink that does not (e.g. border around a chart). He argues the quality of a graphic is, in part, determined by the ratio of the data ink / non-data ink (the higher the better).

This is a useful notion at least to consider in design, although it is one that is [disputed by some](https://eagereyes.org/blog/2013/definition-chart-junk).

One of the principal advantages of maximising the data-ink ratio is that it makes the combination of many charts in _small multiples_ easier to interpret. Consider, for example, a line chart showing previous weekly lecture attendance for a teaching module. This is what it looks like using Vega-Lite with the default settings:

```elm {v}
attendanceChart : Spec
attendanceChart =
    let
        data =
            dataFromColumns []
                << dataColumn "week" (nums [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ])
                << dataColumn "numStudents" (nums [ 98, 89, 92, 73, 89, 80, 88, 69, 66, 70 ])

        enc =
            encoding
                << position X [ pName "week" ]
                << position Y [ pName "numStudents", pQuant ]
    in
    toVegaLite [ data [], enc [], line [] ]
```

If we remove the axes and scale between the maximum and minimum vales, we can produce a more compact design, which Tufte called a _sparkline_:

```elm {l=hidden }
sparkline : List Float -> Spec
sparkline attendance =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        data =
            dataFromColumns []
                << dataColumn "week" (nums [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ])
                << dataColumn "numStudents" (nums attendance)

        enc =
            encoding
                << position X [ pName "week", pAxis [] ]
                << position Y
                    [ pName "numStudents"
                    , pQuant
                    , pAxis []
                    , pScale [ scZero False ]
                    ]
    in
    toVegaLite [ height 30, cfg [], data [], enc [], line [ maStroke "black" ] ]
```

^^^elm {v=(sparkline [ 98, 89, 92, 73, 89, 80, 88, 69, 66, 70 ])}^^^

This allows us to embed the line in tables and more complex assemblages of graphics. As a simple example:

| Year | Attendance                                                           | Comments                                   |
| ---- | -------------------------------------------------------------------- | ------------------------------------------ |
| 2021 | ^^^elm {v=(sparkline [ 90, 91, 92, 90, 90, 87, 89, 88, 88, 90 ])}^^^ | Covid: Fully online teaching               |
| 2020 | ^^^elm {v=(sparkline [ 94, 89, 92, 90, 89, 60, 63, 74, 66, 79 ])}^^^ | Covid: Move to online teaching from week 5 |
| 2019 | ^^^elm {v=(sparkline [ 98, 89, 92, 73, 89, 80, 88, 69, 66, 70 ])}^^^ | Snow in week 4                             |
| 2018 | ^^^elm {v=(sparkline [121,107,89,87,94,93,63,70,72,59 ])}^^^         | No attendance feedback                     |
| 2017 | ^^^elm {v=(sparkline [135,123,95,91,124,92,88,75,76,74 ])}^^^        | Coursework released in week 5              |
| 2016 | ^^^elm {v=(sparkline [119,34,97,89,107,102,75,90,79,93])}^^^         | What happened in week 2?                   |

As part of his minimalist design approach, Tufte refers to elements of a data visualization that do not directly represent data or support understanding as _chart junk_. For example, consider the visualizations in the pages below (from the Information Is Beautiful Awards 2013):

![Chart junk](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/tnsFood.png)
Source: [TNS Food For Thought Brochure](https://www.tnsglobal.com/sites/default/files/TNS-SIAL-Food-brochure.pdf).

{(task|}

Identify the "data ink" and "non-data ink" in the graphics above. Would you consider any of it "chart junk" ? Why do you think some data visualizations include chart-junk?

{|task)}

{(annotation|}

Add your answers here.

{|annotation)}

As with the data-ink ratio, the use or rejection of 'chart junk' is an instructive design position to consider. You could, for example, create two sets of designs using _branching narratives_ (as above), one following Tufte's design approach, and one that does not. There is some evidence that in certain contexts, chart junk can be useful (e.g. [Borkin et al 2012](#references)), although it is important to consider what those contexts are and the purpose of the datavis.

#### Graphical Integrity and the Lie-Factor

Tufte defines the notion of _graphical integrity_ in design, and in particular that size of an effect implied by a graphic (e.g. bar height) should be in direct proportion to the size of the effect in the data it represents (e.g. measurement value). This might seem obvious, but there are many cases where this is not the case. Tufte suggests this can be quantified with a _lie factor_:

$\text{Lie Factor} = \frac{\text{size of effect shown in graphic}} {\text{size of effect in data}}$

Consider the lie factor(s) in the following example that intentionally confuses ordinal (1st, 2nd, 3rd) with quantitative (percentage results) data:

![Green Poster](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/greenPoster.png)

{(task|}

What is the lie factor of the following [visualization produced by the BBC](https://www.bbc.co.uk/news/uk-politics-46738126)?

![BBC Migration graphic](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/migration.png)

{|task)}

{(annotation|}

Add your answer here.

{|annotation)}

{(infobox|}

Tufte's "lie factor" is a very literal quantification of "lying" with visualization. But we can also consider lying when we fail to interpret a graphic correctly, especially if the visualization designer knows that it could be misinterpreted by many people. I recommend spending an hour listening to this entertaining talk by Alberto Cairo on [How Charts Lie](https://youtu.be/Low28hx4wyk) where he illustrates with many examples how lack of 'graphicacy' can lead to being lied to with visualization. It also provides a nice general introduction to the process of data visualization.

{|infobox)}

### A Tufte Makeover

Let's consider how we might apply some of Tufte's principles of information design to a default visualization produced by Vega-Lite.

#### The data

Before we look at the design, we will first create a simple dataset, not from a URL, but in our code directly, here recording the proportions of people who voted to leave the EU in the 2016 referendum, [estimated by Ipsos-MORI](https://www.ipsos.com/ipsos-mori/en-uk/how-britain-voted-2016-eu-referendum):

```elm {l}
brexitData =
    dataFromColumns []
        << dataColumn "ageGroup" (strs [ "18-24", "25-34", "35-44", "45-54", "55-64", "65-74", "75+" ])
        << dataColumn "leave" (nums [ 0.25, 0.4, 0.45, 0.56, 0.61, 0.66, 0.63 ])
```

#### Default design

Here's a simple bar chart as created with the default options in Vega-Lite:

```elm {l v}
vlChart : Spec
vlChart =
    let
        enc =
            encoding
                << position X [ pName "ageGroup" ]
                << position Y [ pName "leave", pQuant ]
    in
    toVegaLite [ brexitData [], enc [], bar [] ]
```

It does a pretty good job at laying out the bars and labelling to ensure we know how to interpret each column. But it doesn't reflect many of Tufte's design principles.

#### 1. Remove all non-data ink

Following Tufte's principle of maximising the data-ink ratio, we should ask ourselves, _can we increase the proportion of "ink" used to show the data?_

One approach is to start by removing _all_ non-data ink and then consider adding back only those elements necessary in the context of the visualization.

```elm {v l siding highlight=[6,10,11,13]}
tufteChart : Spec
tufteChart =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        enc =
            encoding
                << position X [ pName "ageGroup", pAxis [] ]
                << position Y [ pName "leave", pQuant, pAxis [] ]
    in
    toVegaLite [ cfg [], brexitData [], enc [], bar [] ]
```

We do this by adding the [pAxis](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pAxis) option to the position encodings in lines 10 and 11. This function allows us to customise the appearance of an axis, but in this case by providing an empty list (`[]`), we are saying we do not wish the axis to appear at all.

Additionally, we add a [configure](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#configure) option that allows us to modify the style of the chart. Here, [coView](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#coView) states we wish to configure the general view of the chart, stating that it should have no thin grey border (try removing line 6 to see what effect it has).

#### 2. Add bar labels

While we can see the trend in bar heights, too much of the context has been lost, making it hard to know which age group is represented by each bar. So we can add the labels back in again (but no need for the tick marks or baseline):

```elm {v l siding highlight=[7,11]}
tufteChart : Spec
tufteChart =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])

        enc =
            encoding
                << position X [ pName "ageGroup", pAxis [ axTitle "" ] ]
                << position Y [ pName "leave", pQuant, pAxis [] ]
    in
    toVegaLite [ cfg [], brexitData [], enc [], bar [] ]
```

We do two things here. Firstly we add an option to `pAxis` in line 11 stating that we wish the axis title to be blank. As soon as any option is added to `pAxis`, the full axis is added again, so additionally we can add an axis configuration option, [coAxis](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#coAxis) (line 7), to say we wish to have no tick marks and no 'domain' (axis base-line).

#### 3. Add bar height labels

We can apply a similar process to the Y-axis, but this time consider what would be the minimal amount of information we could usefully add. In the context of the EU referendum, the key value is 50%, determining whether or not a majority in any age group voted to leave, so let's only add the 50% marker:

```elm {v l siding highlight=[15]}
tufteChart : Spec
tufteChart =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])

        enc =
            encoding
                << position X [ pName "ageGroup", pAxis [ axTitle "" ] ]
                << position Y
                    [ pName "leave"
                    , pQuant
                    , pAxis [ axTitle "", axValues (nums [ 0.5 ]), axFormat "%" ]
                    ]
    in
    toVegaLite [ cfg [], brexitData [], enc [], bar [] ]
```

Here we specify the axis values explicitly with [axValues](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#axValues), providing just the critical 50% marker (represented by `0.5` scaled as values are in the original dataset).

#### 4. Improving the readability of the chart

Tufte suggests that because horizontal text is easier to read than vertical, we should endeavour to layout text horizontally where possible. In our case, Vega-Lite has (sensibly) arranged bar labels vertically as the chart is not wide enough to fit the text any other way.

By enlarging the chart, we can make sufficient room. How wide and tall should we make it? Tufte suggests that creating charts with an aspect ratio (width divided by height) of the [golden ratio](https://en.wikipedia.org/wiki/Golden_ratio) makes graphics easy to interpret and pleasing to view:

```elm {l}
goldenRatio : Float
goldenRatio =
    1.618
```

```elm {v l siding highlight=[8,21,22]}
tufteChart : Spec
tufteChart =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis [ axcoTicks False, axcoDomain False, axcoLabelAngle 0 ])

        enc =
            encoding
                << position X [ pName "ageGroup", pAxis [ axTitle "" ] ]
                << position Y
                    [ pName "leave"
                    , pQuant
                    , pAxis [ axTitle "", axValues (nums [ 0.5 ]), axFormat "%" ]
                    ]
    in
    toVegaLite
        [ width 450
        , height (450 / goldenRatio)
        , cfg []
        , brexitData []
        , enc []
        , bar []
        ]
```

This looks pretty ugly, with the bar widths out of proportion with the text, so lets shrink the width of the bars and give them a more muted shade, which will also help to give greater relative prominence to that critical 50% threshold and allow us to move the bar labels to within the bar space:

```elm {v l siding highlight=[12,20,35]}
tufteChart : Spec
tufteChart =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration
                    (coAxis
                        [ axcoTicks False
                        , axcoDomain False
                        , axcoLabelAngle 0
                        , axcoLabelPadding -12
                        ]
                    )

        enc =
            encoding
                << position X
                    [ pName "ageGroup"
                    , pScale [ scPaddingInner 0.5 ]
                    , pAxis [ axTitle "" ]
                    ]
                << position Y
                    [ pName "leave"
                    , pQuant
                    , pAxis [ axTitle "", axValues (nums [ 0.5 ]), axFormat "%" ]
                    ]
    in
    toVegaLite
        [ width 450
        , height (450 / goldenRatio)
        , cfg []
        , brexitData []
        , enc []
        , bar [ maFillOpacity 0.5 ]
        ]
```

The gap between bars can be controlled by changing the position scaling ([pScale](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pScale)). The function [scPaddingInner](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#scPaddingInner) is followed by a number representing the proportionate size of the inner gap between bars (where 0=no gap, 0.5 is half the width of a bar, 0.667 is two-thirds the bar width etc.). By setting a negative axis label padding (line 12) we can move the labels upwards into the bottom of each bar. Finally, we override the default bar colouring by setting it explicitly in line 30 with [maFillOpacity](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#maFillOpacity), providing a number between 0-1 to indicate an opacity between completely transparent (0) and completely opaque (1).

Is this an improved design? The answer may depend on the context in which the graphic appears. For example, we currently have no title for the chart or any indication of the how far each value is above or below 50%. But as a design exercise, it can be useful at least to consider Tufte's minimalist design approach.

## 4. Visualization Algebra

Our fourth and final design framework is a more formal algebraic one that was proposed by [Kindlmann and Scheidegger (2014)](#references). You may recall the 7 transformation steps that make up Wilkinson's _Grammar of Graphics_. We can summarise the model by saying it involves two sets of transformations: _r_ : from the data (D) to its representation on computer (R); and _v_: from its computer representation (R) to its visual expression (V):

![Grammar of graphics as two transformations](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/grammarAlgebra.png)

Kindlmann and Scheidegger go on to represent these transformations as a _commutative diagram_:

![Commutative diagram](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/algebra1.png)

To understand it, imagine you have some data **D** (shown in the top-left corner). The data are represented in a computer as **R** (top-centre) and then with a visualization specification, transformed into a visualization **V** (top-right). Now imagine we start with a different dataset **D** (bottom-left). The theoretical transformation that exists to turn the first dataset into this new one we will call **α** (Greek letter alpha). Again, we can represent this new dataset on computer (**R** bottom-centre) and transform it into a visualization **V** (bottom-right).

Two different datasets have resulted in two different visualizations. The theoretical transformation of one visualization to the other we will symbolise with **ω** (Greek letter omega). The data transformation **α** and visualization transformation **ω** give us the basis for considering three design principles that avoid some common data visualization problems.

### Principle of Representation Invariance

Suppose we make **α** an _identity transform_ (in other words a transform that results in no change). This effectively collapses the two values of **D** into one, meaning that we start with a single dataset but potentially have two different representations of it (for example, a JSON file and and CSV file):

![Commutative diagram](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/algebra2.png)

A good representation and visualization transformation should result in the two visualizations **V** showing the same patterns (they are both showing the same data and both involve the same visualization transformation _v_). That is, that **ω** should also be an identity transform. If it is not, we have a problem because it would be the (arbitrary) _representation_ of the data that results in a different visualization, not the actual data. Kindlmann and Scheidegger refer to visualizations with this problem as containing _hallucinators_ (visual characteristics that are not 'real').

### Principle of Unambiguous Data Depiction

Alternatively we can imagine a scenario where we have two different datasets (the upper and lower **D**) that both result in the same visualization **V**:

![Commutative diagram](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/algebra3.png)

Here we also have a problem in that meaningful changes in the data are not reflected in meaningful perceived variation in the visualization. Kindlmann and Scheidegger refer to visualizations with this problem as containing _confusers_ (the visualization muddles two different datasets).

In class, we will consider examples of common confusers in data visualization.

### Principle of Visual-Data Correspondence

Finally, we can consider cases where both the data and the visualization change:

![Commutative diagram](https://staff.city.ac.uk/~jwo/datavis2021b/session02/images/algebra4.png)

Here we would expect that a change in the data (**α**) should result in a corresponding change in the visualization ( **ω**). Reading changes in the visualization should allow us to infer the nature of the changes in the data that caused them. If a known meaningful change in data does not map onto an equivalent change in the visualization the design contains _jumblers_ and if a perceived change in the visualization is a result of inconsequential or more complex variations in the data, the visualization design contains _misleaders_.

In class, we will consider examples of jumblers and misleaders in poor visual-data correspondence.

### Using a narrative schema to support design with visualization algebra

You can see an example of how this approach can support the design process by considering this [EU referendum mapping litvis document](https://staff.city.ac.uk/~jwo/datavis2021b/session02/brexitMapping.md). Note the frontmatter of the document links to an `algebra.yaml` schema that forces us to consider Kindlmann and Scheidegger's three principles.

{(infobox|}

McNutt (2021) used visualization algebra to [evaluate the effectiveness of novel visualization designs](https://arxiv.org/pdf/2104.04042.pdf). Those looking to make substantial use of data visualization in your BSc Project next term might wish to consider this approach to evaluation.

{|infobox)}

## 5. Conclusions

We have considered four contrasting approaches to data visualization design. Each reflecting a different perspective of the design process. There is no single "best" approach to design; what works may depend on the aims and context of the visualization as well as your own experience and expertise.

Corum's approach as a data journalist emphasises the elements of the design process that support _explanation_. It builds on principles and processes including as data-driven design; sketching, and iterative design.

Data Humanism offers a perspective that emphasises the _human and personal context_, not just in visual aesthetics but also in the construction and framing of the visualization.

Tufte's approach to design is grounded in several core principles that we might summarise as a _minimalist perspective_. That is that extraneous visual detail is removed leaving an emphasis on direct data representation to support comparison.

And finally, Kindlmann and Scheidegger's visualization algebra takes a more formal approach to consider the mapping between data and visualization in order to avoid common ambiguities in that mapping.

When proceeding with your own data visualization designs, you should reflect on each of these approaches (and others) to consider how they may help you create more effective visualizations that match the tasks which they address.

## 6. Recommended Reading

**Kirk, A.** (2019) Chapter 2: _The Visualisation Design Process_, pp.31-58 and Chapter 3: _Formulating Your Brief_, pp.61-94 in [Data Visualisation: A Handbook for Data Driven Design, 2nd Edition](https://go.exlibris.link/rwjXvTCv), Sage.

**Munzner, T.** (2015) Chapter 3: _Task Abstraction_, pp.42-64 in [Visualization Analysis and Design](https://go.exlibris.link/9jMy6fQG), CRC Press

**Tufte, E.** (2001) [The Visual Display of Quantitative Information](https://go.exlibris.link/HRpwwyBl), Graphics Press.

### References

**Borkin, M., Vo, A., Bylinskii, Z., Isola, P., Sunkavalli, S., Oliva, A. and Pfister, H.** (2013) [What makes a visualization memorable?](https://vcg.seas.harvard.edu/publications/what-makes-a-visualization-memorable/paper) _IEEE Transactions on Visualization and Computer Graphics_, 19(12), pp.2306-2315.

**D'Ignazio, C. and Klein, L.** (2020) [Data Feminism](https://data-feminism.mitpress.mit.edu)._MIT Press_

**Kindlmann, G. and Scheidegger, C.** (2014) [An algebraic process for visualization design](http://vis.cs.ucdavis.edu/vis2014papers/TVCG/papers/2181_20tvcg12-kindlmann-2346325.pdf) _IEEE transactions on Visualization and Computer Graphics_ 20(12), pp.2181-2190.

**McNutt, A.** (2021) [What are table cartograms good for anyway? An algebraic analysis](https://arxiv.org/pdf/2104.04042.pdf) _Computer Graphics Forum_ 40(3), pp.61-73

**Wood, J., Isenberg, P., Isenberg, T., Dykes, J., Boukhelifa, N. and Slingsby, A.** (2012) [Sketchy rendering for information visualization](http://openaccess.city.ac.uk/1274/) _IEEE Transactions on Visualization and Computer Graphics_, 18(12), pp.2749-2758.

## 7. Practical Exercises

{(task|} Please complete the following exercises before next week's session. They should be added to your session02 folder of your portfolio. Remember to stage, commit and push your answers to github so I can see them. {|task)}

### 1. Evaluating Design Approaches

![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg)

Make sure you have completed all the annotations in response to the tasks above in these lecture notes (e.g. questions on Corum's talk, Weber's scarf etc.). For your continuous assessment, copy your annotations (or a summary of them) into `practicalExercises.md` in session02 of your portfolio folder.

### 2. Creating a Minimalist Tufte Visualization

Create a litvis document called `imdbRoot.md` that contains the following:

````markdown
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
````

Create a second document called `defaultHistogram.md` that contains the following simple histogram specification:

````markdown
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
                << position Y [ pAggregate opCount ]
                << color [ mName "IMDB Rating", mOrdinal ]
    in
    toVegaLite [ data, enc [], bar [] ]
```
````

Finally, create a third document that also `follows: imdbRoot`, but this time containing a modified version of the histogram that follows some of Tufte's minimalist design principles.

_Note:_ This example creates a frequency histogram from a single data column (`IMDB Rating`) using the elm-vegalite functions [pBin](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pBin) and [pAggregate opCount](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pAggregate). You won't need to change those aspects of the specification, but you should be able to change others such as axes, legends, colour encoding etc.

Add a brief evaluation to this third litvis document that assesses which elements of the minimalist design work and which do not.

### 3. _Creativity Challenge:_ Hand Drawn Personal Visualization Sketch

![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg)

Following the principles of data humanism, try logging some personal data (can be anything ranging from time spent outside, to a food diary, to communications with others) and sketching a hand drawn design. Take a photo of it and embed it in your `practicalExercises.md` litvis document (in session02 of your portfolio) with a brief explanation of what you have visualized and a brief reflection on the process of constructing and viewing the visualization.

Share your sketch by posting a photo of it and brief explanation on Slack.

---

_Check your progress._

- [ ] I can recognise and apply the four contrasting approaches to a visualization design.
- [ ] I can distinguish different visualization objectives such as explanation, reflection, art and analysis.
- [ ] I have created litvis documents with branching narratives to explore alternative design options.
- [ ] I have specified a Vega-Lite visualization that follows some of Tufte's design principles.
- [ ] I can identify hallucinators, distractors, jumblers and misleaders in a visualization.
- [ ] I have created a hand-drawn sketch of some personal data visualization.
