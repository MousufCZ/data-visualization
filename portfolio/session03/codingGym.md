---
id: litvis

narrative-schemas:
  - ../../narrative-schemas/teaching.yml
---

@import "../../css/datavis.less"

<!-- Everything above this line should probably be left untouched. -->

# Coding Gym 3 : Parameterising and Transforming Lists

_These exercises are designed to help you build confidence programming with Elm. They assume no prior experience of coding in any language other than that arising from previous weeks of this module. This document is designed to be viewed in VSCode with litvis so you can add your own code blocks to answer each question._

## 3.1. Functions with Parameters

The functions you have created in the previous gyms return the same value every time they are called. You can customise what they return by providing input _parameters_ that are used to generate the output. Parameters are indicated in a type annotation, naming their type and separating each of them and the output with a `->` symbol. For example:

```elm {l}
add : Float -> Float -> Float
add firstNum secondNum =
    firstNum + secondNum


greeting : String -> String
greeting name =
    "Good evening " ++ name
```

Parameterised functions can be _called_ by providing values for each of the parameters after the function name. For example:

```elm {l raw}
summary : Float
summary =
    add 120 84
```

```elm {l raw}
welcomeMessage : String
welcomeMessage =
    greeting "Ada Lovelace"
```

{(task|}

Create separate parameterised functions, each in their own code block, that do the following. Each code block should have the header `elm {l}`. To see the result, you should create separate code blocks with the header `elm {l raw}` and than call the functions you have created providing values for the parameters. See the examples `summary` and `welcomeMessage` above if you are unsure. If created successfully, you should see the evaluation of these functions appear in the preview window.

- A function that given two words as `String` parameters returns a single string with the words separated by an 'and'.
- A function that given a single `String` parameter representing a colour, returns "My favourite colour is ", naming the colour provided.
- A function to multiply three numbers provided as parameters.
- A function that creates a list of integers from 1 to a number provided as a parameter.
- A function that given two `Float` parameters, provides the square root of the sum of their squares.

{|task)}

```elm {l}
concatenate : String -> String -> String
concatenate str stri = 
    str ++ " and " ++ stri


favCol : String -> String
favCol str = 
    "My favourite colour is " ++ str

multi : Float -> Float -> Float -> Float
multi num1 num2 num3 =
    num1 * num2 * num3 

list1 : Int -> List Int
list1 el = 
    List.range 1 el

square : Float ->  Float
square num1 = 
    num1 * num1

srs : Float -> Float -> Float
srs num1 num2 =
    sqrt((square(num1) + square(num2)))
```

```elm {l raw}
two : String
two =
    concatenate "Cat" "Dog"
```

```elm {l raw}
fav : String
fav =
    favCol "Black"
```

```elm {l raw}
x : Float
x =
    multi 2 43 5.5
```

```elm {l raw}
listit : List Int
listit =
    list1 7
```

```elm {l raw}
sq : Float
sq =
    square 2
```

```elm {l raw}
rootsqs : Float
rootsqs =
    srs 4 8
```

## 3.2. Transforming Lists

Just as we applied operators to individual items, we can apply them to all the items in a list. This is commonly achieved with the in-built function [List.map](https://package.elm-lang.org/packages/elm/core/latest/List#map). This function takes two parameters: the first is the function that will be applied to each item in the list and the second is the list itself. For example to increment all the values in a list by 1 we could do the following:

```elm {l}
incByOne : List Int -> List Int
incByOne myList =
    let
        inc num =
            num + 1
    in
    List.map inc myList
```

```elm {l raw}
newList : List Int
newList =
    incByOne [ 10, 20, 30, 40 ]
```

{(task|}

Create separate parameterised functions, each in their own code block, that do the following. Each code block should have the header `elm {l}`. Test them by creating separate functions that supply the list as a parameter (like `newList` above)

- A function that given a list of words, adds the word "potato" to each of them.
- A function that adds 12 to all the items in a list of numbers.
- A function that doubles each item in a list of numbers
- A function that finds the sum of the squares of all numbers in a list (_Hint: You may find the function [List.sum](https://package.elm-lang.org/packages/elm/core/latest/List#sum) helpful here_).
- A function that given an integer, creates a list of even numbers from 2 to that integer.

{|task)}


```elm {l}
PotatoList : List String -> List String
PotatoList myList =
    let
        addPot word =
            word ++ " potato"
    in
    List.map addPot myList


addTwelve : List Int -> List Int
addTwelve myList =
    let
        inc num =
            num + 12
    in
    List.map inc myList

doubleElem : Float -> Float
doubleElem num1 =
    num1 * 2

doubleList : List Float -> List Float
doubleList myList =
    let
        dob num =
            doubleElem(num) 
    in
    List.map dob myList

sqrList : List Float -> List Float
sqrList myList =
    let
        sq num =
            square(num) 
    in
    List.map sq myList
```


```elm {l raw}
potList : List String
potList =
    addPotato ["purple", "potato", "spicy", "cold" ]
```

```elm {l raw}
addTwelveList : List Int
addTwelveList =
    addTwelve [0, 5, 12 ]
```

```elm {l raw}
doubled : Float
doubled =
    doubleElem(40)
```

```elm {l raw}
doubledList : List Float
doubledList =
    doubleElem[0, 5, 12 ]
```

```elm {l raw}
sqList : List Float
sqList =
    sqrList[1, 5, 12 ]
```