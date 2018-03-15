# Proposal for Periodic Boundary Conditions (PBC)

## Problem
Open Babel does not currently account for periodic boundary conditions.  Unit cells are isolated in space instead of accounting for bonds, distances, etc., to neighboring cells.

This issue was migrated as [Github #1536](https://github.com/openbabel/openbabel/issues/1536) and is [requested](https://www.mail-archive.com/openbabel-discuss@lists.sourceforge.net/msg02002.html) [periodically](https://sourceforge.net/p/openbabel/mailman/message/7048390/) on the mailing list.

> It would be great if OB could deal with structures and bond perception over periodic boundaries. This has already been implemented by G. Garberoglio in http://software-lisc.fbk.eu/obgmx/ but it would be great to get his patches merged into the standard version.


## Proposed Enhancement

Update the relevant methods in the API to apply the Minimum Image Convention for bond perception and geometric calculations, such as distances and angles.

I briefly discussed this work with Giovanni Garberoglio, who shared his relevant source code with me.
Our implementations were quite similar, so I am in the process of merging the best parts of our implementations together, which is currently in [my Open Babel repository](https://github.com/bbucior/openbabel/tree/periodic-boundaries).


## Detailed Explanation

### Theory
One way to represent atoms/particles in a periodic cell is to wrap all of their fractional coordinates between 0 and 1.
The minimum image convention is commonly used to calculate distances between two atoms in a unit cell.
By way of example, when calculating an atom's neighbors (`OBMol::ConnectTheDots`), only the closest copy of the other atoms should be considered for the distance comparison.
Otherwise, there would be infinite neighbors to consider.
To apply these conditions, after subtracting the two positions, the fractional coordinates of the vector are standardized in the range (-0.5, 0.5), ensuring that two atoms are no farther than half a box length away (thus the nearest copy).

### Implementation

When a periodic molecule is read, an `OBMol::OB_PERIODIC_MOL` flag is set with the appropriate setters/getters.
In my current implementation, the OBUnitCell is copied using methods like `OBMol::GetPeriodicLattice`.

In order to actually apply periodic boundary conditions, the atoms and bonds need to be aware of them, so they refer back to the settings of the parent OBMol.
Calls to vector-based calculations, such as `OBBond::GetLength`, automatically check for and apply periodicity conditions as necessary.
During implementation, I found that many usages of geometric properties, such as bond lengths, calculate the properties ad-hoc using the atomic coordinates/vectors instead of using the standardized built-in methods, so I cleaned up these usages, particularly in the stereo code.

The algorithm for the minimum image convention itself is now implemented as part of OBUnitCell, specifically in the functions `MinimumImageFractional(vector3)` and `MinimumImageCartesian(vector3)`.
I also added helper functions `UnwrapCartesianNear` and `UnwrapFractionalNear` to unwrap coordinates (thus potentially a molecule) into absolute coordinates extending beyond the unit cell.
For example, torsion is a four-body term.  Since the calculation extends beyond next-nearest neighbors, it is easiest to temporarily "unwrap" a copy of the relevant atoms so that they are in the same unit cell image.

Enabling periodicity is now implemented as an input option for mmcif.
An optional input flag is required to enable periodicity, which will help during the intermediate stages of implementation, when some use cases expect PBC (energy calculations) while others don't (png/svg formats).

### Alternatives to consider
1. **`OBMol::GetPeriodicLattice`, etc.:** The code could be simplified by removing these methods entirely and/or using the unit cell object already saved via `OBMol::SetData`.  If I refactor to that approach, accessing the lattice data would look something like `OBUnitCell * pCell = (OBUnitCell * )pmol->GetData(OBGenericDataType::UnitCell);`.
2. **`OB_PERIODIC_MOL` flag:** The "periodic" property could be moved from a molecular flag to a bool setting within OBMol (like `_autoFormalCharge`)
3. Can we avoid changing the `const` flag for `OBBond::GetLength()` without making a lot of other changes?
4. Could consider saving the coordinates in a new, native OBCoord class instead of converting between Cartesian and fractional as much, but changes to the project may be considerably more involved.

### Status
Proof-of-concept code is written and has been tested as part of an ongoing research project.  Some changes are necessary before it's ready to merge upstream:

1. Decide on the appropriate API and implementation details, then clean up the methods as necessary
2. Write new formal test cases (what would be some good ideas?).  Verify that my changes do not break existing tests on the newest development version of Open Babel.
3. Implement periodicity (or definitive STUB behavior) in areas like `OBBond::SetLength`, `OBBuilder`, and the SVG format.
4. Update Open Babel version in periodic-boundaries branch to current and prepare PR

### Edge cases

1. Small unit cells

Once periodic boundaries are applied, an atom could theoretically be bonded to its image (A-A') or twice to the same atom (A-B-A', which would use the A-B OBAtom pointers nonuniquely).
The easiest method to address these cases would be issuing a warning if any of the lattice parameters are too small (defined as twice a reasonable maximum bond distance).
Otherwise, OBBond and periodicity code would be far too complicated if it had to keep track of the atom images in other unit cells, and methods like `OBBond::GetBond(bgn, end)` will break and/or fail to capture every neighbor.

2. Addition of molecules with dissimilar unit cells

Should any additional checks be performed when two molecules are added together?
Adding a nonperiodic OBMol to a periodic one?
Adding two periodic structures together?
There is a current ambiguity in the source code (`OBMol::operator+=`) for handling additions of 2D structures to 3D.
Note that Materials Studio completely disallows adding periodic structures together unless their lattice parameters match.

3. Atoms on the borders of unit cells

Atoms near unit cell boundaries can easily cause errors if not handled consistently, for example in calculating relative positions between atoms in different unit cells.
`OBUnitCell::WrapFractionalCoordinate` anticipates this using a fuzzy border, where atoms within a certain limit of the unit cell (1.0e-6 in fractional coordinates) are wrapped consistently.
However, atoms may not necessarily be wrapped if read verbatim from a CIF or similar format.

4. Are there any forseeable complications with symmetry operations?  I am not familiar with that code.


## Pros and Cons

Pros associated with this implementation include:

* Minimal number of changes to lines of existing code
* Hiding periodicity behind a flag can reduce compatibility issues
* Once implemented, periodic bonds can be properly exported, e.g. CIF format or energy calculations
* As a side benefit, this work will clean up code in the project to standardize how certain methods are used in the API.  For example, some files use `OBBond.GetLength()` while others manually recalculate the distance (or angle, torsion, etc.)

Cons associated with this implementation include:

* Certain formats become strange and nonstandard in periodic systems, e.g. SMILES of polymers are currently indistinguishable from a hydrocarbon ring
* From limited testing, it appears that InChI cannot handle wrapped coordinates in the unit cell, particularly when determining stereochemistry
* Possibility of breaking assumptions in downstream codes, though these projects are likely frozen on old versions of Open Babel
* Updates to other formats, such as SVG, are required to make sure the output is still reasonable
* Ease of introducing subtle bugs when handling periodic structures.  Maintenance could be more difficult depending on how contributors implement calculations (see also the last "pro" point above)
* Not handling unit cells below a certain size


## Interested Contributors
@bbucior

