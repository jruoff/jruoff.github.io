---
layout: post
category : programming
tagline: Lexicographical ordering and less than comparison
tags : [C++, programming, flaws]
---
{% include JB/setup %}

Recently I noticed a design flaw in the C++ standard. Even though this flaw was foreseen by Alexander Stepanov, the designer of the C++ standard template library, it is prevalent in its current implementation. The flaw leads to an algorithm with exponential time-complexity for lexicographical ordering implemented in terms of `operator<`. In case you are wondering, this type of ordering is the default for all STL containers.


## Sorting and Searching

A common task in programming is to sort a range of values. Usually, to find things quickly by means of binary search. It is not coincidence that Donald Knuth has dedicated the whole third volume of "The Art of Computer Programming" to this exact topic.

### Less-than comparison
To sort a range of values, an ordering relation needs to be defined on these values. This ordering relation defines in which order elements should appear in the sorted range. Ordering relations can be defined in a variety of ways. To define an ordering in C++, by default, one needs to implement `operator<`. That is, a function which given `x` and `y` returns true if and only if `x < y`. This is enough, because we can implement all other ordering operators in terms of this one, thanks to mathematics:

	(x >  y)  ==   (y < x)
	(x <= y)  ==  !(y < x)
	(x >= y)  ==  !(x < y)


### Lexicographical ordering

When composing values that have an ordering defined, it is often desirable to extend the ordering to the composition. For example, there is an ordering defined on characters (defined by the alphabet). Words are composed of characters, so can we define an ordering on words? Yes! We can use lexicographical ordering (as used in a dictionary) to define an ordering on words, in terms of the ordering on characters. This way we can do sorting and searching on words.

An even simpler example would be to consider the composition of two values into a pair. In C++ you could implement lexicographical ordering on pairs in terms of the ordering on the first and second values as follows:

	template <typename T, typename U>
	bool operator<(const pair<T, U> &x, const pair<T, U> &y)
	{
		if (x.first < y.first) return true;
		if (y.first < x.first) return false;
		return x.second < y.second;
	}

Note however, that in the case the two pairs are equal, the function `bool operator<(T, T)` is called twice. This doesn't scale under composition. For example, the function `bool operator<(T, T)` is called 16 times when comparing two equal values of the type `pair<pair<pair<pair<T, U>, U>, U>, U>`. In general, `operator<` becomes an operation that has exponential complexity in the number of layers of composition!

Note that this problem always occurs when implementing a lexicographical ordering in terms of `operator<`, that is, it occurs in all STL containers.


### Three-way comparison

Instead of implementing `operator<` to define an ordering, it is possible to define an ordering by implementing a three-way comparison function `compare`. This function should satisfy the following (trichotomy) properties:

	(x <  y)  ==  (compare(x, y) <  0)
	(x == y)  ==  (compare(x, y) == 0)
	(x >  y)  ==  (compare(x, y) >  0)

Using the a three way comparison to define ordering, we could implement our lexicographical ordering on pairs as follows:

	template <typename T, typename U>
	int compare(const pair<T, U> &x, const pair<T, U> &y)
	{
		int r = compare(x.first, y.first);
		if (r != 0) return r;
		return compare(x.second, y.second);
	}

Note that both `int compare(T, T)` and `int compare(U, U)` are called at most once. Which means that our comparison function scales under composition. For example, now the function `int compare(T, T)` is called only once when comparing two values of the type `pair<pair<pair<pair<T, U>, U>, U>, U>`. As many programmers would (and should be reasonably able to) expect.


## Conclusion

Although I first assumed `operator<` and `compare` to be equivalently adequate to define an ordering on a type, this is not the case. Even though both of them can be easily implemented in terms of each other, when considering big-O notation, without loss of efficiency. However, one cannot implement `compare` in terms of `operator<` as efficiently as I would like. On the contrary, the converse, implementing `operator<` in terms of `compare`, can be done efficiently. Moreover, I believe this implementation to be one of the most efficient implementations of `operator<` for all practical purposes. This is because, for all practical purposes, if I need to find out whether `x < y`, I get the information whether `x == y` or `x > y` for free. In a sense `compare` is more primitive than `operator<`.

The thing that I found most surprising, is that the seemingly innocent decision to implement `operator<` in instead of `compare` to define an ordering, leads to exponential overhead. Furthermore, I find it interesting that this exponential behavior is in "the size of the composition", as opposed to "the size of the input" which is traditionally considered to be of importance.
