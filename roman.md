---
layout: project
link: "ageiersbach/RomanDate"
project: "RomanDate"
tag: "roman"
---

I created this app to learn Erlang. It takes a Date and returns a string
representation of the date according to *ab urbe condite*, or AUC. It also
will provide the roman numeral for a number.

Since there's very little code I can include it all here:

{% comment %}
TODO
I should have a more detailed description of what I did here!
- what was it like working with Erlang?
- what did I find difficult?
{% endcomment %}

```
to_roman(Number) ->
  build_roman( Number, "", lists:reverse( roman_numerals_dict()) ).

%% ROMAN NUMERAL BUILDER FUNCTIONS

build_roman(Number, RomanNumeral, NumeralsList) when Number == 0 ->
  RomanNumeral;

build_roman(Number, RomanNumeral, NumeralsList) when Number > 0 ->
  {Divisor, Numeral} = greatest_divisor(Number, NumeralsList),
  NewNumber = Number - Divisor,
  build_roman( NewNumber , RomanNumeral ++ Numeral, NumeralsList ).

greatest_divisor(Number, NumeralsList) ->
  lists:max( lists:filter( divides_number(Number) , NumeralsList)).

divides_number(Number) ->
  fun({K,_}) -> Number div K > 0 end.

%% TREE BUILDER FUNCTIONS

roman_numerals_dict() ->
  orddict:from_list( build_list(roman_numerals(), []) ).

subtract({Number, Numeral}, {OtherNumber, OtherNumeral}) ->
  { Number - OtherNumber, OtherNumeral ++ Numeral }.

is_less_than(NumeralTuple, OtherNumeralTuple) ->
  if
    element(1, NumeralTuple) < element(1, OtherNumeralTuple) -> true;
    true -> false
  end.

lesser_numerals(NumeralTuple) ->
  fun(A) -> is_less_than(A, NumeralTuple) end.

subtract(NumeralTuple) ->
  fun(A) -> subtract(NumeralTuple, A) end.

numerals_less_than_given(NumeralTuple) ->
  lists:filter(lesser_numerals(NumeralTuple), roman_numerals()).

subtracted_numerals(NumeralTuple) ->
  lists:map( subtract(NumeralTuple), numerals_less_than_given(NumeralTuple)).

not_duplicate_numeral() ->
 fun(A) ->
   Key = element(1, A),
   Dict = orddict:from_list(roman_numerals()),
   not orddict:is_key(Key, Dict)
 end.

numerals_with_subtracted(NumeralTuple) ->
  SubtractedNumerals = lists:filter( not_duplicate_numeral(), subtracted_numerals(NumeralTuple)),
  [NumeralTuple] ++ SubtractedNumerals.

build_list(L, BuiltList) when length(L) == 0 ->
  BuiltList;

build_list(L, BuiltList) when length(L) > 0 ->
  {Tuple, Remaining} = lists:split(1, L),
  NewBuiltList = BuiltList ++ numerals_with_subtracted(lists:last(Tuple)),
  build_list( Remaining, NewBuiltList ).

roman_numerals() ->
  [
    {    1,  "I" },
    {    5,  "V" },
    {   10,  "X" },
    {   50,  "L" },
    {  100,  "C" },
    {  500,  "D" },
    { 1000,  "M" }
  ].
  ```
