---
title: "elm-satで数独を解く"
date: 2021-05-09T21:30:09+09:00
tags: ["Elm", "formal methods"]
---

[elm-sat](https://package.elm-lang.org/packages/tortis/elm-sat/latest/)という SAT ソルバーの存在を知ったので、数独を解いてみる。変数 mondai を乱数で適当に(数独のルールに従う範囲で)定義するようにすれば数独の問題作成にも(問題として面白いかは別として)使える。

<!--more-->

Qiita あたりに解説とともに載せようかと思ったけど、あまり時間が作れないのでとりあえずコードのみ。

マニュアルに記載の通り、パフォーマンスで言うと minisat の js バインディングとかの方が良いとのことなので、Elm で Port とか使うの面倒臭いとか遊びの範囲での利用になると思う。
ただ、CNF 形式のデータを`List (List Int)`で作成してソルバーに渡せば良いだけなので、インタフェースとしての使い勝手は良い。(とはいえあ自分が使うようなケースだと SMT ソルバーの方が良いと思うけど)

```elm
module Main exposing (..)

import Browser
import Html as H exposing (Html)
import Html.Attributes exposing (style)
import List.Extra as List
import Random
import Random.Extra.List as RandomList
import Sat


type alias Model =
    List Int


type Msg
    = Seed ( List Int, List Int )


main : Program () Model Msg
main =
    Browser.element
        { init = \_ -> ( [], init )
        , view =
            \code ->
                sudoku code
        , update = update
        , subscriptions = \_ -> Sub.none
        }


init : Cmd Msg
init =
    List.range 2 9
        |> RandomList.shuffle
        |> Random.map ((++) [ 1 ])
        |> (\x -> Random.pair x x)
        |> Random.generate Seed


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    ( model, Cmd.none )


mondai : List (List Int)
mondai =
    []
      |> List.append [ [ 1 ], [ 11 ], [ 21 ], [ 31 ], [ 41 ], [ 51 ], [ 61 ], [ 71 ], [ 81 ] ]
      |> List.append [ [ 90 ], [ 170 ], [ 250 ], [ 330 ], [ 410 ], [ 490 ], [ 570 ], [ 650 ], [ 730 ] ]

cnf : List Int
cnf =
    vars 81
        |> List.append mondai
        |> allLines
        |> allCols
        |> allBoxes
        |> Sat.solve
        |> Maybe.map (List.filter ((<) 0))
        |> Maybe.withDefault []


vars : Int -> List (List Int)
vars i =
    case i of
        0 ->
            []

        _ ->
            let
                cell =
                    (81 - i) * 9
            in
            List.range 1 9
                |> List.map (\x -> cell + x)
                |> (\x -> x :: vars (i - 1))


allLines : List (List Int) -> List (List Int)
allLines xs =
    List.range 0 8
        |> List.map forEachLine
        |> List.concat
        |> List.append xs


forEachLine : Int -> List (List Int)
forEachLine y =
    List.range 1 9
        |> List.map (\n -> forEachCell1 y n)
        |> List.concat


forEachCell1 : Int -> Int -> List (List Int)
forEachCell1 line num =
    List.range 0 8
        |> List.map (\col -> (line * 9 + col) * 9)
        |> (\xs -> checkIn xs num)



-- ///


allCols : List (List Int) -> List (List Int)
allCols xs =
    List.range 0 8
        |> List.map forEachColumn
        |> List.concat
        |> List.append xs


forEachColumn : Int -> List (List Int)
forEachColumn x =
    List.range 1 9
        |> List.map (\n -> forEachCell2 x n)
        |> List.concat


forEachCell2 : Int -> Int -> List (List Int)
forEachCell2 col num =
    List.range 0 8
        |> List.map (\line -> (line * 9 + col) * 9)
        |> (\xs -> checkIn xs num)



-- ///


allBoxes : List (List Int) -> List (List Int)
allBoxes xs =
    List.range 0 8
        |> List.map forEachBox
        |> List.concat
        |> List.append xs


forEachBox : Int -> List (List Int)
forEachBox b =
    List.range 1 9
        |> List.map (\n -> forEachCell3 b n)
        |> List.concat


forEachCell3 : Int -> Int -> List (List Int)
forEachCell3 box num =
    let
        cells =
            [ 0, 1, 2, 9, 10, 11, 18, 19, 20 ]
                |> List.map ((+) (27 * (box // 3) + 3 * remainderBy 3 box))
                |> List.map ((*) 9)
    in
    checkIn cells num



-- ///


checkIn : List Int -> Int -> List (List Int)
checkIn members num =
    members
        |> List.map ((+) num)
        |> List.map negate
        |> List.uniquePairs
        |> List.map (\( x, y ) -> [ x, y ])


sudoku : List Int -> Html Msg
sudoku cs =
    cs
        |> List.indexedMap
            (\i c ->
                c - (i * 9)
            )
        |> List.groupsOf 9
        |> List.map sudokuLine
        |> H.div []


sudokuLine : List Int -> Html Msg
sudokuLine nums =
    nums
        |> List.map (String.fromInt >> H.text)
        |> List.map
            (\x ->
                H.div
                    [ style "width" "48px"
                    , style "height" "48px"
                    ]
                    [ x ]
            )
        |> H.div [ style "display" "flex" ]
```
