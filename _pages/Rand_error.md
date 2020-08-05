The [[wikipedia:Rand index|Rand index]] is a well-known measure of the similarity between two data clusterings<ref name="Rand71">{{cite journal
| author = William M. Rand
| title = Objective criteria for the evaluation of clustering methods
| journal = Journal of the American Statistical Association
| year = 1971
| volume = 66
| pages = 846–850
| number = 36
| doi = 10.2307/2284239
}}</ref>. Recently, it has been proposed as a '''measure of segmentation performance''', since a segmentation can be regarded as a clustering of pixels<ref name="Unnikrishnan2007">{{cite journal
|author = R. Unnikrishnan, C. Pantofaru, and M. Hebert
| title = Toward objective evaluation of image segmentation algorithms
| journal = IEEE Transactions on Pattern Analysis and Machine Intelligence
| year = 2007
| volume = 29
| pages = 929-944
| issue = 6
| doi = 10.1109/TPAMI.2007.1046
}}</ref>. More formally, define a segmentation as an integer-valued labeling of an image. Each object in a segmentation consists of a set of pixels sharing a common label. 

The [[wikipedia:Rand index|Rand index]] is defined as a measure of agreement: 

Given two segmentations <math>S_1</math> and <math>S_2</math> of an image <math>I</math> with <math>n</math> pixels, we define:

* <math>a</math>, the number of pairs of pixels in <math>I</math> that are in the same object in <math>S_1</math> and in the same object in <math>S_2</math> (i.e., they have the same label)

* <math>b</math>, the number of pairs of pixels in <math>I</math> that are in different objects in <math>S_1</math> and in different objects in <math>S_2</math> (i.e., they have different labels)

The Rand index, <math>RI</math>, is:
:<math> RI = \frac{a+b}{{n \choose 2 }}</math>

Here we instead define the closely related [[Rand error]], which is a measure of disagreement. The [[Rand error]] (RE) is the '''frequency with which the two segmentations disagree over whether a pair of pixels belongs to same or different objects''':

:<math> RE = 1 - RI</math>

==Implementation in Fiji==
The [[Rand error]] metric is implemented in the [[Trainable Weka Segmentation]] library. Here is an example of how to use it in a [[Beanshell_Scripting|Beanshell script]]:

<source lang=java>
import trainableSegmentation.metrics.RandError;
import ij.IJ;

// original labels
originalLabels = IJ.openImage("/path/original-labels.tif");

// proposed (new) labels
proposedLabels = IJ.openImage("/path/proposed-labels.tif");

// threshold to binarize labels
threshold = 0.5;

metric = new RandError( originalLabels, proposedLabels );
randError = metric.getMetricValue( threshold );

IJ.log("Rand error between source image " + originalLabels.getTitle() + " and target image "
+ proposedLabels.getTitle() + " = " + randError); 

</source>

==See also==
* [[Topology preserving warping error]].
==References==

<references/>

[[Category:Segmentation]]