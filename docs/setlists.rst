setlists
========

A ``setlist`` is an ordered, indexed
collection with unique elements.	It it more than just an **ordered Set**
in that the elements are accessible by index (ie. not just a linked set).

However, ``setlist``'s are not comparable like sets or lists. Equality
testing still works, but ``setlist(('a', 'c')) < setlist(('a', 'b'))`` does not
because we'd have to choose to compare by order or by set comparison.

There are two classes provided:

:class:`collections_extended.setlist`
	This is a mutable setlist
:class:`collections_extended.frozensetlist`
	This is a frozen (implements :class:`collections.abc.Hashable`) version of a setlist.

Both classes implement :class:`collections.abc.Sequence`, :class:`collections.abc.Set`

Examples
--------

.. code-block:: python

	>>> from collections_extended import setlist
	>>> import string
	>>> sl = setlist(string.ascii_lowercase)
	>>> sl
	setlist(('a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'))
	>>> sl[3]
	'd'
	>>> sl[-1]
	'z'
	>>> 'r' in sl	# testing for inclusion is fast
	True
	>>> sl.index('m')	# so is finding the index of an element
	12
	>>> sl.insert(1, 'd')	# inserting an element already in raises a ValueError
	Traceback (most recent call last):
	...
		raise ValueError
	ValueError
	>>> sl.index('d')
	3


Compared to existing similar implementations
--------------------------------------------

Most implementations I've see are ordered sets where items are not accessible
by index.

Compared to Standard Types
--------------------------

setlist vs. list
^^^^^^^^^^^^^^^^

* Inclusion testing is O(1)
* Finding an element is O(1)
* Adding an element that is already present raises a ValueError

setlist vs. set
^^^^^^^^^^^^^^^

* Elements are ordered and accessible by index
* Adding an element is as slow as adding to a list
	* Amortized O(n) for arbitrary insertions
	* O(1) for appending

New Methods
-----------
Aside from the methods expected from Sequence and Set, this provides:
- ``setlist.shuffle(random=None)`` Because random.shuffle(setlist) doesn't work, this is provided to do the same.

Errors
------
Some methods will raise a ``ValueError`` when trying to add or remove elements
when they respectively already or do not currently exist in the setlist.
Each method has an analagous version that does/doesn't raise a ValueError.
Methods implementing the Set methods do not raise ``ValueError`` while the one's
implementing do. All will raise ``TypeError`` if you use unhashable values.
The bulk operations are atomic, if any single value is unhashable or a duplicate,
no changes will be made to the ``setlist``.

======================  ===============  ==============
Raises ``ValueError``   No               Yes
Interface               ``Set``          ``Sequence``
======================  ===============  ==============
Add a single value      ``add``          ``append``
Add multiple values     ``update``       ``extend``
Remove a single value   ``discard``      ``remove``
Remove multiple values  ``discard_all``  ``remove_all``
======================  ===============  ==============

The setlist constructor by defualt does not raise ``ValueError`` on duplicate values
because we have to choose one or the other and this matches the behavior of Set.
There is a flag ``raise_on_duplicate`` that can be passed to ``__init__`` to
raise a ValueError if duplicate values are aoassed.

Quirks
------
* Swapping elements, eg. ``sl[0], sl[1] = sl[1], sl[0]``, doesn't work because
  it is implemented by first setting one element then the other. But since
  the first element it tries to set is still in the setlist, nothing happens.
  This causes random.shuffle not to work on a setlist.
