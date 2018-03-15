# Proposal to handle reactions as OBMols

## Problem

Currently reactions are stored and handled as OBReaction objects with the individual components of a reactions available as shared pointers. This causes problems for language bindings (as SWIG cannot adequately handled shared pointers) and for plugins that must return or handle both reactions and molecules.

IMO the treatment of reactions as a separate class to OBMol causes more problems that it helps.

## Proposed Enhancement

I propose that the OBMol object be used to store reactions. Atoms will be annoted with their reaction role and component, and a facade will be provided to handle reaction-related activities.

## Detailed Explanation

Let me begin with an example of the sorts of difficulties that the distinction between OBMol and OBReaction is causing:

  obabel -irsmi -:C>>O -orsmi # fine
  obabel -ismi -:C>>O -orsmi  # segfault

How can one run a reaction SMARTS (or even a regular SMARTS) against an OBReaction? How can we depict an OBReaction?

One solution to all of this is for everything to take OBMolBase and use typeids or somesuch to determine the actual type, so that they can be handled differently. But actually everything becomes so much easier to code if we just use OBMol for everything.

For example, reading SMILES. The current code scans for '>', splits the string, and reads each part separately. Instead, we just read from left-to-right once (not twice), we don't do any string copies, and we annotate the atoms as we read them in, changing the reaction role every time we come to a '>'.

## Implementation

For each atom, we need to annotate its reaction role (none, reactant, product, agent) and its component (an integer). The first can be done through flags (two bits); the second will either need a dedicated field or else an OBPairData of some type. Given that reaction components aren't really required to store a reaction, an OBPairData might be a better choice, at least for now.

For each molecule, we probably should annotate whether it contains a reaction, probably via a flag. This would allow even the empty molecule to be handled correctly.

A facade class will be provided that hides these low-level details from the user. It will also provide methods like GetComponent(1), GetReactant(1), NumReactants(), and so on.

While it should be possible to interconvert between this representation and an actual OBReaction, I am not very keen on having two ways of representing a reaction.

## For comparison

RDKit, and Indigo use a separate Reaction class. OEChem doesn't. The CDK has a Reaction class but John has more recently written a convertor to/from the Molecule representation which he uses for SMARTS searching (at least).

## Pros and Cons

Pros associated with this implementation include:
* It will be possible to use reactions from the language bindings.
* Existing API functions that handle OBMols will at least accept reactions.
* File formats that mix reactions and molecules can be properly handled without the user need to choose one or the other (e.g. rsmi vs smi)
* I think that this simplifies the codebase

Cons associated with this implementation include:
* The idea of storing a reaction in a container for a molecule may seem strange, though I note that already multiple molecules can be stored in an OBMol.

## Interested Contributors
@baoilleach
