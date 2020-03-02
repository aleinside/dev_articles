# Comprehension: a short comparison between Python and Elixir

It is known that Python is a very concise programming language. 

In this article I want to focus on one of the most powerful features of Python, the *list comprehension*, and in particular on its comparison with the *for comprehension* in Elixir. As we will see, with some shrewdness, Elixir can be almost as concise as Python, at least when using a comprehension.

### Substring example
Let's see a first example. Suppose that given a string of **N** digits from 0 to 9 as input, we want to calculate the maximum sum of the digits of all substrings of length ceil(**N**/2).
For example given the string "09121", the substring are "091", "912" and "121", so we are looking for max(10, 12, 4) and we get 12 as a result. It is useful to note that the number of substring in which we are interested is floor(**N**/2 + 1).

Let's translate this in Python. We use a step by step approach checking the intermediate results in the shell. First of all we want to iterate on all the substrings:

```python
>>> input = '09121'
>>> N = len(input)
>>> L = math.ceil(N / 2)                # length of substrings
>>> K = math.floor(N / 2 + 1)           # number of substrings
>>> [input[i:i+L] for i in range(K)]
['091', '912', '121']
```

This is the first comprehension. Now we want to get the sum of the digits of every substring, and to do this we first need to convert each substring to the list of digits that it contains. We will compose our first comprehension with another one:
```python
>>> [[c for c in input[i:i+L]] for i in range(I)]
[['0', '9', '1'], ['9', '1', '2'], ['1', '2', '1']]
>>> [[int(c) for c in input[i:i+L]] for i in range(K)]
[[0, 9, 1], [9, 1, 2], [1, 2, 1]]
>>> [sum([int(c) for c in input[i:i+L]]) for i in range(K)]
[10, 12, 4]
```
The final step is getting the max of the sums. 
```python
>>> max([sum([int(c) for c in input[i:i+L]]) for i in range(K)])
12
```

In the end we get this very concise expression:

`max([sum([int(c) for c in input[i:i+L]]) for i in range(K)])`

Let's see what we can do in Elixir:

```elixir
iex(1)> input = "09121"
"09121"
iex(2)> l = ceil(n/2)       # length of substrings
3
iex(3)> k = floor(n/2+1 )   # number of substrings
3
iex(4)> for i <- 0..k-1, do: String.slice(input, i, l)
["091", "912", "121"]

```
Let's proceed exactly as we did before:
```elixir
iex(5)> for i <- 0..k-1, do: String.graphemes(String.slice(input, i, l))
[["0", "9", "1"], ["9", "1", "2"], ["1", "2", "1"]]
iex(6)> for i <- 0..k-1, do: for c <- String.graphemes(String.slice(input, i, l)), do: String.to_integer(c)
[[0, 9, 1], [9, 1, 2], [1, 2, 1]]
iex(7)> for i <- 0..k-1, do: Enum.sum(for c <- String.graphemes(String.slice(input, i, l)), do: String.to_integer(c))
[10, 12, 4]
iex(8)> Enum.max(for i <- 0..k-1, do: Enum.sum(for c <- String.graphemes(String.slice(input, i, l)), do: String.to_integer(c)))
12
```

Too reduce the noise a little bit we can import the functions from String and Enum explicitly:
```elixir
iex(9)> import String, only: [graphemes: 1, slice: 3, split: 3, to_integer: 1]
String
iex(10)> import Enum, only: [max: 1, sum: 1]
Enum
iex(11)> max(for i <- 0..k-1, do: sum(for c <- graphemes(slice(input, i, l)), do: to_integer(c)))                               
12
```
Note that we could not import the entire `String` and `Enum` modules because we would get a CompileError when trying to use the `slice/3` function which is defined in both modules.

One feature that we can exploit here is the pipe operator `|>` of Elixir. Using it we can actually write the expression in the order that we constructed it: 
```elixir
iex(12)> (for i <- 0..k-1, do: (for c <- input |> slice(i, l) |> graphemes(), do: to_integer(c)) |> sum()) |> max()
12
```

I would say that the Python version is still more readable and easier to understand, but we got close to it with the Elixir version.

#### Differences
- While in Python we could iterate directly over a string, in Elixir we had to use the graphemes function from the String module to get the list of characters to iterate on.
- The slice syntax of Python is a little bit more concise because there is no need to call a function. 
- The Elixir syntax for the range is a little bit more concise. Note that while range are exclusive in Python, they are inclusive in Elixir.
- The order of the for comprehension is the opposite.
- Variables should start with a lower case letter in Elixir
- To make the code more readable we had to import the external function one by one which is not very convenient

### Filtering example
Suppose that in the previous example we are not sure what is the number of substring of length L. In this case we can just using the filter features of our comprehensions.

Removing the use of K, in Python we would get:
```python
>>> [input[i:i+L] for i in range(N)]
['091', '912', '121', '21', '1']
```
Now we can filter the substring of length L
```python
>>> [input[i:i+L] for i in range(N) if len(input[i:i+L]) == L]
['091', '912', '121']
```

So the entire code becomes:
```python
max([sum([int(c) for c in input[i:i+L]])
     for i in range(N) if len(input[i:i+L]) == L])
```

While in Elixir we would get:
```elixir
iex(13)> for i <- 0..n, do: String.slice(input, i, l)  
["091", "912", "121", "21", "1", ""]
iex(14)> for i <- 0..n, String.length(slice(input, i, l)) == l, do: slice(input, i, l)
["091", "912", "121"]

```
and here we had to use the module prefix for the length function because it is ambiguous, since the Kernel module also exposes a length function for lists.

In both languages we got a repetition, the argument to the function calculation the length of the string is the same code we use in the result of the comprehension.
In elixir we can bypass this issue by assigning the value to a variable and refer to the variable later:
```elixir
for(
  i <- 0..n,
  String.length(sub = slice(input, i, l)) == l,
  do: for(c <- graphemes(sub), do: to_integer(c)) |> sum()
) |> max()
```
As we can see we introduced an intermediate variable `sub` to remove the code duplication.

#### Differences
- At this point the Python code is still more concise but maybe not so readable anymore.
- We couldn't remove the duplication in the Python expression as we did in the Elixir one

### Idiomatic way in Elixir
Just for completeness I want to show the preferred way for Elixir programmers, one that we would find in production code.
```elixir
import Enum, only: [filter: 2, map: 2,, max: 1, sum: 1]

0..n
|> map(&slice(input, &1, l))
|> filter(&(String.length(&1) == l))
|> map(fn sub -> sub |> graphemes() |> map(&to_integer(&1)) |> sum() end)
|> max()
```
Here we see the use of map and filter functions to which we pass a lambda function, that in Elixir has two different forms, we have the concise form with *function capturing* that employs the `&` operator and the more extended form with *anonymous functions* that uses the `fn ... -> ... end` construct. Not that in the second map argument we had to employ both forms, because it is not valid syntax to have two nested function capturing (the reason is quite obvious, the parameter are not named so you would have ambiguity).
One thing that I don't like so much of this code is the use of nested map functions.

### Conclusion
In this article I wanted to explore the Elixir *for comprehension* which is not seen so often in production code due to the preferred use of map and filter functions, and show how close it is to the *list comprehension* in Python which instead is usually a best practice and preferred over the use of map and filter. As we saw in the last example the Elixir version is even a little bit more general, but the Python version is of course more concise.
