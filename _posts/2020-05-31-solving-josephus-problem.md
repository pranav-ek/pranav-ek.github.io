---
layout: post
title: Solving Josephus problem 
date: '2020-05-23 15:48:52'
tags: 
 - Python
 - Math
 - Algorithm
 - Datastructure
---

How many of you had an aha moment while watching a video on YouTube? This blog post is based on one of my aha moment.

### Josephus problem

Few soldiers standing in the circle to be executed. The counting begins at a particular point and proceeds around the circle in a fixed direction. A set of soldiers will be executed, and few will be skipped, this process will continue until only one soldier remains, and he is freed.

The task here is to find a position in the initial circle to avoid the execution.

Following is the video which explains it in detail with some examples.

<iframe width="560" height="315" src="https://www.youtube.com/embed/uCsD3ZGzMgE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

 My 'aha!' moment was when I realized that this problem can be easily solved using binary numbers. 
 
 We are going to solve this problem in Python using Deque, and later with the binary number.

Say, the total number of soldiers is `7`, and the number to skip is `2`, which means the soldier `1` is skipped and `2` will be executed. The next soldier to executed will be `4` and so on. The soldier `7` will survive. Try it using a pen and paper.

#### Solution using Deque

The [Deque](https://docs.python.org/2/library/collections.html#collections.deque) contains elements from [1,8). We need to keep on eliminating soldiers until the number of soldiers becomes one, this is accomplished by using the rotate and pop function. 

```python
from collections import deque

soldiers = deque([*range(1,8)])

while len(soldiers) > 1:
    soldiers.rotate(-2)
    print('Eliminated ', soldiers.pop())
print('The survivor is ' , soldiers.pop())
```

If you notice we are passing an argument to the rotate function, it rotates the deque by the number specified. Using a negative number will rotate the deque to the left. The second element in the original list will become the last element after rotation.

 The pop function will be removing the elements from the right end. As the list is rotated, the last element will be the person who will get eliminated first. This process will continue until the while loop ends.
 
 The last person in the list is the survivor.

#### Solution using Binary 

Unlike the Deque method, this solution will directly get into the answer. If the number of soldiers is seven, then this will tell us where you need to sit to avoid getting killed.

```python
number_of_soldiers = bin(7)
survivor = number_of_soldiers[3:] + number_of_soldiers[2] 
print('The survivor is ',int(survivor,2))
```

We are converting the number to its [binary](https://en.wikipedia.org/wiki/Binary_number) form using the bin function. If you print the binary, it will be like `0b111`. Did you notice the `0b` in the string? we are not interested in that.

To make this method work, we need to get the first binary value in the string and append it to the last. 

That's what we are doing with `number_of_soldiers[3:] + number_of_soldiers[2] `. The `number_of_soldiers[3:] ` will return `11`, by removing `0b1` from it. Then we are appending the value of the second index `1` to the string.

While printing the survivor, we are converting the binary to an integer. The second argument in the int function is the base. We are saying the [int](https://docs.python.org/3.4/library/functions.html?highlight=int#int) function that the input is in binary (base 2) convert it to integer for me.





  
