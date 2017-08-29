# Proposal for tweaks to the OBReaction API

## Problem
While reactants and products are stored in a list by OBReaction, agents are represented by a single molecule. However, multiple agents are not uncommon (e.g. solvent, catalyst).

The use of a shared_ptr prevents access to OBReaction in the language bindings.

## Proposed Enhancement

I propose to treat agents the same way as reactants and products.

I propose that shared_ptr is replaced by a regular pointer.

## Detailed Explanation

I've been encouraged to implement support for RInChI, but have been stymied by these two issues (the latter mainly affecting my ability to test the code). What I actually want to do is completely overhaul the handling of reactions (use an overlay over OBMol, rather than a separate OBReaction), but this relatively minor change will at least let me proceed with RInChI support.

## Pros and Cons

Pros associated with this implementation include:
* Reactions with multiple agents can be handled
* It will be possible to use reactions from the language bindings.

Cons associated with this implementation include:
* These are API-breaking changes

## Interested Contributors
@baoilleach
