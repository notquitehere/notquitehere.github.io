---
title: "Answering common code interview questions in Python 3"
date: 2022-07-04
published: true
tags: python interviewing
category:
---

It's fairly common practice in interviews for the interviewee to be asked to present their answer to a reasonably simple coding problem in the language of their choice. As a result it can be good practice to work through some of these in preparation. The list I'm using is from Simpl¡Learn's article [Top 40 Coding Interview Questions You Should Know](https://www.simplilearn.com/coding-interview-questions-article).

Answers occasionally have explanations.

1. Reverse a string
    A string is an immutable sequence of unicode code points[^1]. Which means you can do anything you can do to an iterable to a string. Including loop through it backwards.

    ```python
    string = "Hello World!"

    print(string[::-1])
    ```

2. Determining if a string is a palindrome

    ```python
    def palindrome(string):
        return string == string[::-1]

    print(palindrome("rotator"))
    print(palindrome("not palindromic"))
    ```

3. Determine how many times each character appears in a string

    While you could use a normal dictionary for this and check if the key exists it's neater to use a `defaultdict`[^2]as you can pass an initial value to the `default_factory`.

    ```python
    from collections import defaultdict

    def count_the_occurances(string):
        occurances = defaultdict(int)
        for i in string:
            occurances += 1

        return occurances

    print(count_the_occurances("Hello World!"))
    print(count_the_occurances("rotator"))
    ```

    For an even shorter solution you can (and should) use `Counter`[^3] instead:

    ```python
    from collections import Counter

    print(Counter("Hello World!"))
    ```

4. Are these two strings anagrams?

    Remember, `str` is immutable so you can't use `.sort()` (without converting to a list first) as that sorts in place. The built in function `sorted()` on the other hand returns a new sorted list from the iterable[^4].

    ```python
    def anagram(string1, string2):
        return sorted(string1) == sorted(string2)

    print(anagram("anagram", "gramana"))
    print(anagram("anagram", "not"))
    ```

5. Count the vowels and consonants in a given string:

    Although you could theoretically do this without a regex you would need to account for all the edge cases, an issue which is avoided by calling `.lower()` on your string to remove any capitalisation and then using regex to remove all the characters that aren't in the range `a-z`.

    ```python
    import re

    def get_vowels_consonants(string):
        v, c = 0, 0
        for i in re.sub(r"^a-z", "", string.lower()):
            if (i in "aeiou"):
                v += 1
            else:
                c += 1

        return v, c

    print(get_vowels_consonants("the quick brown fox"))
    print(get_vowels_consonants("aeiou"))
    ```

6. Get the matching elements in an integer array

    ```python
    arr = [ 0, 1, 2, 3, 4, 5, 0, 2, 3 ]

    print(set([i for i in arr if arr.count(i) > 1]))
    ```

7. Implement Bubble Sort[^5]

    ```python
    def bubble(arr):
        for i in range(len(arr)):
            for j in range(len(arr) - i - 1):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]

        return arr

    print(bubble([16, 4, 8, 6, 4, 25, 6, 2, 14, 17]))
    ```

8. Implement Insertion Sort[^6]

    ```python
    def insertion(arr):
        for i in range(1, len(arr)):
            while j > 0 and arr[j-1] > arr[j]:
                arr[j-1], arr[j] = arr[j], arr[j-1]
                j -= 1
        return arr

    print(insertion([11, 0, 4, 25, 24, 18, 10, 21, 12, 8]))
    ```

9. Reverse an array

    The `list` type is mutable in python so you can use the built in `reverse()` to swap the elements in place or you can return a new list by looping backwards through the elements of the list as you would for a string reversal.

    ```python
    arr = [ 0, 1, 2, 3, 4, 5 ]

    print(arr.reverse())
    print(arr[::-1])
    ```

10. Swap two numbers without using a third variable

    If you're being asked to do this one in Python they're likely being a bit sneaky. Here's the pythonic way:

    ```python
    a = 4
    b = 6

    a, b = b, a

    print(f"a = {a}, b = {b}")
    ```

    And the mathematical way:

    - make a the sum of both numbers
    - make b the result of subtracting b from a (b is now swapped)
    - subtract b from the sum a (a is now swapped)

    ```python
    a = 2
    b = 12

    a += b
    b = a - b
    a -= b

    print(f"a = {a}, b = {b}")
    ```

11. Print a given number from Fibonacci

    This can be done with or without recursion. Clearly if they're asking you to do recursion it's because they want to you demonstrate that you understand how it works. However, if they don't give you an instruction to recurse it you might be better demonstrating without as it is computationally less expensive (if you can avoid a recursive solution you should).

    Without recursion:

    ```python
    def fib(n):
        a, b = 0, 1
        for i in range(n-1):
            a, b = b, a+b

        return b

    print(fib(10))
    ```

    With recursion:

    ```python
    def fib(n):
        if n &lt;= 1:
            return n
        return fib(n-1) + fib(n-2)

    print(fib(10))
    ```

    Or as a lambda if you are so inclined:

    ```python
    fib = lambda x : x if x &lt;= 1 else fib(x-1) + fib(x-2)

    print(fib(10))
    ```

12. N Factorial

    This is another one where there are a few possible answers and it's worth asking some follow up questions to identify what they're actually looking for.

    If you can't get a useful answer you should probably go with something like:

    ```python
    def factorial(n):
        f = 1
        for i in range(n, 1, -1):
            f *= i

        return f
    ```

    You can do this as a 1 liner using `reduce`:

    ```python
    from functools import reduce

    reduce(lambda x, y : x * y, range(1, 6))
    ```

    Or `product`:

    ```python
    from math import product

    product([i for i in range(1, 6)])
    ```

    You could use recursion:

    ```python
    def factorial(n):
        if n == 0:
            return 1
        return n * factorial(n-1)

    factorial_lambda = lambda x : 1 if x == 0 else x * factorial_lambda(x-1)
    ```

    Or, if you're feeling fairly flippant you could straight up use `factorial`:

    ```python
    from math import factorial

    print(factorial(5))
    ```

13. Reverse a linked list:

    Under the hood Python's lists are variable length arrays[^7], so to get a linked list you're going to need to use `deque`. If you get this question they're probably just checking to see if you know that `list` isn't a linked list structure and that `deque` exists.

    ```python
    from collections import deque

    linked_list = deque("abcdefg")
    linked_list.reverse()

    print(linked_list)
    ```

14. Implement Binary Search

    ```python
    def bin_search(list, item, location=-1):
        if len(list) == 0:
            return -1 // not found
        else:
            mid = len(list) // 2
            if list[mid] == item:
                return location
            else:
                if item > list[mid]:
                    return location + bin_search(list[mid:], item, mid)
                else:
                    return location + bin_search(list[0:mid], item, mid)
    ```

15. Find the second largest number in an array

    ```python
    def second_largest(arr):
        largest = max(arr)
        penultimate = min(arr)
        for i in arr:
            if penultimate &lt; i &lt; largest:
                penultimate = i

        return penultimate
    ```

    Or if you're feeling a little flippant:

    ```python
    def second_largest(arr):
        return sorted(arr)[len(arr)-1]
    ```

16. Remove all occurances of a given character from a string:

    ```python
    string = "the quick brown fox jumped over the lazy dog"
    string.replace(e, "")
    ```

17. Demonstrate inheritance

    ```python
    class Animal:
        def __init__(self, colour):
            self.colour = colour

    class Cat(Animal):
        def __init__(self, sound):
            super().__init__(colour)
            self.sound = sound

    my_cat = Cat("green", "meow")

    print(my_cat.colour)
    ```

18. Demonstrate overloading

    Python doesn't support overloading natively, you can define as many methods as you like with the same name but only the last one will work. However, you can use the `@dispatch()` decorator to mimic the behaviour.

    ```python
    from multipledispatch import dispatch

    class Animal:
        @dispatch()
        def talk(self):
            print("Hello!")

        @dispatch(str)
        def talk(self, sound):
            print(f"{sound.capitalize()}!")

        my_cat = Animal()
        my_cat.talk()
        my_cat.talk("meow")
    ```

19. Demonstrate overriding

    ```python
    class Animal:
        def hello(self):
            print("Hello World!")

    class Cat(Animal):
        def __init__(self, sound):
            super().__init__()
            self.sound = sound

        def hello(self):
            print(f"{self.sound.capitalize()}!")

    my_cat = Cat("meow")
    my_cat.hello()
    ```

20. Check if a number is prime

    For the sake of this example we're going to pretend that negative primes don't exist.

    ```python
    def is_prime(n):
        if n in [0,1]:
            return False
        elif n == 2:
            return True

        for i in range(2, (n // 2)+1 ):
            if n % i == 0:
                return False

        return True
    ```

21. Sum all the elements in an array

    ```python
    arr = [11, 26, 25, 11, 2, 23, 15, 22, 21, 23]

    print(sum(arr))
    ```

## Some additional questions you might get

1. Tell me the most common items in this list and the index of the first instance of each of them

    ```python
    from collections import Counter

    my_list = [5, 4, 2, 3, 8, 3, 9, 9, 8, 4]

    def find_frequency(my_list):
        d = Counter(my_list)

        max_vals = [ k for k,v in d.items() if v == max(d.values())]
        positions = [ my_list.index(i) for i in max_vals ]

        return max_vals, positions

    print(find_frequency(my_list))
    ```

2. Sum all the numbers in this arbitrary tree structure:

    ```python
    mytree = { "aaa": 1,
            "bbb": {
                "ccc": 1,
                "ddd": 1,
                "eee": {
                    "fff": 1,
                    "ggg": 1,
                    },
                "hhh": 1,
                }
            }

    def get_total(atree):
        if type(atree) == int:
            return atree
        elif len(atree) == 1:
            return get_total(atree[0])
        else:
            values = list(atree.values()) if type(atree) != list else atree
            return get_total(values[0]) + get_total(values[1:])

    print(get_total(mytree))
    ```

[^1]: <https://docs.python.org/3/library/stdtypes.html#text-sequence-type-str>

[^2]: <https://docs.python.org/3/library/collections.html?highlight=collections#collections.defaultdict>

[^3]: <https://docs.python.org/3/library/collections.html?highlight=collections#collections.Counter>

[^4]: <https://docs.python.org/3/library/functions.html#sorted>

[^5]: <https://en.wikipedia.org/wiki/Bubble_sort>

[^6]: <https://en.wikipedia.org/wiki/Insertion_sort>

[^7]: <https://docs.python.org/3/faq/design.html#how-are-lists-implemented-in-cpython>
