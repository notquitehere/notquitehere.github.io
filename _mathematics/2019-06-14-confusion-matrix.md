---
title: 'Math Notes: The Confusion Matrix'
date: 2019-06-14
modified: 2019-06-14
tags: math
category: mathematics
published: true
---

A confusion matrix is a simple way to visually present the accuracy of a classification algorithm. A confusion matrix can only be constructed for values which are already known so this analysis uses your labelled evaluation set rather than your unlabelled test set[^1].

A lot of the explanations of how these work use the smallest possible matrix (that of a binary classifier) but that makes larger grids harder to comprehend, therefore it's worth looking at the theory before working through simple examples.

<table class="table table-bordered">
<tbody>
<tr>
<td class="class-sig"></td>
<td class="class-sig"><span class="math">\(A_{pred}\)</span></td>
<td class="class-sig"><span class="math">\(B_{pred}\)</span></td>
<td class="class-sig"><span class="math">\(C_{pred}\)</span></td>
</tr>
<tr>
<td class="class-sig"><span class="math">\(A_{act}\)</span></td>
<td class="true-positive"><span class="math">\(T_p\)</span></td>
<td><span class="math">\(E_{AB}\)</span></td>
<td><span class="math">\(E_{AC}\)</span></td>
</tr>
<tr>
<td class="class-sig"><span class="math">\(B_{act}\)</span></td>
<td><span class="math">\(E_{BA}\)</span></td>
<td class="true-positive"><span class="math">\(T_p\)</span></td>
<td><span class="math">\(E_{BC}\)</span></td>
</tr>
<tr>
<td class="class-sig"><span class="math">\(C_{act}\)</span></td>
<td><span class="math">\(E_{CA}\)</span></td>
<td><span class="math">\(E_{CB}\)</span></td>
<td class="true-positive"><span class="math">\(T_p\)</span></td>
</tr>
</tbody>
</table>

This is a confusion matrix for a fairly simple three class problem. It allows you to Ascertain some basic facts about how well your classifier is doing.

How to read this:

- each **row** tells you the total number of points which should be in that class
- each **column** tells you the total number of points your classifier assigned to the classic

From this you can work out:

- The **true positives** ($T_p$): those points which were classified as being in a class and actually were. These are the values on the diagonal, highlighted in green.
- The **false positives**: those points which were classified as being in a class even though they were actually in a different one. This is the sum of all the values in the classes _column_ except those which were $T_p$. For class B this would be $E_{AB} + $E_{CB}$
- The **true negatives**: those points which weren't in class and weren't classified as being part of it. This is the sum of all the values _not_ in the classes column or row. So for class C this is the the pink area in the table below.
- The **false negatives**: all the values ascribed to a different class than the one they were actually in. This is the sum of all the error values in a classes _row_ so for class A the false negative would be $E_{AB} + E_{AC}$.

<table class="table table-bordered">
<tbody>
<tr>
<td class="class-sig"></td>
<td class="class-sig"><span class="math">\(A_{pred}\)</span></td>
<td class="class-sig"><span class="math">\(B_{pred}\)</span></td>
<td class="class-sig"><span class="math">\(C_{pred}\)</span></td>
</tr>
<tr>
<td class="class-sig"><span class="math">\(A_{act}\)</span></td>
<td class="true-negative"><span class="math">\(T_p\)</span></td>
<td class="true-negative"><span class="math">\(E_{AB}\)</span></td>
<td><span class="math">\(E_{AC}\)</span></td>
</tr>
<tr>
<td class="class-sig"><span class="math">\(B_{act}\)</span></td>
<td class="true-negative"><span class="math">\(E_{BA}\)</span></td>
<td class="true-negative"><span class="math">\(T_p\)</span></td>
<td><span class="math">\(E_{BC}\)</span></td>
</tr>
<tr>
<td class="class-sig"><span class="math">\(C_{act}\)</span></td>
<td><span class="math">\(E_{CA}\)</span></td>
<td><span class="math">\(E_{CB}\)</span></td>
<td><span class="math">\(T_p\)</span></td>
</tr>
</tbody>
</table>

## Worked example: binary classifier

For a binary classifier tested on $n=175$ data points:

<table class="table table-bordered">
<tbody>
<tr>
<td class="class-sig"></td>
<td class="class-sig"><span class="math">\(A_{pred}\)</span></td>
<td class="class-sig"><span class="math">\(B_{pred}\)</span></td>
</tr>
<tr>
<td class="class-sig"><span class="math">\(A_{act}\)</span></td>
<td class="true-positive">100</td>
<td>5</td>
</tr>
<tr>
<td class="class-sig"><span class="math">\(B_{act}\)</span></td>
<td>10</td>
<td class="true-positive">60</td>
</tr>
</tbody>
</table>

- **True positives**:
    - A = 100
    - B = 60
- **False positives**:
    - A = 10
    - B = 5
- **True negatives**:
    - A = 60
    - B = 100
- **False negatives**:
    - A = 5
    - B = 10

## Resources:

- [Data School: Simple guide to confusion matrix terminology](https://www.dataschool.io/simple-guide-to-confusion-matrix-terminology/)
- [Confusion Matrix For Multiple Classes](https://youtu.be/FAr2GmWNbT0) Dr Noureddin Sadawi [Research fellow at Brunel University, London](http://people.brunel.ac.uk/~csstnns/)

[^1]: How best to divide up your data is covered in [Model Selection and Evaluation](/model-selection-and-evaluation)
