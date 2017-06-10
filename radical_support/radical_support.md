# Supporting radicals

## Problem

After changing Open Babel to use an explicit count of implicit hydrogens, radical support was lost and needs to be restored. It needs to be somewhat handled in a different way than before as the previous support was built as a correction on top of the original implicit valence code.

## Proposed Enhancement

We will use the degree of under-valence to infer basic radical characteristics - is it a radical, diradical, triradical? Where two spin states are possible, we will store this information as unknown/paired/unpaired with flags on the atom.

## Detailed Explanation

### Under-valence

The fact that something is a radical does not need to be stored explicitly. Instead it can be inferred from the degree of undervalence. 

For example [CH3] is one under-valent, and so is a radical. [CH2] is two under-valent, and so is a diradical.

A function OBAtomNumRadicalElectrons(OBAtom*) can return this value. This will be done based on the code in 'GetTypicalValence'.

### Spin state

I propose a function OBAtom::SetSpinState(enum UNKNOWN/PAIRED/UNPAIRED) to distinguish between the high and low spin versions of diradicals (and higher radicals).

I do not want to use the word 'multiplicity' or the enum SINGLET/DOUBLET etc here as it may lead the user to conclude that this function is required to indicate something is a radical. That is not the case. It is only to distinguish between singlets and triplets, doublets and quadruplets, or to indicate that that information is not known.

The UNKNOWN value is important as this information is not present in SMILES strings for example and we should not infer it. Only if known, should we indicate the multiplicity in a MOL file. We should however provide ops (e.g. --singlet/--triplet) to set the multiplicity if undefined for all diradicals in a molecule (note to self: avoid confusion with singlet/triplet for the molecule itself).

This information will be stored using two bits in the atom flags.

## Pros and Cons

Pros associated with this implementation include:
* The data that is stored with the atom (spin state), versus data that is perceived from the structure (under valence) are kept separate. (Note to self: the docs for one should point to the other.)

Cons associated with this implementation include:
* To answer the question, "is this atom a singlet?", the user will need to combine information from the under-valence and the spin state.
* It is not possible to indicate that something is a radical if the valence is not consistent with this.

## Interested Contributors
@baoilleach

