# Proposal for removal of Kekule and Aromatic Bond Orders

## Problem
In Open Babel, bonds have two orders, that returned by GetBondOrder() and that stored via flags as IsKSingle(), IsKDouble(), IsKTriple().

Seperately, GetBondOrder() should not return '5' for aromatic bonds (it's not clear if it does, but let's agree that it shouldn't and code to that).

## Proposed Enhancement

I am working on replacing the existing Kekulization code. I propose to remove any code associated with any bond orders starting with a 'K'. I also will remove any references to bond orders of 5.

## Detailed Explanation

### The problem with Kekule Bond Orders

The 'K' in these methods appears to refer to Kekule. Although these functions are marked as deprecated, they are still used, for example in SMILES writing. There is also an associated function KBOSum(). This all confused me.

The main problem is simply that Kekule bond orders are not distinct from 'real' bond orders. Kekulization sets the bond orders - there aren't two sets of bond orders, before and after. Before, the bond is 'aromatic' (or to be exact, it is part of an alternating double/single bond system), but afterwards it either a single bond or a double bond. It is an invariant of the toolkit (like all others) that bonds must be kekulized by the reader.

### The problem with bond order of 5 for aromaticity

First of all, what if there is a bond of order 5? Second, why is this even needed? All bonds are Kekulized by the reader and information on aromaticity of bonds is available via IsAromatic() (stored via a flag). At this point it's not clear to me whether the value of '5' is ever set anywhere. What I do know is that the BOSum() method treats the value of '5' as special instead of just totting up the bond orders.

## Pros and Cons

Pros:
* Both of these cases describe situations where the code and the situations handled are unnecessary. Their removal will lead to simpler and clearer code, less confusion, and performance improvements.

Cons associated with this implementation include:
* This needs to be done in association with a rewrite of the Kekulization.

## Interested Contributors
@baoilleach
