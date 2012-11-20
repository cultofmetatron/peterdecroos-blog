---
layout: post
title: "sorting algorithms in python - part 1"
date: 2012-09-10 00:11
comments: true
categories: algorithms sorting python
---

I'm in a couple of weeks into Robert Sedgewick's class on algorithms currently
running on [coursera](https://www.coursera.org/course/algs4partI). To round off the weekend
I decided to reimpliment some of the classic sortting algorithms in python.

The compare function

{% codeblock lang:python %}
    def cmp(a, b):
        if (a == b):
            return 0
        elif (a > b):
            return 1
        else:
            return -1

	def swap(i, j, array):
	    swapv = array[i]
	    array[i] = array[j]
	    array[j] = swapv
	    return array

	def biggest(array):
	    i = 0
	    biggest_val = array[0]
	    biggest_index = i
	    while(i < len(array)):
	        if (array[i] > biggest_val):
	            biggest_val = array[i]
	            biggest_index = i
	        i = i + 1
	    return (biggest_index, biggest_val)

	def minv(array, start_index):
	    i = start_index
	    min_val = array[start_index]
	    min_index = i
	    while(i < len(array)):
	        if (array[i] < min_val):
	            min_val = array[i]
	            min_index = i
	        i = i + 1
	    return (min_index, min_val)

	def min(array):
	    return minv(array, 0)

{% endcodeblock %}
Selection Sort
{% codeblock lang:python %}

	def selectionSort(array):
	    i = 0
	    array1 = array
	    while (i < len(array1)):
	        array1 = swap(i, minv(array1, i)[0], array1)
	        i = i + 1
	    return array1
{% endcodeblock %}
Insertion sort is just a specialized case of shellsort so I created a base composite function that encapsulates the core of both algorithms.
{% codeblock lang:python %}

	def gapSort(array, gap):
	    """helper function to aid insertion sort and shell sort"""
	    i = 0
	    array1 = array
	    while (i < len(array)):
	        if (i > 0):
	            j = i
	            while ((cmp(array1[j], array1[j-gap]) < 0) and (j != 0 ) ):
	                #while object at i is less than the one before it
	                swap(j, j-gap, array1)
	                j = j - 1
	        i = i + 1
	    return array1

	def insertionSort(array):
	    return gapSort(array, 1)



	def shellSort(array):
	    """sorts using the shellsort algorithm"""
	    vals = [3*h+1 for h in range(len(array)/3)][::-1]
	    for val in vals:
	        array = gapSort(array, val)
	    return array

{% endcodeblock %}
Finally, Merge sort; Running in N log(N), it is the only algorithms other than Quicksort
worth using on large datasets.

{% codeblock lang:python %}

	def merge(array, p, q, r):
	"""The merge function"""
	    if ((r - p) > 1):
	        left = array[p:q+1]
	        loggr("left" + str(left))
	        right = array[q+1:r+1]
	        loggr("right"+ str(right))
	        left.append('c')
	        right.append('c')
	        i = 0
	        j = 0
	        for k in range(p, r+1):
	            if left[i] <= right[j]:
	                array[k] = left[i]
	                i = i + 1
	            else:
	                array[k] = right[j]
	                j = j + 1
	    elif ((r - p) == 1 ):
	        if (array[r] < array[p]):
	            i = array[p]
	            j = array[r]
	            array[p] = j
	            array[r] = i



	def mergeSort(array):
	    def sort(p, r, msg):
	        if p < r:
	            q = (p+r)/2
	            if (r - p) > 1:
	                sort(p, q, "in left array")
	                sort(q+1, r, "right array")
	            merge(array, p, q, r)
	    sort(0, len(array)-1, "root")
	    return array

{% endcodeblock %}

Quick sort and its probabilistic guarantee of fast enough run time strikes me as the most
mathematically perverse form of black magic. Beautiful in the inherent underlying fabric
of its utility. I'll cover that when I get to part 2.




