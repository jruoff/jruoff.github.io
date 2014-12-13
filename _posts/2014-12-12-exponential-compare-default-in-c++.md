---
layout: post
category : programming
tagline: Lexicographical ordering and less than comparison
tags : [C++, programming, flaws]
---
{% include JB/setup %}

Recently I noticed a design flaw in the C++ standard. (Even though this flaw was foreseen by Alexander Stepanov, the designer of the C++ standard template library.) Leading to an algorithm with exponential time-complexity for lexicographical ordering defined in terms of `operator<`. In case you are wondering, yes, this type of ordering is the default for all STL containers.


## Sorting and Searching

A common task in programming is to sort a range of elements. Usually, to find things quickly by means of binary search. It is not coincidence that Donald Knuth has dedicated the whole third volume of "The Art of Computer Programming" to this exact topic.

To sort things, an ordering relation needs to be defined. This ordering relation defines in which order elements should appear in the sorted range. Ordering relations can be defined in a variety of ways. To define an ordering in C++, by default, one needs to define `operator<`. That is, a function which given `x` and `y` returns true if and only if `x < y`. This is enough, because we can define all other ordering operators in terms of this one, thanks to mathematics:

	(x >  y)  ==   (y < x)
	(x <= y)  ==  !(y < x)
	(x >= y)  ==  !(x < y)


## Lexicographical ordering

When composing elements that have an ordering defined, it is often desirable to extend the ordering to the composition. For example, there is an ordering defined on characters (defined by the alphabet). Words are composed of characters, so can we define an ordering on words? Yes! We can use lexicographical ordering (as used in a dictionary) to define an ordering on words, in terms of the ordering on characters. This way we can do sorting and searching on words.

An even simpler example would be to consider the composition of two elements into a pair. In C++ you could define lexicographical ordering on pairs in terms of the ordering on the first and second elements as follows:

	template <typename T, typename U>
	bool operator<(const pair<T, U> &x, const pair<T, U> &y)
	{
		if (x.first < y.first) return true;
		if (y.first < x.first) return false;
		return x.second < y.second;
	}

Note that in the case the two pairs are equal, the function `bool operator<(T, T)` is called twice. This doesn't scale under composition. For example, the function `bool operator<(T, T)` is called 16 times when comparing two equal values of the type `pair<pair<pair<pair<T, U>, U>, U>, U>`. In general, `operator<` becomes an operation that has exponential complexity in the number of layers of composition!

Note that this problem always occurs when defining a lexicographical ordering in terms of `operator<`, that is, it occurs in all STL containers.

On the contrary, if a three-way comparison function was used to define the ordering, the exponential complexity problem could be easily avoided.
