---
layout: post
title: "Quicksort Algorithm in Javascript"
date: 2013-02-14 18:49
comments: true
categories: javascript algorithms sorting
---

Here's a basic Quicksort algorithm. You can call this code using Node.js 


{% codeblock lang:javascript %}
	var Sort = require('./quicksort.js');
	list = [3, 4, 5,2, 5];
	var sorted = Sort(list, function(a, b) {
		if (a > b) {
			return 1;
		} else if (a === b) {
			return 0;
		} else {
			return -1;
		}
	});
{% endcodeblock %}

have fun and hope this proves useful to somebody!


{% codeblock lang:javascript %}
//quicksort.js
module.exports = function(list1, compareFunction) {
    var list = list1;
    var cmp = compareFunction;

    var swap = function(i, j) {
        /* swaps the indexer A[i] and A[j] */
        var _i = list[i];
        var _j = list[j];
        list[j] = _i;
        list[i] = _j;
    }

    var partition = function(p, r) {
        /* the partition function goes through the array
         *  list[
         */
        var r = r;
        var i = p; //on intiial pass, p is 0

        /* for each entry i where A[i] is less than A[r-1]
         * where r-1 is the index of the Penultimate item
         */
        //from j = 0 to j = r-1
        //exchange A[i] and A[j] if A[j] > A[r]
        for (var j = p; j < r; j++) {
            //console.log("lets try this ", list);
            //console.log("i : " + i + " j : " + j);
            if (cmp(list[j],list[r]) <= 0 ) {
                //swap them!! and then increment i
                swap(i, j);
                i++;
            }
        }

        swap(i, r);
        return i;


    }

    var quickSort = function(p, r) {
        if (p < r) {
            q = partition(p, r);
            quickSort(p, q - 1);
            quickSort(q + 1, r)

        }

    }

    var initialize = function() {
        quickSort(0, list1.length-1);

    }
    //compute the sorted value
    initialize();

    return list1;

}
{% endcodeblock %}
