# Rename valence and degree methods

## Problem

Two fundamental properties of nodes in a chemical graph are the degree and valence, the former being the number of incident bond and the latter being the sum of their bond orders. Typically you want the values either restricted to the explicit graph, or also including implicit hydrogens.

OBAtom currently has the following relevant methods: GetValence(), GetHvyValence(), GetHeteroValence() and BOSum(). The problem is that the first three functions are named incorrectly - they relate to the degree not the valence. The final function actually returns the explicit valence but could benefit from a more obvious name.

## Proposed Enhancement

I propose removing the four functions listed above from the API, and instead add the following: CalcExplicitValence(), CalcTotalValence(), GetExplicitDegree(), GetTotalDegree().

The "Total" functions are simply the Explicit ones plus the implicit H count. "Calc" is used instead of "Get" for the valence methods to indicate that there is work involved in returning the valence (and so the user should cache the value if repeatedly required).


## Details

The names are quite long, and so we could drop the "Get/Calc". Alternatively, we could drop the "Total" from those names.

## Pros and Cons

Pros associated with this implementation include:
* Names for core API methods that correspond to their function
* No more need for Noel's API cheatsheet on how to calculate degree and valence
* Updating to the new API just needs a search+replace for the most part.

Cons associated with this implementation include:
* GetValence() will be replaced by CalcExplicitDegree() which is not obvious.
