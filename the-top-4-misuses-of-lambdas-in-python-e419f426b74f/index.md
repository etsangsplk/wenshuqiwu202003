# 外卖

Lambda一直是Python初学者的硬主题之一，他们一开始就不惜一切代价避免使用它们。

一段时间后，当恐惧消失时，他们开始学习lambda，并发现自己一点也不难。 然后，它们开始使用lambda，但不幸的是，有些人可能过多地使用了lambda，导致如上所述的各种滥用。

我希望本文可以帮助解决其中一些问题。

通过避免这些误用并遵循最佳实践技巧，我敢打赌，正确使用lambda的Python代码将具有更好的可读性和可维护性。
# 4.表情太笨拙

这比以前的方法少见。 但是一些程序员只是尽可能地使用lambda来努力编写最多的Python代码。 有时需要付出一定的代价-可读性。

假设我们有一个字符串列表，我们需要使用一个奇怪的要求对它们进行排序：单词中不同元音的数量。 在sorted（）函数中使用lambda如下所示。
```
>>> texts = ['iiiii', 'bag', 'beto', 'blackboard', 'sequoia']>>> sorted(texts, key=lambda x: len(set([l for l in list(x) if l in ['a','e','i','o','u']])))['iiiii', 'bag', 'beto', 'blackboard', 'sequoia']
```

它可以按预期工作，但绝对不是最易读的代码。 下面的代码怎么样？
```python
>>> texts = ['iiiii', 'bag', 'beto', 'blackboard', 'sequoia']
>>> def number_distinct_vowels(x):
...     vowels = ['a', 'e', 'i', 'o', 'u']
...     vowels_only = [l for l in list(x) if l in vowels]
...     distinct_vowels = set(all_vowels)
...     return len(distinct_vowels)
... 
>>> sorted(texts, key=number_distinct_vowels)
['iiiii', 'bag', 'beto', 'blackboard', 'sequoia']
```

当然，我们需要再写几行代码，但是新代码难道不具有更好的可读性吗？

最佳实践4：如果lambda的表达式过于繁琐，则编写一个正则函数。
# 3.高阶函数使用不当

当我们说高阶函数时，是指可以通过将函数作为参数或通过返回函数来对其他函数进行操作的函数。

与当前主题相关的函数是map（），filter（）和reduce（），它们在lambda的许多教程中都已或多或少地被使用。 但是，这导致对lambda以及更高阶函数的某种滥用。

出于演示目的，我将在本教程中仅使用map（）函数，但相同的原理也适用于其他高阶函数。

假设我们有一个整数列表，并且希望有一个包含它们的平方的列表。 下面将lambda与map（）函数一起使用。

我们将获得一个迭代器-map（）函数中的map对象，然后将其转换为列表，我们需要在此迭代器上调用list（）函数。
```
>>> numbers = [1, 2, 3, 5, 8]>>> squares = list(map(lambda x: x * x, numbers))>>> squares[1, 4, 9, 25, 64]
```

实际上，可以通过列表理解方便地实现相同的功能-不需要高阶函数或lambda。 更加简洁易读，不是吗？

当然，掌握列表理解能力是另一个“ Pythonic功能”主题，需要另一个教程。
```
>>> numbers = [1, 2, 3, 5, 8]>>> squares = [x * x for x in numbers]>>> squares[1, 4, 9, 25, 64]
```

最佳实践3：考虑使用带列表推导的lambda替换高阶函数。
# 2.将其分配给变量

在一些教程（包括我的一些教程）中，我已经看到了将lambdas分配给变量的方法，但是它主要是向初学者展示lambdas本质上是函数。

但是，某些初学者可能已将其作为一种好习惯，并认为lambda只是声明短函数的便捷方式。 以下代码片段向您展示了这种滥用。
```
>>> divide_two_numbers = lambda x, y: x / y>>> divide_two_numbers(4, 5)0.8
```

为什么要避免这种情况？ 如果您还记得上面提到的内容，那么lambda应该只使用一次，因此没有理由将lambda分配给变量。

如果确实需要使用相关功能，则应使用def关键字来声明一个常规函数，如下所示。

如果您认为使用此简单功能的两行代码并不酷，我们可以将其重写为一行：defdivid_two_numbers_fun（x，y）：返回x / y，其工作方式相同。
```
>>> def divide_two_numbers_fun(x,y): ...     return x / y... >>> divide_two_numbers_fun(7, 8)0.875
```

避免为变量分配lambda的主要原因是出于调试/维护的目的，尤其是在生产/团队合作环境中。

让我们看一个简单的例子，可能发生的事情。 在实际情况下，事情可能会变得复杂得多。
```python
>>> divide_two_numbers(3, 0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 1, in <lambda>
ZeroDivisionError: division by zero
>>> divide_two_numbers_fun(3, 0)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 1, in divide_two_numbers_fun
ZeroDivisionError: division by zero
```

如您在上面看到的，通过常规函数的声明，我们确切地知道哪个函数导致了错误。 相比之下，使用lambda只能告诉我们存在一个lambda导致错误。

为什么没有显示功能名称？

这是因为lambda是匿名函数，所有这些函数都具有相同的名称-<lambda>。 您能想象如果您的同事发现数十个<lambda>存在错误会多么令人沮丧？
![Image by Gerd Altmann from Pixabay](1!DhWGGV529d1o0PL8Vw5MAQ.jpeg)
> Image by Gerd Altmann from Pixabay


最佳实践2：要多次使用常规函数而不是lambda。
# 1.重新发明轮子

lambdas的第一个误用是忽略了现有的内置函数。

让我们仍然以sorted（）函数为例。 假设我们有一个字符串列表，我们想使用它们的长度对它们进行排序。

当然，lambda函数lambda x：len（x）可以工作，但是直接使用内置的len（）函数怎么样？
```python
>>> pets = ['dog', 'turtle', 'bird', 'fish', 'kitty']
>>> sorted(pets, key=lambda x: len(x))
['dog', 'bird', 'fish', 'kitty', 'turtle']
# The built-in len() function
>>> sorted(pets, key=len)
['dog', 'bird', 'fish', 'kitty', 'turtle']
```

这是另一个涉及使用max（）函数的示例。
```python
>>> number_tuples = [(4, 5, 7), (3, 1, 2), (9, 4, 1)]
>>> sorted(number_tuples, key=lambda x: max(x))
[(3, 1, 2), (4, 5, 7), (9, 4, 1)]
# The built-in max() function
>>> sorted(number_tuples, key=max)
[(3, 1, 2), (4, 5, 7), (9, 4, 1)]
```

最佳实践1：在编写自己的Lambda之前先考虑一下内置函数。
# Python中Lambda的前4个错误
## 使用Python Lambda的最佳做法
![Photo by John Schnobrich on Unsplash](1!0DZieY6EVfk1bej_ohZ7Lw.jpeg)
> Photo by John Schnobrich on Unsplash


lambda被认为是非常Python的语言，是Python编程中最受欢迎的功能之一。 如此之多，以至于许多Python程序员都想尽可能地使用它们。

当然，lambda具有使我们的代码简洁的优势，但是在项目中过度使用它们会导致滥用，从而降低我们代码的可读性和可维护性。

在开始研究这些误用是什么之前，让我们先快速回顾一下lambda。 如果您对它们非常了解，则可以跳到下一部分。

Lambda，也称为lambda函数，是匿名函数，可以接受任意数量的参数，而只有一个表达式。 它们的声明由lambda关键字表示。 基本语法如下。
```
lambda arguments: expression
```

Lambda最适合需要小的功能且仅使用一次的地方。 lambda的一种常见用法是将其设置为内置sorted（）函数中的关键参数。 这是一个例子。
```
>>> students = [('Mike', 'M', 15), ('Mary', 'F', 14), ('David', 'M', 16)]>>> sorted(students, key=lambda x: x[2])[('Mary', 'F', 14), ('Mike', 'M', 15), ('David', 'M', 16)]# The students are sorted by age
```

许多教程在解释lambda是什么以及可以在哪里使用lambda方面做得很好，因此，没有充分的理由在这里重复大量的讲解。

相反，本文的目的是向您展示最常见的lambda误用，以便在以下所列情况以外的其他情况下使用lambda时，您可能会正确使用它们。
```
(本文翻译自Yong Cui, Ph.D.的文章《The Top 4 Misuses of Lambdas in Python》，参考：https://medium.com/better-programming/the-top-4-misuses-of-lambdas-in-python-e419f426b74f)
```
