#### Second research question

##### Which industries will allow the most success for a new generation of scientists, researchers, and engineers?



```elm {l=hidden}
dataExp =
    dataFromUrl "industry_trend1.csv"
```
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
                --<< size [ mName "Weight_in_lbs", mQuant ]
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



____
# Insight

The following data visualisations branch starts with understanding which industries are leading the way for research and development.





____
# Design choice 


I took advantage of Tufte's minimalistic suggestion and looking at data-ink ration. Focusing on the dimensions of beauty, clarity, effectiveness, and simplicity.  I have also I focused on the effective use of colour and visual variable to answer my research question.

#### Data tidy
I wanted to make my data less confusing as there are currently 32 industries in my data sets. I was inspired by minimalism and focus on the top 7 fastest growing industries that are leading R&D expenditure. 

I decided to focus on these as this would be a perfect starting point for students to explore where. If they are interested, they have the option to explore other industries.

However, due to my data set no longer being complicated, easier to visualise, process and share. I did not have to apply any further transformation to it.


#### Next

Having mentioned above in insight 2 that I lacked data variables, this led me to abandon the coding visualisation. My skills to use vegalite was not good enough to make a comprehensive piece that will capture attention. My data lacked the quality to deal with the goal of promoting research and development landscape.

This led me to turn to creating a humanistic visualisation. I was inspired by Giorgia Lupi who used simple data to postcards. Simple data can become intriguing subject point. I wanted to do exactly the same with my design choice. ([See visualisation 3](#visualisation))

My first goal was to analyse all experimenting I had done using vegalite, as well as, this about the use of minimalism, striking colour and selecive visual variable.

The following pie chart stuck with me and I wanted to expand on it by combining data humanism and minimalism.
![pie chart](industryPieChart.png) 

Understanding the importance of sketching ideas on paper, I creating the follwing:

![Industry sketching](draftIndustry.jpg)

I also took the same approach for visualisation 4 and 4.1.
![Geo sketching](draftGeo.jpg) 

From this coursework, what I took away the most is that the possiblity is endless when sketching potential data visualisation possibilities. Drawing an idea made a physical representation and it allowed me to see characteristics and relationships of concepts more easily.

The final industry ranking sketch focused on our human instincts to draw, it was inspired by caveman paintings. I choose this approach because it is simple! 

Having completed the final piece however was not what I really wanted. It was lacking additional information. In this case, I believe for pie charts and scatterplots a story is better told giving a fuller information. This design slection is contrary to the belief of Edward Tufte.

Despite my current finding, I procceded to experiment further with more advanced data visuation with Edward Tufte's approach for my geographical data visualisation.

My data visualisation did not produe the inspiring revelations I wanted it to do so. It did not lead me to ask further question to investigate. This led me to go back to basics and go back and read Visualization Analysis and Design by Tamara Munzner before advancing to my final geo data visaulisation. 
----

 just like here and it's simillar/not to the other designs. 
[academic & impartial evidence to support my design choice and also justify data choice] 