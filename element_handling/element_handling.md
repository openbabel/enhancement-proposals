# Improve performance when handling elements

## Problem

Reading and writing file formats often involves converting between string representations of elements and their atomic number. This core functionality is provided by OBElementTable, but it is not designed for performance (see GetAtomicNum()). This class has also been used as a container for associated information such as RGB values and electron affinity, properties which may be out-of-scope or better stored elsewhere.

In addition, the API has accumulated several convenience functions such as OBAtom::IsHydrogen() which should be retired.

## Proposed Enhancement

I propose an OBElement namespace which would contain a set of functions such as GetSymbol() and GetAtomicNum(). Other functions related to physical properties will probably go here too. These will be optimised for performance. In particular, GetAtomicNum() will only support element names. The current code also translates English names, e.g. "Silver", but I'm not aware of any file format that requires this support.

To replace IsHydrogen() and friends, the OBElement namespace should contain an Elements enum with values like Carbon, etc.

The existing OBElement (used as a container class for element related information) would be removed. The OBIsotopeTable will probably also be converted into an OBElement namespaced function as part of this API change.

## Detailed Explanation

### Performance issues

Consider OBElementTable::GetAtomicNum(string name, int &iso). Just from this prototype, we can already see that the supplied name is copied. Then a case insensitive match is done to the first 3 characters of everything in the element table. One at a time. Then it's compared to the English names of the elements. Then D or Deuterium, T or Tritium, then Hl (yup, a little surprise for us all there), and then '*'. If the name is actually '*', then that's over 200 string comparisons. But even for 'C', there is more work done than necessary.

The other functions aren't so bad, e.g. GetSymbol() is an array lookup. But if the array was compiled-in, this would be more efficient and also would avoid the overhead of function calls on the OBElement object.

### Implementation

The current implementation reads data from a file at runtime, and uses an instance of a global object. The new implementation will instead compile everything in. For example, GetAtomicNum() will use a switch statement on the characters, similar to that already used in the SMILES parser. GetSymbol() will do an array lookup.

For convenience, the data used to generate the arrays will be a file similar to elements.txt, but it will be a header file. Using a multiple include trick, this will be read into arrays in the source code at compile time. (I think this is possible.)

## Pros and Cons

Pros associated with this implementation include:
* Better performance
* A leaner codebase
* No need for elements.txt/elements.h in the data directory

Cons associated with this implementation include:
* It may be a somewhat more difficult to add a new element, though I don't expect it to be too bad

## Interested Contributors
@baoilleach
