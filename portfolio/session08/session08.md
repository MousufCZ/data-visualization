---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 8: Geographic Layout

## Table of Contents

1.  [Introduction](#1-introduction)
2.  [GeoJSON and TopoJSON](#2-geojson-and-topojson)
3.  [Working with Multiple Data Sources](#3-working-with-multiple-data-sources)
4.  [Map Projections](#4-map-projections)
5.  [Relaxed spatial Layout](#5-relaxed-spatial-layout)
6.  [Conclusions](#6-conclusions)
7.  [Recommended Reading](#7-recommended-reading)
8.  [Practical Exercises](#8-practical-exercises)

{(aim|}

This session is designed to consider how positional encoding can be used to represent geographic data. It considers some of the properties of geographic data that require specific treatment when designing and implementing geovisualization.

By the end of this session you should be able to:

- recognise the basic structure of geoJSON and topoJSON data files.
- exploit the spatial autocorrelation of many geographic phenomena in your visualization design.
- use [mapshaper](https://mapshaper.org) to simplify and transform geospatial data.
- combine geospatial boundary data with attribute data in your data visualizations.
- select an appropriate global map projection depending on the region and purpose of your geovisualization.
- use grid mapping to arrange data visualizations in a relaxed geographic configuration

{|aim)}

---

## 1. Introduction

We have seen in previous sessions how important _location_ is as a visual variable, both in the placement of individual marks within a visualization view, and in the arrangement of multiple views in more complex compositions. In this session we continue to explore the role of location, but specifically in the context of _geographic data_, where we are more familiar with the correspondence between graphic location and geographic location, largely in the context of cartography.

There are a few reasons why geographic layout deserves special treatment and by way of an introduction, we will briefly consider three of them:

**Spatial autocorrelation:** Geographic phenomena often have a characteristic quality that makes them amenable to visualization. This was summarised by [Waldo Tobler](#references) in his assertion that has become known as the "first law of geography" (somewhat analogous to the second law of thermodynamics):

> Everything is related to everything else, but near things are more related than distant things

What this means in practice is that many of the geographic phenomena we may wish to visualize are likely to exhibit structures or clusters of more similar features. If we encode the quality of those features with position we are likely to be able to detect them visually.

![Contoured surface](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/contouredSurface.png)
_An elevation relief map. Contours are only possible because altitude is spatially autocorrelated (elevation at any point is more likely to be more similar to a point close by than to a distant one)._

{(infobox|}While the focus in this session is inevitably on how to visualize geographic data, it is worth noting that Tobler's "first law" can also be measured numerically by calculating [spatial autocorrelation](https://gisgeography.com/tobler-first-law-of-geography/) (a numeric measurement that captures the relationship between attribute similarity and spatial similarity) {|infobox)}

**Geographical coordinates and map projection:** The second reason we might single out geographic data for special treatment is the way we reference location. So far, we have seen how we can encode any two data fields with `position X` and `position Y` in order to show them graphically on a 2d plane. We have already seen examples of doing this with geographical location where we encode _longitude_ with `position X` and _latitude_ with `position Y`. However, the two angles longitude and latitude don't reference a 2d location, but a _vector_ in 3d polar coordinates. We need to _project_ that vector onto a 2d plane before we can visualize it. As we shall see, there are very many ways in which we can do that, depending on which geographic properties we wish to prioritise in our visualization.

```elm {l=hidden}
graticuleSpec =
    asSpec
        [ graticule [ grStep ( 15, 15 ) ]
        , geoshape [ maFilled False, maStroke "black", maStrokeWidth 0.1 ]
        ]


countrySpec =
    asSpec
        [ dataFromUrl "https://gicentre.github.io/data/geoTutorials/world-110m.json"
            [ topojsonFeature "countries1" ]
        , geoshape [ maFill "black", maFillOpacity 0.1 ]
        ]


noBorderCfg =
    configure
        << configuration (coView [ vicoStroke Nothing ])
```

```elm {v}
projectionProblem : Spec
projectionProblem =
    let
        globeSpec =
            asSpec
                [ projection [ prType orthographic, prRotate -25 -20 0 ]
                , layer [ graticuleSpec, countrySpec ]
                ]

        rectSpec =
            asSpec
                [ width 400
                , projection [ prType equirectangular ]
                , layer [ graticuleSpec, countrySpec ]
                ]
    in
    toVegaLite
        [ noBorderCfg []
        , hConcat [ globeSpec, rectSpec ]
        ]
```

_Projection of 3d global geographic coordinates onto a 2d plane._

**Complex geometry:** Thirdly, we can recognise that the geometry of individual geographic features can be more complex than the simple shapes we have used so far in our visualizations (circles, bars, triangles etc.). While we might represent individual locations as point symbols (as you did with the 'Bad Teeth' example in Session 5), spatial data may also be represented as lines or polygons. Those polygons may be "complex" in sense of containing collections of disjoint (unconnected) polygons and 'holes'.

![Geometry types](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/geometryTypes.png)
_point, line, polygon and complex polygon geometric features_.

And sometimes, those holes can contain 'islands', that themselves contain holes, which...

![Nested islands ](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/nestedIslands.jpg)
_[Vulcan Point Island, Philippines](https://goo.gl/maps/V6ouZ2yULVs) as an example of complex ring geometry_

## 2. GeoJSON and TopoJSON

To store the complex geometry necessary to represent geographic features we need more than the simple pairs of numeric values we have so far used to represent the position of a mark. Vega-Lite, as well as many modern systems for handling geospatial data, use [geoJSON](http://geojson.org) and [topoJSON](https://github.com/topojson/topojson) formats to capture the geometry of a feature's boundary as well as (optionally) some of the attributes associated with the feature.

As an example, here is how we might specify a base-map of London boroughs by reading a topoJSON file containing the geometry of borough boundaries and the names of each borough:

```elm {l v highlight=[4-6,,8-9,14-16]}
londonMap : Spec
londonMap =
    let
        londonGeodata =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        proj =
            projection [ prType transverseMercator, prRotate 2 0 0 ]
    in
    toVegaLite
        [ width 500
        , height 400
        , londonGeodata
        , proj
        , geoshape [ maFill "lightgrey", maStroke "white" ]
        ]
```

This example happens to specify the _map projection_ (lines 8 and 9) of the geographic coordinates, but more on this later.

Note that there is no position encoding in the way we have specified previously (`position X` etc.). Instead, all the positional information is stored in `londonBoroughs.json`. We use the [geoshape](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#geoshape) mark instead of simple circles, bars etc. That handles the drawing of each `boroughs` feature stored in file referenced by `londonGeodata`.

To understand what precisely is stored in one of these geo files, let's consider some simpler examples:

### A simple GeoJSON file

Suppose we wished to show a simple rectangular region of the earth's surface:

```elm {v}
globeRect : Spec
globeRect =
    let
        rectSpec =
            asSpec
                [ dataFromUrl "https://gicentre.github.io/data/geoTutorials/geoJson1.json" []
                , projection [ prType orthographic, prRotate 30 -25 0 ]
                , geoshape [ maStroke "#00a2f3", maFill "#00a2f3", maFillOpacity 0.3 ]
                ]
    in
    toVegaLite
        [ configure (configuration (coView [ vicoStroke Nothing ]) [])
        , layer [ graticuleSpec, countrySpec, rectSpec ]
        ]
```

The geoJSON format to represent the blue rectangle looks like this:

```Javascript
{
  "type": "Feature",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [ [-3, 52], [4, 52], [4, 45], [-3, 45], [-3, 52] ]
    ]
  }
}
```

Each pair of numbers is in _longitude,latitude_ order and can be used to represent a location on the earth's surface. Longitude is scaled between -180 degrees and 180 degrees centred at 0 through the Greenwich meridian; latitude between -90 (south pole) and 90 (north pole), centred on the equator.

The list of pairs provides an ordered sequence of point locations that, when defining a `Polygon`, bound a feature's extent. Note the matching of the first and last coordinates to 'close' the polygon.

Now suppose we wished to store more than one feature:

```elm {v}
twoFeatures : Spec
twoFeatures =
    toVegaLite
        [ width 200
        , height 200
        , dataFromUrl "https://gicentre.github.io/data/geoTutorials/topoJson2.json"
            [ topojsonFeature "myRegions" ]
        , projection [ prType orthographic ]
        , geoshape [ maStroke "#00a2f3", maFill "#00a2f3", maFillOpacity 0.5 ]
        ]
```

Here is a geoJSON representation of this pair of features:

```Javascript
{
  "type": "FeatureCollection",
  "features": [{
      "type": "Feature",
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [ [-3, 52], [4, 52], [4, 45], [-3, 45], [-3, 52] ]
        ]
      }
    },
    {
      "type": "Feature",
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [ [-3, 59], [4, 59], [4, 52], [-3, 52], [-3, 59] ]
        ]
      }
    }
  ]
}
```

The geoJSON now contains two `Polygon` features. Both are elements of an outer `features` array which in this case contains two elements but could contain many more. Notice how the common edge between these two adjacent regions is duplicated (the boundary between (-3,52) and (4,52)). In this case that duplication isn't a big problem, but imagine that boundary was a complex meandering line with many hundreds of coordinate pairs. To avoid unnecessary duplication, we would need a more efficient alternative...

### A Simple TopoJSON file

The topoJSON format stores all boundaries as distinct _arcs_ (edge pathways) that are then reassembled to form feature boundaries. If more than one feature shares a common arc, it is only stored once. So the geoJSON file above could be represented as the following topoJSON file:

```javascript
{
  "type": "topology",
  "objects": {
    "myRegions": {
      "type": "GeometryCollection",
      "geometries": [{
        "type": "Polygon",
        "arcs": [ [0, 1] ]
      }, {
        "type": "Polygon",
        "arcs": [ [-1, 2] ]
      }]
    }
  },
  "arcs": [
    [ [-3, 52], [4, 52] ],
    [ [ 4, 52], [ 4, 45], [-3, 45], [-3, 52] ],
    [ [-3, 52], [-3, 59], [ 4, 59], [ 4, 52] ]
  ]
}
```

The arcs are referenced by their array position (0, 1 and 2 in this example) to form the two polygons.

{(task|}Why do you think arc 1 is referenced by `-1` in the second polygon? Try drawing out the vertices, boundary lines and regions if you are not sure. You may also need to consult the [topoJSON format specification](https://github.com/topojson/topojson-specification#214-arc-indexes) (Hint: "ones' complement").{|task)}

So far, we have only considered the geometry of features. Each feature can additionally have an `id` that stores some property to be associated with the feature, for example a country name, or population count. Keeping with our simple two-region example, this is how an `id` can be associated with each feature:

```Javascript {highlight=[9,13]}
{
  "type": "topology",
  "objects": {
    "myRegions": {
      "type": "GeometryCollection",
      "geometries": [{
        "type": "Polygon",
        "arcs": [ [0, 1] ],
        "id" : "southern region"
      }, {
        "type": "Polygon",
        "arcs": [ [-1, 2] ],
        "id" : "northern region"
      }]
    }
  },
  "arcs": [
    [ [-3, 52], [4, 52] ],
    [ [ 4, 52], [ 4, 45], [-3, 45], [-3, 52] ],
    [ [-3, 52], [-3, 59], [ 4, 59], [ 4, 52] ]
  ]
}
```

We can now encode the value associated with each `id` with colour in a visualization specification:

```elm {l v highlight=[5,6,10]}
featuresWithID : Spec
featuresWithID =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/topoJson3.json"
                [ topojsonFeature "myRegions" ]

        enc =
            encoding
                << color [ mName "id", mNominal ]

        proj =
            projection [ prType orthographic ]
    in
    toVegaLite [ data, proj, enc [], geoshape [] ]
```

The `id` is a concise way of identifying a single property in a topoJSON file, but only one `id` is permitted for each feature. TopoJSON and GeoJSON files that need to store multiple attributes for each feature may additionally store `properties` objects that can have any number of JSON objects associated with them. Here is an example of our two-region topoJSON file where each feature contains no `id` but instead the properties `myRegionName` and `myPopulationCount`:

```Javascript {highlight=[8-11,15-18}
{
  "type": "topology",
  "objects": {
    "myRegions": {
      "type": "GeometryCollection",
      "geometries": [{
        "type": "Polygon",
        "properties": {
            "myRegionName": "southern region",
            "myPopulationCount": 27000,
          },
        "arcs": [ [0, 1] ]
      }, {
        "type": "Polygon",
        "properties": {
            "myRegionName": "northern region",
            "myPopulationCount": 18000,
          },
        "arcs": [ [-1, 2] ]
      }]
    }
  },
  "arcs": [
    [ [-3, 52], [4, 52] ],
    [ [ 4, 52], [ 4, 45], [-3, 45], [-3, 52] ],
    [ [-3, 52], [-3, 59], [ 4, 59], [ 4, 52] ]
  ]
}
```

We can access any of the properties in a visualization specification by using dot notation to identify nested JSON elements. For example, instead of `id`, to access `myRegionName` for each feature we would use `properties.myRegionName` or, as below, `properties.myPopulationCount` to colour each feature by its population count:

```elm {l v}
geoFeatureExample : Spec
geoFeatureExample =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/topoJson4.json"
                [ topojsonFeature "myRegions" ]

        enc =
            encoding
                << color [ mName "properties.myPopulationCount", mQuant ]

        proj =
            projection [ prType orthographic ]
    in
    toVegaLite [ data, proj, enc [], geoshape [] ]
```

Note that if the property name contains spaces or other non-alphanumeric characters, you need to use JavaScript's array syntax instead of dot syntax (e.g. `properties['some name']` rather than `properties.some name`).

{(infobox|}

If you would like to find out more about working with geoJSON and topoJSON, including creating specifications programmatically and using 'command-line cartography' to process geospatial data, see these tutorials (view the litvis .md files in VSCode)

- [Geospatial File Formats](https://raw.githubusercontent.com/gicentre/litvis/main/documents/tutorials/geoTutorials/geoFormats.md)
- [Generating Global Map Projection Geo Files](https://raw.githubusercontent.com/gicentre/litvis/main/documents/tutorials/geoTutorials/geoGenerating.md)
- [Importing geographic datasets](https://raw.githubusercontent.com/gicentre/litvis/main/documents/tutorials/geoTutorials/geoImporting.md)

{|infobox)}

### geo/topo JSON data sources

To speed up the creation of some of the more common spatial units (from a UK/US perspective), you can use the following topoJSON files directly in your visualizations:

| Spatial units                   | Source                                                                                  | feature          | id                    | other properties / files                                                                  |
| ------------------------------- | --------------------------------------------------------------------------------------- | ---------------- | --------------------- | ----------------------------------------------------------------------------------------- |
| World countries                 | [world-110m.json](https://gicentre.github.io/data/geoTutorials/world-110m.json)         | `countries1`     | 3-letter country code | `name`, `Alpha-2`                                                                         |
| US states                       | [us-10m.json](https://vega.github.io/vega-lite/data/us-10m.json)                        | `states`         | state id              |                                                                                           |
| US counties                     | [us-10m.json](https://vega.github.io/vega-lite/data/us-10m.json)                        | `counties`       | county id             |                                                                                           |
| UK parliamentary constituencies | [ukConstituencies.json](https://gicentre.github.io/data/uk/ukConstituencies.json)       | `constituencies` | constituency id       | [constituencyCentroids.csv](https://gicentre.github.io/data/uk/constituencyCentroids.csv) |
| London boroughs                 | [londonBoroughs.json](https://gicentre.github.io/data/geoTutorials/londonBoroughs.json) | `boroughs`       | borough name          | [londonCentroids.csv](https://gicentre.github.io/data/geoTutorials/londonCentroids.csv)   |

But in many cases you may wish to create visualizations based on alternative geographies. You may find data already in topoJSON format, but it is common to have to convert the data yourself. This usually involves the following stages:

1.  Find a geographic boundary file.
2.  Convert coordinates to longitude/latitude
3.  Simplify boundary geometry to speed up rendering time
4.  Save file locally in topoJSON format

Many _Geographic Information Systems (GIS)_ such as [QGIS](https://www.qgis.org/en/site/) can be used to achieve these steps, but these are major pieces of software and using them is not simple. Instead we can use the online tool [mapshaper](https://mapshaper.org), which greatly simplifies some of the more common data shaping steps.

For example, suppose we wished to create a map using the standard GB regions (Southwest, Southeast, West Midlands etc.). The boundaries are available from the [ONS geodata portal](https://geoportal.statistics.gov.uk/datasets?q=Electoral%20Boundaries&sort=name) but they are only downloadable as _shapefiles_ using Ordnance Survey National Grid coordinates. Using mapshaper, we could do the following:

1.  [Download the boundaries](https://geoportal.statistics.gov.uk/datasets/european-electoral-regions-december-2016-full-clipped-boundaries-in-great-britain) as a _shapefile_ collection and drag the downloaded files into [mapshaper](https://mapshaper.org)
2.  Open the mapshaper console and type `mapshaper -proj wgs84` to convert the data into global longitude/latitude coordinates.
3.  Click _Simplify_ and drag the _settings_ slider to the right to simplify as much as possible while still producing an acceptable boundary file. You may have to select _repair_ if simplification breaks any boundary lines.
4.  Click _Export_ and select the _TopoJSON_ file format. Move the downloaded file this creates to the same folder as your markdown document or place it in your github data repo for remote access.

![mapshaper conversion](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/mapshaper.png)
_Stages 1 – 3 of geospatial file conversion in Mapshaper_

You should now be able to display the map data in your own visualization specification.

```elm {l v}
regionMap : Spec
regionMap =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/uk/gbRegions.json"
                [ topojsonFeature "European_Electoral_Regions_December_2016_Full_Clipped_Boundaries_in_Great_Britain" ]
    in
    toVegaLite
        [ height 400
        , width 300
        , geoData
        , geoshape [ maFill "lightgrey", maStroke "white" ]
        ]
```

(_this example uses an external URL, but if you have followed the steps above, you should be able to replace `https://gicentre.github.io/data/uk/gbRegions.json` with the name of your converted file_).

To help with the creating your own topoJSON map files in this way, I have created a [geo/topoJSON viewer](https://observablehq.com/@jwolondon/spatial-json-viewer) which you can use to identify the name of the `topojsonFeature` (the 'feature' column in the table above) and the attributes available within the file (used to link with other files as shown in Section 3 below).

{(infobox|}

If you would like to see a richer set of examples of manipulating data with [mapShaper](https://github.com/mbloch/mapshaper) and litvis, you can look at my [30 day map challenge examples](https://github.com/jwoLondon/30dayMapChallenge). Most involve some pre-processing of geospatial data with mapshaper before visualizing in litvis.

![30 day map challenge](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/30dayMapChallengeBanner.jpg)

{|infobox)}

## 3. Working with Multiple Data Sources

In the examples above, the topoJSON file contained both the geometry of feature boundaries and properties (or "attributes") associated with each feature. However, in many data visualizations the boundaries of map features may come from a different data source to the attributes we wish to match them with. In such cases we need to _join_ the two data sources.

For example, suppose we have a [Tidy data table](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#Table) on the percentage of households owning a car or van for each of the London Boroughs ([source](https://www.racfoundation.org/assets/rac_foundation/content/downloadables/car%20ownership%20rates%20by%20local%20authority%20-%20december%202012.pdf)):

```elm {l=hidden}
carOwnershipTable : Table
carOwnershipTable =
    """borough, carOwnership
       Barking and Dagenham,60.4
       Barnet,71.3
       Bexley,76.3
       Brent,57.0
       Bromley,76.5
       Camden,38.9
       City of London,30.6
       Croydon,66.5
       Ealing,64.7
       Enfield,67.5
       Greenwich,58.0
       Hackney, 35.4
       Hammersmith and Fulham,44.8
       Haringey,48.2
       Harrow,76.5
       Havering,77.0
       Hillingdon,77.3
       Hounslow,68.4
       Islington,35.3
       Kensington and Chelsea,44.0
       Kingston upon Thames,74.9
       Lambeth,42.2
       Lewisham,51.9
       Merton,67.4
       Newham,47.9
       Redbridge,72.1
       Richmond upon Thames,75.3
       Southwark,41.6
       Sutton,76.6
       Tower Hamlets,37.0
       Waltham Forest,58.1
       Wandsworth,54.7
       Westminster,37.1"""
        |> fromCSV
```

^^^elm{m=(tableSummary 6 carOwnershipTable)}^^^

We can join this table with the geo data in the topoJSON file as long as both have some common key that links rows in the two tables, which in this case will be the borough name. Once joined we can colour each borough according to the car ownership value to create a _choropleth map_:

```elm {interactive l v highlight=15}
londonChoropleth : Spec
londonChoropleth =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        carData =
            dataFromColumns []
                << dataColumn "borough" (strColumn "borough" carOwnershipTable |> strs)
                << dataColumn "carOwnership" (numColumn "carOwnership" carOwnershipTable |> nums)

        trans =
            transform
                << lookup "id" (carData []) "borough" (luFields [ "carOwnership" ])

        enc =
            encoding
                << color
                    [ mName "carOwnership"
                    , mQuant
                    , mTitle "% households with car"
                    ]
                << tooltips [ [ tName "id" ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset 0 ])
    in
    toVegaLite
        [ width 600
        , height 450
        , background "grey"
        , cfg []
        , geoData
        , trans []
        , enc []
        , geoshape [ maStroke "white" ]
        ]
```

> Note the spatial autocorrelation of car ownership here - there is a clear association with distance from the centre of London so that close-by boroughs are more likely to show similar car ownership patterns than distant ones.

The function to join the topoJSON data containing the borough boundaries with the table of car ownership data is [lookup](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookup), which is this example is saying add the `"carOwnership"` column from `carData` to the primary dataset (`geoData`) where `borough` in `carData` is equal to `id` in the primary dataset. It is the equivalent of [leftJoin](https://package.elm-lang.org/packages/gicentre/tidy/latest/Tidy#leftJoin) we considered in Session 5.

![lookup anatomy](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/lookup.png)

You are likely to find yourself using this function whenever you need to join separate attribute data to your geospatial topo/geoJSON data.

While choropleth maps like the one above are commonplace and can be informative, they do not work well when the land area of each feature (boroughs in the example above) do not correspond with the numbers of people occupying them.

When a large area does not equal a large number of values, we are potentially creating a misleading map, with our attention drawn away from the most important areas of the map. A notable example of this problem is the choropleth map of US presidential election results favoured by the 2016 winning US president (and the losing candidate in 2020):

![US election choropleth](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/usElections.jpg)
_Left: Choropleth map of 2016 election results being installed in the White House (photo: Trey Yingst); Right: The same data shown as a proportional circle map (New York Times, 2016)_

So how might we create a better alternative for cases where there are large discrepancies between land area and population?

Following the New York Times example above, we could symbolise data magnitude with some symbol sized in proportion to value rather than land area. The problem we have with using the topoJSON boundary files to do this is that the geometry of each region specifies a bounding polygon, not a point location. So we need to convert each region into a point, most commonly by calculating its _centroid_. Doing so programmatically isn't as straightforward as taking the average of all boundary vertices, but helpfully, _mapshaper_ can do the calculations for us.

![centroid calculation](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/centroids.png)
_Centroids of (a) simple polygon; (b) complex polygon; (c) population weighted region. Dashed circle indicates (incorrect) value derived by taking mean coordinates of polygon vertices_

You can use the [geo/topoJSON viewer](https://observablehq.com/@jwolondon/spatial-json-viewer) to generate centroids from any geo or topo JSON file (click the `Save centroids as CSV` button).

Alternatively, you can use [mapshaper](https://mapshaper.org) to do this as follows:

1.  Drag the topoJSON file containing polygons into mapshaper.
2.  In the mapshaper console, type `mapshaper -each 'cx=this.centroidX, cy=this.centroidY' -o 'centroids.csv'`
3.  Move the downloaded centroids file to the same location as your litvis document.

In the case of our London boroughs example, this generates a file whose first few lines look like this, where `FID` is short for 'feature id':

```csv
FID,cx,cy
Kingston upon Thames,-0.28683993623482656,51.38778201868287
Croydon,-0.0871497395734254,51.3554033548877
Bromley,0.05158107231454641,51.37200567800018
Hounslow,-0.36712638707752276,51.46840118781318
:
:
```

{(infobox|}Mapshaper can do a lot more besides simple file conversion and line simplification. See the [Mapshaper command-line reference](https://github.com/mbloch/mapshaper/wiki/Command-Reference) for further details.{|infobox)}

Once we have a data file containing the centroid locations, we can join them to our main data table just as we did for the choropleth map (note in the example below, the centroid file is referenced with an absolute URL `https://gicentre.github.io/data/geoTutorials/londonCentroids.csv`, but if you have used mapshaper to create a local copy of the centroids file you should be able to simply use `centroids.csv`).

```elm {l v interactive highlight=[13-14,19-21,37]}
londonCircles : Spec
londonCircles =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        carData =
            dataFromColumns []
                << dataColumn "borough" (strColumn "borough" carOwnershipTable |> strs)
                << dataColumn "carOwnership" (numColumn "carOwnership" carOwnershipTable |> nums)

        centroidData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonCentroids.csv"

        backgroundSpec =
            asSpec [ geoData, geoshape [ maFill "#ddd", maStroke "white" ] ]

        trans =
            transform
                << lookup "FID" (carData []) "borough" (luFields [ "carOwnership" ])

        enc =
            encoding
                << position Longitude [ pName "cx" ]
                << position Latitude [ pName "cy" ]
                << size
                    [ mName "carOwnership"
                    , mScale [ scRange (raNums [ 0, 1000 ]), scType scPow, scExponent (1 / 0.86) ]
                    ]
                << tooltips
                    [ [ tName "FID" ]
                    , [ tName "carOwnership" ]
                    ]

        circleSpec =
            asSpec [ centroidData [], trans [], enc [], circle [ maOpacity 0.5 ] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coLegend [ lecoOrient loBottomRight, lecoOffset 0 ])
    in
    toVegaLite
        [ width 640, height 480, cfg [], layer [ backgroundSpec, circleSpec ] ]
```

{(task|}Can you modify the example above to add a new layer displaying the borough name as text?
You will probably want to offset the text labels using [maDy](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#maDy) so they do not overlap with the proportional symbols.{|task)}

A useful summary of good practice in map design that, among other issues, considers the choropleth vs proportional symbol mapping, is given by Ken Field's blog on [Mapping Coronavirus, Responsibly](https://www.esri.com/arcgis-blog/products/product/mapping/mapping-coronavirus-responsibly/). It includes (with justification) this rule of thumb:

> There are very very few golden rules in cartography but this is one of them: you cannot map totals using a choropleth thematic mapping technique.

{(task|}Why does Field suggest you should not show totals (i.e. absolute counts of things) in a choropleth map? Are there ever any exceptions to this rule?{|task)}

## 4. Map Projections

So far, we have treated position encoding of geographical coordinates in much the same way as we have for any other pair of quantitative values. The only difference being that instead of `position X` and `position Y` we have used `position Longitude` and `position Latitude`. But it is important to realise that longitude and latitude are quite different to Cartesian _x_ and _y_ coordinates, in that that they represent a pair of _angles_ not distances:

A _(longitude, latitude)_ pair therefore defines an infinitely extending _vector_ originating from the earth's centre. To represent a geographic location, we need to consider the intersection of that vector with the surface of the earth. And to see it graphically, we need to project that 3d location onto a 2d plane.

![Graticule](https://staff.city.ac.uk/~jwo/datavis2021b/session08/images/geoCoords.jpg)

By default, Vega-Lite will project all _(longitude, latitude)_ pairs onto a plane using an [Equal Earth projection](https://en.wikipedia.org/wiki/Equal_Earth_projection) and assuming a particular model of the earth's surface known as [WGS84](https://en.wikipedia.org/wiki/World_Geodetic_System#A_new_World_Geodetic_System:_WGS_84). For data visualization involving small regions of the earth's surface, such as the London maps shown above, the choice of projection is often not critical as its 3d and 2d projected surfaces will be very similar.

But for global-scale maps, or for those where geospatial accuracy is a priority, the choice of map projection becomes more important. The following example shows how we can specify a map projection in Vega-Lite:

```elm {l v highlight=[8,9,15]}
globalMap : Spec
globalMap =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/world-110m.json"
                [ topojsonFeature "countries1" ]

        proj =
            projection [ prType equirectangular, prRotate -156 0 0 ]
    in
    toVegaLite
        [ width 640
        , height 320
        , geoData
        , proj
        , geoshape [ maFill "lightgrey" ]
        ]
```

The two parameters you are most likely to want to specify in your own geospatial visualizations will be the [prType](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#prType) and possibly the projection rotation (via [prRotate](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#prRotate)). To see the effect of changing these parameters try changing the selections below.

```elm {v interactive}
projections : Spec
projections =
    let
        geoData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/world-110m.json"
                [ topojsonFeature "countries1" ]

        ps =
            params
                << param "projection"
                    [ paValue (str "EqualEarth")
                    , paBind
                        (ipSelect
                            [ inName "prType "
                            , inOptions
                                [ "Albers"
                                , "AlbersUsa"
                                , "AzimuthalEqualArea"
                                , "AzimuthalEquidistant"
                                , "ConicConformal"
                                , "ConicEqualArea"
                                , "ConicEquidistant"
                                , "EqualEarth"
                                , "Equirectangular"
                                , "Gnomonic"
                                , "Mercator"
                                , "NaturalEarth1"
                                , "Orthographic"
                                , "Stereographic"
                                , "TransverseMercator"
                                ]
                            ]
                        )
                    ]
                << param "lambda" [ paValue (num 0), paBind (ipRange [ inName "prRotate lambda", inMin -180, inMax 180 ]) ]
                << param "phi" [ paValue (num 0), paBind (ipRange [ inName "prRotate phi", inMin -90, inMax 90 ]) ]
                << param "gamma" [ paValue (num 0), paBind (ipRange [ inName "prRotate gamma", inMin -180, inMax 180 ]) ]

        proj =
            projection
                [ prType (prExpr "projection")
                , prRotateExpr "lambda" "phi" "gamma"
                ]

        sphereSpec =
            asSpec
                [ sphere
                , geoshape [ maFill "rgb(204,206,186)", maStroke "black", maStrokeOpacity 0.3 ]
                ]

        gratSpec =
            asSpec
                [ graticule [ grStep ( 10, 10 ) ]
                , geoshape [ maStroke "grey", maStrokeWidth 0.2 ]
                ]

        landSpec =
            asSpec
                [ geoData
                , geoshape
                    [ maFill "rgb(235,219,181)"
                    , maStroke "rgb(216,148,112)"
                    , maStrokeWidth 0.2
                    ]
                ]
    in
    toVegaLite
        [ width 640
        , height 320
        , ps []
        , proj
        , layer [ sphereSpec, landSpec, gratSpec ]
        ]
```

When projecting from a 3d globe to a 2d surface, it is not possible to avoid some kind of distortion. That might be the area of different features (e.g. `mercator`), or their shape (e.g. gnomonic) or distances between pairs of locations (e.g. `stereographic`). Notice also that the amount of distortion often varies across the map depending on the rotation values. Selecting an appropriate projection therefore involves assessing which spatial properties are most important to preserve and where, given the objective of the visualization. For example, if making a visual assessment of relative size of features is important, selecting an area-preserving projection such as `conicEqualArea` or `albers` might be a priority. If visualization is concerned with Antarctica, selecting a rotation that minimises distortion around the southern latitudes (e.g. `azimuthalEquidistant` with `prRotate 0 90 0` might be more important.

For UK data, it is common to project longitude/latitude data to [Ordnance Survey National Grid (OSGB)](https://en.wikipedia.org/wiki/Ordnance_Survey_National_Grid), which attempts to minimise and balance distortion around Britain. Sometimes data will already be in OSGB (see below), but if not, it can be approximated with `projection [ prType transverseMercator, prRotate 2 0 0 ]`

## 4.1 Pre-Projected Data

To reproject data in Vega-Lite, the geospatial coordinates that represent geographic features must be longitude/latitude values. But in many cases data may already have been projected, for example, as Ordnance Survey National Grid coordinates (see [OS OpenData](https://www.ordnancesurvey.co.uk/opendatadownload/products.html)).

To work with such data in Vega-Lite you need either to (a) 'unproject' the data back to longitude/latitude coordinates, or (b) avoid projecting the data within your specification.

Option (a) was discussed in section 2 above where [mapshaper](https://mapshaper.org) can be used to 'unproject' with `mapshaper -proj wgs84`. But in many cases option (b) may be preferable, especially if you wish to combine with other data also using the same projection. To do this in Vega-Lite we can project using the [identityProjection](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#identityProjection) which leaves the coordinates unaltered. Commonly though we also need to flip the Y-axis so that 'north' is at the top.

Here's an example that uses this approach to create a landuse map of Clerkenwell from data that use Ordnance Survey National Grid coordinates.

```elm {l  highlight=[8,9,58]  v}
clerkenwellModern : Spec
clerkenwellModern =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/30dayMapChallenge/clerkenwell2019Polys.json"
                [ topojsonFeature "clerkenwell" ]

        proj =
            projection [ prType identityProjection, prReflectY True ]

        greenFilter =
            transform
                << filter (fiExpr "datum.properties.leisure == 'garden' || datum.properties.leisure == 'park'")

        greenSpec =
            asSpec
                [ greenFilter []
                , geoshape
                    [ maColor "rgb(211,215,176)"
                    , maStroke "#ab9"
                    , maFillOpacity 1
                    ]
                ]

        landuseFilter =
            transform
                << filter (fiExpr "isValid(datum.properties.landuse) || isValid(datum.properties.amenity)")

        landuseSpec =
            asSpec
                [ landuseFilter []
                , geoshape
                    [ maColor "#bbb"
                    , maStroke "#666"
                    , maStrokeWidth 0.5
                    , maOpacity 0.8
                    ]
                ]

        buildingsFilter =
            transform << filter (fiExpr "isValid(datum.properties.building)")

        buildingsSpec =
            asSpec
                [ buildingsFilter []
                , geoshape
                    [ maColor "rgb(204,151,116)"
                    , maStroke "black"
                    , maStrokeWidth 0.2
                    , maFillOpacity 1
                    ]
                ]
    in
    toVegaLite
        [ width 600
        , height 580
        , data
        , proj
        , layer [ landuseSpec, greenSpec, buildingsSpec ]
        ]
```

## 5. Relaxed spatial Layout

In all the examples we have seen so far, the graphical location of geospatial data has been determined precisely by the geographic location of the entity being depicted. But in many cases, even if we wish to understand spatial patterns in our data such as clusters of similar attributes, a _precise_ geographic location is not required. In such cases we might choose to relax the positional encoding somewhat, freeing up our designs to support other tasks more effectively.

To illustrate the process of relaxing spatial layout, let's first consider how we might use a conventional mapping layout to show patterns in residential dwelling density over space and time. The data are available from the [London data store](https://data.london.gov.uk/dataset/number-and-density-of-dwellings-by-borough) and can be read as the following table:

```elm {l=hidden}
dwellings : Table
dwellings =
    """
GSSCode,dph2001,dph2002,dph2003,dph2004,dph2005,dph2006,dph2007,dph2008,dph2009,dph2010,dph2011,dph2012,dph2013,dph2014,dph2015,dph2016,dph2017,dph2018,dph2019
E09000001,15.9,16.2,16.3,16.7,17.1,17.1,17.1,17.2,17.4,17.3,17.5,17.6,17.7,19.1,19.8,20.0,20.0,20.5,20.7
E09000002,18.1,18.1,18.2,18.2,18.3,18.4,18.5,18.7,18.7,18.8,18.8,18.9,19.0,19.2,19.4,19.6,19.7,19.8,20.1
E09000003,15.0,15.1,15.2,15.3,15.4,15.6,15.6,15.8,15.9,16.0,16.1,16.3,16.5,16.6,16.7,16.9,17.1,17.4,17.6
E09000004,14.2,14.3,14.3,14.5,14.5,14.5,14.6,14.6,14.7,14.7,14.8,14.8,14.9,15.0,15.1,15.1,15.2,15.2,15.3
E09000005,23.5,23.8,24.0,24.2,24.3,24.7,25.0,25.2,25.5,25.8,25.9,26.1,26.2,26.4,26.7,27.0,27.3,27.5,27.9
E09000006,8.6,8.6,8.6,8.6,8.7,8.8,8.8,8.9,8.9,8.9,9.0,9.0,9.1,9.1,9.1,9.2,9.2,9.3,9.3
E09000007,42.7,43.1,43.4,43.5,43.9,44.2,44.5,44.8,45.2,45.5,45.8,46.0,46.2,46.5,46.7,47.1,47.7,48.1,48.5
E09000008,16.3,16.3,16.4,16.4,16.5,16.5,16.6,16.8,16.9,17.0,17.1,17.2,17.3,17.5,17.6,17.9,18.2,18.4,18.6
E09000009,21.7,21.7,21.8,21.9,21.9,22.1,22.3,22.6,22.8,22.8,22.9,23.0,23.2,23.3,23.5,23.6,23.8,24.0,24.4
E09000010,13.7,13.9,14.0,14.2,14.2,14.3,14.4,14.6,14.7,14.7,14.8,14.9,14.9,15.0,15.1,15.1,15.3,15.3,15.4
E09000011,18.9,19.0,19.2,19.5,19.8,20.1,20.2,20.3,20.3,20.4,20.5,20.7,20.7,21.0,21.2,21.5,22.0,22.4,22.7
E09000012,45.8,46.5,47.4,48.1,48.7,49.3,50.1,51.1,52.3,53.4,53.8,54.4,54.8,55.4,56.0,56.5,57.1,57.8,58.6
E09000013,44.9,45.1,45.2,45.6,45.9,46.2,46.6,46.9,47.2,47.7,48.0,48.3,48.5,48.9,49.7,49.9,50.5,51.4,52.0
E09000014,31.6,31.9,32.1,32.3,32.8,33.2,33.7,34.1,34.6,34.9,35.2,35.6,35.8,36.0,36.0,36.1,36.4,36.8,37.0
E09000015,16.0,16.0,16.2,16.3,16.4,16.5,16.7,16.8,16.9,17.0,17.1,17.2,17.4,17.4,17.5,17.7,17.8,18.0,18.2
E09000016,8.2,8.2,8.3,8.3,8.4,8.4,8.5,8.5,8.6,8.6,8.7,8.7,8.7,8.7,8.8,8.8,8.9,8.9,9.0
E09000017,8.6,8.6,8.6,8.7,8.7,8.8,8.8,8.8,8.9,8.9,9.0,9.1,9.2,9.2,9.3,9.3,9.4,9.5,9.6
E09000018,15.2,15.3,15.5,15.7,15.8,16.0,16.3,16.6,16.8,16.9,17.1,17.2,17.3,17.4,17.5,17.5,17.6,17.8,18.0
E09000019,56.0,56.7,57.9,58.7,59.2,59.8,61.0,62.2,63.7,64.8,65.2,66.0,66.6,67.5,67.8,68.5,69.0,69.2,69.8
E09000020,67.6,67.7,67.8,68.1,68.2,68.3,68.3,68.3,68.3,68.4,68.5,68.6,68.6,69.1,69.9,70.2,70.5,70.7,70.8
E09000021,16.8,16.8,16.9,17.0,17.2,17.2,17.3,17.4,17.4,17.5,17.5,17.6,17.6,17.7,17.8,17.9,18.0,18.0,18.2
E09000022,44.2,44.6,44.8,45.1,45.4,45.9,46.4,47.0,47.4,47.9,48.5,48.8,49.0,49.5,50.0,50.5,50.9,51.5,51.9
E09000023,30.9,31.0,31.1,31.4,31.6,31.9,32.1,32.4,32.7,33.0,33.3,33.6,34.2,34.4,34.8,35.2,35.7,35.8,36.3
E09000024,21.5,21.4,21.4,21.4,21.3,21.4,21.4,21.4,21.5,21.5,21.5,21.6,21.8,21.9,22.0,22.1,22.2,22.4,22.5
E09000025,24.2,24.5,24.8,25.1,25.2,25.5,25.7,25.9,26.2,26.6,26.8,27.0,27.2,27.7,28.2,28.6,29.2,29.7,30.3
E09000026,16.7,16.8,16.9,17.0,17.2,17.3,17.5,17.6,17.7,17.9,18.0,18.0,18.1,18.1,18.2,18.2,18.3,18.4,18.5
E09000027,13.3,13.4,13.4,13.5,13.6,13.8,13.8,13.9,13.9,14.0,14.0,14.1,14.2,14.2,14.3,14.3,14.4,14.5,14.6
E09000028,35.8,36.0,36.3,36.8,37.5,38.0,38.8,39.5,40.0,40.6,41.2,41.6,42.0,42.5,42.9,43.4,44.2,44.5,45.5
E09000029,17.7,17.7,17.8,17.8,17.9,17.9,17.9,18.0,18.1,18.1,18.2,18.3,18.4,18.4,18.5,18.6,18.8,18.9,19.1
E09000030,37.2,38.1,38.8,39.5,41.0,42.3,43.7,45.0,46.6,48.0,48.8,50.2,50.6,50.9,51.3,52.5,54.7,55.6,56.3
E09000031,23.7,23.8,23.9,24.0,24.1,24.3,24.6,24.8,25.1,25.2,25.3,25.5,25.6,25.7,25.8,26.1,26.4,26.5,26.7
E09000032,34.3,34.5,34.9,35.2,35.7,36.1,36.6,37.0,37.5,38.0,38.2,38.5,38.8,39.1,39.4,40.2,40.9,41.5,42.0
E09000033,45.9,46.8,47.5,48.5,49.1,50.0,50.7,51.5,52.2,52.9,53.7,54.1,54.4,54.6,55.0,55.4,56.0,56.5,56.9
"""
        |> fromCSV
```

^^^elm m=(tableSummary 3 dwellings )^^^

Each row contains the dwelling density values (dwellings per hectare) for a London borough. Unlike the previous examples, the borough is referenced not by its familiar name, but its standard [GSS code](https://gss.civilservice.gov.uk/wp-content/uploads/2018/03/Coding-and-Naming-Policy-for-UK-Statistical-Geographies-6.pdf). This provides a way of referencing spatial units without risking the ambiguities of spelling variations ("Richmond" or "Richmond upon Thames" or "Richmond-upon-Thames"?).

Each column represents the dwelling density for a given year for each borough. It should be apparent that the table is not yet in tidy format (see Session 5), so let's _gather_ the columns so that all dwelling figures sit in a single column:

```elm {l}
tidyDwellings : Table
tidyDwellings =
    let
        headings =
            List.map2 Tuple.pair
                (List.range 2001 2019 |> List.map (\s -> "dph" ++ String.fromInt s))
                (List.range 2001 2019 |> List.map String.fromInt)
    in
    dwellings |> gather "year" "dph" headings
```

^^^elm m=(tableSummary 5 tidyDwellings)^^^

To map these data, we need to join the dwelling data and the geospatial data. We do this similarly to the way we did with car ownership, except that here we make dwelling data the primary data source and lookup the geospatial boundaries as the secondary data source. We can then facet the maps by year to produce a set of small multiples:

```elm {l v}
dwellingMaps : Spec
dwellingMaps =
    let
        dwellingData =
            dataFromColumns []
                << dataColumn "GSSCode" (strColumn "GSSCode" tidyDwellings |> strs)
                << dataColumn "dwellings" (numColumn "dph" tidyDwellings |> nums)
                << dataColumn "year" (numColumn "year" tidyDwellings |> nums)

        boundaryData =
            dataFromUrl "https://gicentre.github.io/data/geoTutorials/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        trans =
            transform
                << lookup "GSSCode" boundaryData "properties.GSSCode" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "dwellings"
                    , mQuant
                    , mLegend
                        [ leTitle "Dwellings per hectare"
                        , leOrient loNone
                        , leX 520
                        , leY 650
                        , leDirection moHorizontal
                        ]
                    ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coFacet [ facoSpacing 5 ])
    in
    toVegaLite
        [ cfg []
        , dwellingData []
        , trans []
        , columns 4
        , facetFlow [ fName "year", fHeader [ hdTitle "" ] ]
        , specification (asSpec [ width 160, height 120, enc [], geoshape [] ])
        ]
```

The problem with this spatial juxtaposition is that it is very hard to compare dwelling densities between maps (not least because the data are so strongly spatially autocorrelated). So an alternative design would be to create a single map and show the trends over time for each borough. However, because of the very different sizes of boroughs, there isn't much room to fit in a time series chart in some of those smaller central London boroughs.

This is where relaxing the spatial arrangement of the map can help. Instead of showing each borough as an irregular shape, let's convert each one into a square and position it at its approximate geographic location.

We need to capture the location of each borough, this time not as a polygon boundary but as a single 'centroid' location. Here is a table that includes those values pre-computed.

```elm {l=hidden}
boroughs : Table
boroughs =
    """GSSCode,fullName,shortName,longitude,latitude,gridCol,gridRow
E09000021,Kingston upon Thames,Kng,-0.2869147,51.3878934,3,6
E09000008,Croydon,Crd,-0.0871655,51.3553174,5,6
E09000006,Bromley,Brm,0.0515366,51.371984,6,6
E09000018,Hounslow,Hns,-0.3671516,51.4683805,1,4
E09000009,Ealing,Elg,-0.3310193,51.5224761,2,3
E09000016,Havering,Hvg,0.221117,51.5643681,8,3
E09000017,Hillingdon,Hdn,-0.4456619,51.5414712,1,3
E09000015,Harrow,Hrw,-0.3412746,51.5977185,3,2
E09000005,Brent,Brt,-0.2678117,51.5585544,3,3
E09000003,Barnet,Brn,-0.2100179,51.6160184,4,2
E09000022,Lambeth,Lam,-0.1182788,51.4530792,4,5
E09000028,Southwark,Swr,-0.0746025,51.4731374,5,5
E09000023,Lewisham,Lsh,-0.0202512,51.4480862,6,5
E09000011,Greenwich,Grn,0.0562385,51.4728035,7,5
E09000004,Bexley,Bxl,0.1403452,51.4588106,8,5
E09000010,Enfield,Enf,-0.0872653,51.6509919,5,1
E09000031,Waltham Forest,Wth,-0.0126454,51.5940218,6,2
E09000026,Redbridge,Rdb,0.075865,51.5856698,7,3
E09000029,Sutton,Stn,-0.1775706,51.3620888,4,7
E09000027,Richmond upon Thames,Rch,-0.3129899,51.4421709,2,5
E09000024,Merton,Mrt,-0.1972469,51.4099498,4,6
E09000032,Wandsworth,Wns,-0.1864426,51.4513537,3,5
E09000013,Hammersmith and Fulham,Hms,-0.223125,51.4976035,2,4
E09000020,Kensington and Chelsea,Kns,-0.1926146,51.5032686,3,4
E09000033,Westminster,Wst,-0.1612971,51.5138977,4,4
E09000007,Camden,Cmd,-0.1574152,51.5463925,4,3
E09000030,Tower Hamlets,Tow,-0.032468,51.5150561,6,4
E09000019,Islington,Isl,-0.1074466,51.5455039,5,3
E09000012,Hackney,Hck,-0.0656452,51.5497653,6,3
E09000014,Haringey,Hgy,-0.1074732,51.5903739,5,2
E09000025,Newham,Nwm,0.0363853,51.5283428,7,4
E09000002,Barking and Dagenham,Bar,0.1335219,51.5452723,8,4
E09000001,City of London,Cty,-0.0923237,51.5152901,5,4"""
        |> fromCSV
```

^^^elm m=(tableSummary 5 boroughs)^^^

```elm {l v}
afterTheFloodLayout : Spec
afterTheFloodLayout =
    let
        data =
            dataFromColumns []
                << dataColumn "gridCol" (numColumn "gridCol" boroughs |> nums)
                << dataColumn "gridRow" (numColumn "gridRow" boroughs |> nums)
                << dataColumn "borough" (strColumn "shortName" boroughs |> strs)

        enc =
            encoding
                << position X [ pName "gridCol", pAxis [] ]
                << position Y [ pName "gridRow", pAxis [] ]
                << text [ tName "borough" ]

        squares =
            asSpec [ enc [], rect [ maFill "lightgrey", maStroke "white" ] ]

        labels =
            asSpec [ enc [], textMark [] ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite [ cfg [], width 500, height 437.5, data [], layer [ squares, labels ] ]
```

Note how we are no longer using longitude and latitude to reference location, but integer grid values.
This particular grid arrangement of London boroughs was designed by [After The Flood](https://aftertheflood.com/projects/future-cities-catapult/) and allocates an equal space to each borough with the grid alignment making it easier to compare values across rows and columns.

With this relaxed geography we have space to show changes over time in dwelling density as a series of small multiples. Not only is this much more compact than the collection of faceted maps, it more clearly shows differences in dwelling density between boroughs as well borough differences in rate of change in dwelling density over time:

```elm {v l interactive}
dwellingMap : Spec
dwellingMap =
    let
        d33 =
            leftJoin ( tidyDwellings, "GSSCode" ) ( boroughs, "GSSCode" )

        data =
            dataFromColumns []
                << dataColumn "gridCol" (numColumn "gridCol" d33 |> nums)
                << dataColumn "gridRow" (numColumn "gridRow" d33 |> nums)
                << dataColumn "borough" (strColumn "fullName" d33 |> strs)
                << dataColumn "dwellings" (numColumn "dph" d33 |> nums)
                << dataColumn "year" (numColumn "year" d33 |> nums)

        enc =
            encoding
                << position X [ pName "year", pQuant, pAxis [] ]
                << position Y [ pName "dwellings", pQuant, pAxis [] ]
                << row [ fName "gridRow", fHeader [ hdTitle "" ] ]
                << column [ fName "gridCol", fHeader [ hdTitle "" ] ]
                << tooltip [ tName "borough" ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coHeader [ hdLabelFontSize 0.1 ])
                << configuration (coFacet [ facoSpacing 0 ])
    in
    toVegaLite [ data [], cfg [], width 60, height 48, bar [], enc [] ]
```

Developing and using techniques for relaxing geography is one of the research themes in the giCentre. For example, see the papers by [Meulemans _et al_, 2017](#references) or [Wood _et al_, 2011](#references) that both explore the design space of grid maps and apply them in the process of visual data analytics.

{(task|}

What are the advantages and disadvantages of grid maps (and other forms of relaxed geography) compared with 'accurate' mapping of geospatial features?

What might determine whether you used relaxed or precise geospatial encoding?

{|task)}

For other examples of relaxed spatial layouts see the following from the 30 day map challenge:

- [Global country grid](https://github.com/jwoLondon/30dayMapChallenge/blob/main/d05Raster.md)
- [London green spaces](https://github.com/jwoLondon/30dayMapChallenge/blob/main/d08Green.md)
- [US migration](https://github.com/jwoLondon/30dayMapChallenge/blob/main/d12Movement.md)
- [London voting bias](https://github.com/jwoLondon/30dayMapChallenge/blob/main/d15Names.md)
- [US Presidential election results](https://github.com/jwoLondon/30dayMapChallenge/blob/main/d23Population.md)

## 6. Conclusions

In many ways visualizing spatial data is no different to visualizing any other types of data. We can choose which aspects of the data we encode with which channels to suit task at hand. But the correspondence between graphic position and geographic position makes positional encoding particularly compelling. On top of this, the prevalence of spatial autocorrelation in geographic phenomena means that spatial structure is very likely to be seen with positional encoding.

The added complexity of the irregular geometry of most spatial data means that boundaries of geographic features are typically stored in their own (geoJSON / topoJSON) files. The challenge of using such data is linking these boundary data with the particular thematic data we wish to visualize. We have seen how we can do this if the boundary and thematic data share some common key such as region name or GSS code. We can link the keys with the [lookup](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookup) function.

Whenever you wish to map geographic data, it may be tempting to use a faithful projection of the geography, but you should consider how important is a precise depiction of geographic boundaries and position. If it is not, relaxing the way geography is projected into graphic space provides additional design freedom. This can be particularly useful when embedding small multiples of other charts within this quasi-geographic space.

## 7. Recommended Reading

To become good at data visualization you will need to read beyond the lecture notes. 'Recommended reading' points you to accessible sources of further information related to the content of each session. At the very minimum, please read the relevant chapter of Munzner (2015).

**Munzner, T.** (2015) Chapter 8: _Arrange Spatial Data_, pp.179-198 in [Visualization Analysis and Design](https://go.exlibris.link/9jMy6fQG), CRC Press

### References

**Meulemans, W., Dykes, J., Slingsby, A., Turkay, C. and Wood, J.** (2017) [Small multiples with gaps](http://openaccess.city.ac.uk/15167/). _IEEE Transactions on Visualization and Computer Graphics_, 23(1), pp.381-390.

**Tobler, W.** (1970) [A computer movie simulating urban growth in the Detroit region](https://www.jstor.org/stable/pdf/143141.pdf), _Economic Geography_, 46(sup1), pp.234-240.

**Wood, J., Badawood, D., Dykes, J. and Slingsby, A.** (2011) [BallotMaps: Detecting name bias in alphabetically ordered ballot papers](https://openaccess.city.ac.uk/436). _IEEE Transactions on Visualization and Computer Graphics_, 17(12), pp.2384-2391.

## 8. Practical Exercises

{(task|} Please add answers to the following to your portfolio repo for session 8. You may also wish to devote some time this week to working on your datavis project, remembering to push regularly to the `project` folder of your repo.{|task)}

### 1. Labelling spatial features

If you haven't done so already, modify the London Car Ownership proportional circle map so that it also includes text labels for each borough.

_Hint: The existing specification contains two layers – one for the base-map and one for the circles. You will need to add a third layer that contains text marks using the same centroid data that were used to position each circle._

### 2. Map Projections

Which map projection types and rotations would be most appropriate for the following tasks. Why?

You may find it helpful to use the interactive projection explorer in [Section 4](#4-map-projections) to help explore your choices.

| Task                                                                    | Projection | Justification |
| :---------------------------------------------------------------------- | :--------- | :------------ |
| Global distribution of bad teeth per capita                             |            |               |
| Arctic ice sheet fluctuation over the last 50 years                     |            |               |
| Inter-country migration flows                                           |            |               |
| Bird migration patterns from the Netherlands to Europe, Asia and Africa |            |               |
| Crime patterns in the West Midlands                                     |            |               |
| Global incidences of Coronavirus                                        |            |               |
| Major ocean current flows                                               |            |               |

### 3. Global Data Geospatial Data Challenge

In the _Data Challenge_ of session 5, you should have created a proportional circle map of some global data from Gapminder. See if you can enhance it with any of the geospatial techniques discussed in this session (e.g. choosing an appropriate map projection; combining with country boundary data etc.).

If you didn't manage to complete the original exercise you can use as a start point the global democracy data considered in [democracyData.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/democracyData.md) and [democracyVisualization.md](https://staff.city.ac.uk/~jwo/datavis2021b/session05/democracyVisualization.md).

---

_Check your progress._

- [ ] I can display the boundaries within a topoJSON file in my own data visualization.
- [ ] I can extract an id or properties from a topoJSON file
- [ ] I can use [mapshaper](https://mapshaper.org) to convert and simplify geospatial data into a topoJSON file suitable for visualization.
- [ ] I can link geospatial boundary data with attribute data from a separate table using the [lookup](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookup) function.
- [ ] I can select an appropriate global map projection depending on the region and purpose of my geovisualization.
- [ ] I can create a grid map of London and use it to arrange a collection of charts in a quasi-geographic layout.

---

<!-- Appendix: Code for producing radial (pie and rose) charts using an azimuthal map projection -->

```elm{l=hidden}
pie : Spec -> Spec
pie =
    stack [ prType azimuthalEquidistant, prRotate 0 90 0 ]


donut : Spec -> Spec
donut geoData =
    let
        circleCoords lat =
            List.map (\x -> ( toFloat (x * 10), toFloat lat )) (List.range 0 36)

        pieSpec =
            asSpec
                [ dataFromJson geoData [ jsonProperty "features" ]
                , geoshape [ maStroke "white", maStrokeWidth 3 ]
                , encoding (color [ mName "properties.cat", mNominal, mTitle "" ] [])
                ]

        blankSpec =
            asSpec
                [ dataFromJson (geometry (geoPolygon [ circleCoords 30 ]) []) []
                , geoshape [ maFill "#fff" ]
                ]
    in
    asSpec
        [ projection [ prType azimuthalEquidistant, prRotate 0 90 0 ]
        , layer [ pieSpec, blankSpec ]
        ]


rose : Spec -> Spec
rose geoData =
    let
        gratSpec =
            asSpec
                [ graticule [ grStep ( 5, 5 ) ]
                , geoshape [ maStrokeWidth 0.2, maFilled False ]
                ]

        roseSpec =
            asSpec
                [ dataFromJson geoData [ jsonProperty "features" ]
                , geoshape [ maFillOpacity 0.7, maStroke "#fff", maStrokeWidth 4 ]
                , encoding (color [ mStr "#ccc" ] [])
                ]
    in
    asSpec
        [ projection [ prType azimuthalEquidistant, prRotate 202 -90 0 ]
        , layer [ roseSpec, gratSpec ]
        ]


range : Float -> Float -> Float -> List Float
range mn mx step =
    List.range 0 ((mx - mn) / step |> round)
        |> List.map (\x -> mn + (toFloat x * step))


scanl : (a -> b -> b) -> b -> List a -> List b
scanl fn b =
    let
        scan a bs =
            case bs of
                hd :: tl ->
                    fn a hd :: bs

                _ ->
                    []
    in
    List.foldl scan [ b ] >> List.reverse


toPolar : List ( String, Float ) -> Spec
toPolar pairs =
    case pairs of
        [] ->
            -- Invisible polygon if no data provided.
            geoFeatureCollection [ geometry (geoPolygon [ [ ( 0, 90 ), ( 1, 90 ), ( 0, 90 ) ] ]) [ ( "cat", str "" ) ] ]

        [ ( label, _ ) ] ->
            -- Full circle if singleton provided.
            geoFeatureCollection
                [ geometry (geoPolygon [ [ ( 0, 90 ), ( 359.9, -89 ), ( 0, -89 ), ( 0, 90 ) ] ])
                    [ ( "cat", str label ) ]
                ]

        _ ->
            -- Segments if two or more data items provided.
            let
                geom ( a, b, label ) =
                    geometry (geoPolygon [ [ ( 0, 90 ), ( b, -89 ), ( a, -89 ), ( 0, 90 ) ] ])
                        [ ( "cat", str label ) ]

                adjacent xs =
                    List.map2 Tuple.pair xs (List.drop 1 xs ++ List.take 1 xs)

                triplet =
                    let
                        degs xs =
                            scanl (\a b -> 360 * a / List.sum xs + b) 0 xs
                    in
                    pairs
                        |> List.unzip
                        |> Tuple.second
                        |> degs
                        |> adjacent
                        |> List.map2 (\label ( a, b ) -> ( a, b, label )) (List.unzip pairs |> Tuple.first)
            in
            geoFeatureCollection (List.map geom triplet)


toRose : List ( String, Float ) -> Spec
toRose pairs =
    case pairs of
        [] ->
            -- Invisible polygon if no data provided.
            geoFeatureCollection [ geometry (geoPolygon [ [ ( 0, 90 ), ( 1, 90 ), ( 0, 90 ) ] ]) [ ( "cat", str "" ) ] ]

        [ ( label, _ ) ] ->
            -- Full circle if singleton provided.
            geoFeatureCollection
                [ geometry (geoPolygon [ [ ( 0, 90 ), ( 359.9, -89 ), ( 0, -89 ), ( 0, 90 ) ] ])
                    [ ( "cat", str label ) ]
                ]

        _ ->
            -- Segments if two or more data items provided.
            let
                geom ( ( a, b ), r, label ) =
                    geometry (geoPolygon [ [ ( 0, 90 ), ( b, r ), ( a, r ), ( 0, 90 ) ] ])
                        [ ( "cat", str label ) ]

                adjacent xs =
                    List.map2 Tuple.pair xs (List.drop 1 xs ++ List.take 1 xs)

                quad =
                    let
                        degs =
                            range 0 360 (360 / (List.length pairs |> toFloat))

                        maxR =
                            List.unzip pairs |> Tuple.second |> List.maximum |> Maybe.withDefault 1

                        rs =
                            List.unzip pairs |> Tuple.second |> List.reverse |> List.map (\r -> 80 - 169.9 * r / maxR)

                        labels =
                            List.unzip pairs |> Tuple.first |> List.reverse
                    in
                    degs |> adjacent |> List.map3 (\label r ab -> ( ab, r, label )) labels rs
            in
            geoFeatureCollection (List.map geom quad)


stack : List ProjectionProperty -> Spec -> Spec
stack proj geoData =
    asSpec
        [ configure (configuration (coView [ vicoStroke Nothing ]) [])
        , width 200
        , height 200
        , projection proj
        , dataFromJson geoData [ jsonProperty "features" ]
        , geoshape [ maStroke "#fff" ]
        , encoding (color [ mName "properties.cat", mNominal, mTitle "" ] [])
        ]
```
