---
autogenerated: true
title: Colocalization Analysis
breadcrumb: Colocalization Analysis
layout: page
categories: Cookbook,Tutorials,Colocalization,Color processing
description: test description
---

{% include biginfo-box content='See [:Category:Colocalization](_Category_Colocalization ) for pages about colocalization.' %} {% include learn content='techniques' %}

## What is colocalization?

Suppose you are given some images by a colleague, or have some images of your own, and you want to measure the amount of colocalisation between two of the dyes or stains in the images. First you have to define what you mean by colocalisation, and that is not trivial. For one place to start reading about colocalisation and for how to correctly capture quantitative fluorescence microscopy images suitable for colocalisation analysis, look here: [Image Processing Courses at BioDIP, Dresden](http://www.biodip.de/wiki/Teaching_Material#IPF_Image_Processing_Courses). A recent review is [by Dunn in 2011](http://ajpcell.physiology.org/content/300/4/C723).

### Methods of colocalization analysis

#### Pixel intensity spatial correlation analysis

Here just are two of many colocalization coefficients to express the intensity correlation of colocalizing objects in each component of a dual-color image:

1.  **Pearson's correlation coefficient.** It is not sensitive to differences in mean signal intensities or range, or a zero offset between the two components. The result is +1 for perfect correlation, 0 for no correlation, and -1 for perfect anti-correlation. Noise makes the value closer to 0 than it should be.
2.  **Manders split coefficients.** Proportional to the amount of fluorescence of the colocalizing pixels or voxels in each colour channel. You can get more details in [Manders et al.](Media_Manders.pdf ) Values range from 0 to 1, expressing the fraction of intensity in a channel that is located in pixels where there is above zero (or threshold) intensity in the other colour channel.

These coefficients measure the amount or degree of colocalization, or rather correlation and co-occurrence respectively (but should not be expressed as % values, because that is not how they are defined). But if there is nothing to compare them to, what do they mean? A statistical significance test was derived by Costses to evaluate the probability that the measured value of Pearson's correlation, r between the two colour channels is significantly greater than values of r that would be calculated if there was only random overlap of the same information. This test is performed by randomly scrambling the blocks of pixels (instead of individual pixels, because each pixel's intensity is correlated with its neighboring pixels) in one image, and then measuring the correlation of this image with the other (unscrambled) image. You can get more details in [Costes et al.](Media_Costes_etalColoc.pdf ) The result of this tests tell us if the Pearsons r and split Manders' coefficients we measure are better than pure chance or not.

Other coefficients include ranked correlations such as Spearman and Kendal's Tau, Li's ICQ and the % of intensities or area (volume) above one or both thresholds (coming soon to Coloc\_2). Some others are described in the literature that have been used in publications, but that have been refuted as insensitive, such as the overlap coefficient from the Manders paper, which [J. Adler et al.](Media_Adler_et_al-2010-Cytometry_Part_A.pdf ) showed to have large problems in interpretation compared to Pearson's r and Manders' split coefficients.

Other methods include ICCS (image cross correlation spectroscopy) and a derivative of that called [PPI](http://www.anes.ucla.edu/~wuyong/PPI/index.html) (protein proximity index, [original article](Media_Wu2010-crosscorrelationPPI_Coloc.pdf )).

#### Object-based overlap analysis

This cookbook does not cover object-based overlap analysis, since it requires segmenting the image into objects and background, and that is a whole branch of image processing in itself. See the [JACoP](JACoP ) imageJ plugin for object based methods.

#### Do state the spatial resolution explicitly\!

The {% include wikipedia title='Pauli exclusion principle' text='Pauli exclusion principle'%} states that two particles can not have the same quantum numbers so they can not be in the same place. So, actually nothing is "really" colocalised. At the other extreme, a universe of one voxel (not cubic of course) is completely colocalised - everything is inside it. Practically, our situation lies between the two extremes. We must colocalise at some defined and explicit spatial scale: In our case the optical resolution or image pixel spacing, whichever is the larger value in nm, micrometers, mm, meters, km, etc. The colocalization measurement we make only means anything in relation to the spatial scale we are working at, so it needs to be explicitly stated. You can find more details about optical resolution and image pixel spacing in the following sections.

## Why scatter plots instead of colour merge images?

Far too often, composite/merge images of red and green channels are considered sufficient to demonstrate colocalisation. This is plain wrong. The problems with red/green merge images for colour-blind people aside, there is another very good reason to require scatter plots: the perception of human eyes and brain can be fooled very easily. Just have a look at this image:

![SpiralsRGY.png](/images/pages/SpiralsRGY.png "SpiralsRGY.png")

Most people might think that the image contains 4 distinct colours: 2 sets of thin spirals are in dark red and dark green, and 2 thick prominent spirals of yellow-green and yellow. However, the yellow and yellow-green actually have <b>exactly the same color\!</b> You can verify this yourself by calling {% include bc content='File | Open Samples | [Spirals (Macro)](Spirals_Macro )'%} in Fiji.

![GreenYellowSimultColourIllusion.png](/images/pages/GreenYellowSimultColourIllusion.png "GreenYellowSimultColourIllusion.png")

Here is another one. The two circles look like different colours, but they are exactly the same if you measure the pixel values.

So... now, how do you feel about determining colocalization by looking for yellow blobs? Doesn't make much sense does it? We notice that the shades and hues of colours look different according to what other colours they are next to\! So, you need to measure something from the pixel values, not simply subjectively "look at" a red/green colour merge image.

An even better reason to always look at scatterplots / 2D histograms / cytofluorograms is that they actually show the thing you are looking for and talking about - the correlation (or not) between the intensities of the 2 colour channels of the pixels over space.

![Ch1coloc.gif](/images/pages/Ch1coloc.gif "Ch1coloc.gif") ![Ch2coloc.jpg](/images/pages/Ch2coloc.jpg "Ch2coloc.jpg") ![Scatterplotcoloc.jpg](/images/pages/Scatterplotcoloc.jpg "Scatterplotcoloc.jpg") ![ColocQuadrants.jpg](/images/pages/ColocQuadrants.jpg "ColocQuadrants.jpg")

In the scatterplot or 2D Histogram (Thanks Tony Collins for this nice figure) the two intensity values for each pixel or voxel are plotted against each other, and the brighter the colour, the more pixels or voxel have those two intensity values for their two colour channels. Here we see if there is correlation immediately by eye, in the presence of a cloud of information in the middle of the 2D histogram. We can fit that cloud with a linear regression and measure correlation coefficients.

After setting thresholds in both colour channels, we see the scatterplot or 2D Histogram is split into 4 areas, quadrants. The contents of each can be used to calculate different colcoalization results.

## A sample dataset

Let's open a sample data set that we know should have very good colocalization because the 2 subunits of a dimeric protein are stained with green and red dyes respectively. The methods of Pearson, Manders, Costes and Li should work very well for this sample, but maybe we can see some problems with the data? Maybe we can decide if the data is suitable for this analysis or not?

**Open this sample data file [colocsample1bRGB\_BG.tif](https://fiji.sc/samples/colocsample1bRGB_BG.tif).** Then use the "Image-Color-Split Channels" menu command to get a separate z stack for the 2 dyes (you can throw the blue one away\!).

If you like, you can change the look up tables of the images (LUTs) so one is "green" and one is "magenta". Of course the colors here are always false. These false colors are only useful to tell which channel is which. The optoelectronic detectors we use only see photons, and don't know what color they are; that is determined by the fluorescence emission filters we use. There is no such thing as a green dye or a red dye, since they have broad emission spectra not a single wavelength corresponding to a certain "color". If I want to show DAPI in green and EGFP as magenta, there is nothing "wrong" about that.

<img src="/images/pages/SplitChannels.png" width="300"/>

## ImageJ plugins for colocalization analysis

There are several plugins available for performing colocalization analysis. In addition to the options described below, see also the [index of pages related to colocalization](_Category_Colocalization ).

### Coloc 2

Coloc 2 implements and performs the pixel intensity correlation over space methods of {% include wikipedia title='Pearson product-moment correlation coefficient' text='Pearson'%}, [Manders](Media_Manders.pdf ), [Costes](Media_Costes_etalColoc.pdf ), [Li](Media_LietAlColoc.pdf ) and more, for scatterplots, analysis, automatic thresholding and statistical significance testing.

None of this gives sensible results unless you have your imaging hardware set up appropriately and have acquired images properly, and have performed appropriate controls for bleed-through and chromatic shift etc. See [here for hardware set up guidelines](Colocalization_-_hardware_setup_and_image_acquisition ).

This plugin supersedes the [Colocalization Threshold](Colocalization_Threshold ) and [Colocalization Test](Colocalization_Test ) plugins, which unfortunately were buggy and hard to maintain. So we started from scratch with a carefully planned and designed new plugin. While the old plugins are described below as well, we recommend that you use Coloc 2 instead.

One main feature of Coloc 2 is the standardised PDF output, which is intended to make the results of different colocalization experiments comparable.

Please see the [Coloc2](Coloc2 ) page for complete instructions on using the Coloc 2 plugin, including common pitfalls of the pixel intensity spatial correlation methods that it employs.

### JaCoP

[JaCoP](JaCoP ) is a compilation of co-localization tools:

  - Calculating a set of commonly used co-localization indicators:

<!-- end list -->

  -   - Pearson's coefficient
      - Overlap coefficient
      - k1 & k2 coefficients
      - Manders' coefficient

  - Generating commonly used visualizations:
    
      - Cytofluorogram

  - Having access to more recently published methods:
    
      - Costes' automatic threshold
      - Li's ICA
      - Costes' randomization
      - Objects based methods (2 methods: distances between centres and centre-particle coincidence)

All methods are implemented to work on 3D datasets.

See the [JaCoP](JaCoP ) page for full details.

### Colocalization Finder

The Colocalization Finder plugin displays a correlation diagram (called scatterPlot picture) from two initial pictures having the same size together with a RGB overlap of the original images (called Composite picture).

See the [Colocalization Finder web page](http://punias.free.fr/ImageJ/colocalization-finder.html) for further details.

## Precautions and notes

### Check image data for problems and suitability for analysis

Questions you should ask before attempting colocalisation analysis from 2 colour channel images, using the pixel intensity spatial correlation methods of Manders and Costes:

1.  Is the image data noisy?
2.  Has there been lossy compression?
3.  Is the intensity information saturated / clipped / overexposed?
4.  Is there a problem with uniform background / detector offset?
5.  What is the spatial resolution?

To begin with, we should check the images for problems that might make the colocalisation analysis methods fail or be unreliable.

1.  Significant noise (uncertainty in the pixel values - usually from detection of too few photons) means the methods we will use will significantly underestimate the true colocalization, or even completely fail to give the "right" result.
2.  Lossy compression messes up the intensity information of the pixels, causing the colocalization result to be more or less wrong.
3.  Intensity clipping/saturation is bad news. Pixel intensity correlation measurements rely on the pixel intensities being true and not clipped to 255 when they were really higher\! See the [Detect Information Loss](Detect_Information_Loss ) tutorial for details on how to detect such problems.
4.  We should look for wrong offset / high background (since this confuses the auto threshold method, since zero signal is not zero pixel intensity but some larger number)
5.  Spatial resolution

The spatial resolution of the images, by definition, determines the spatial resolution that you are measuring colocalization at. The results will be different at different levels of spatial resolution. If the image is one big pixel, everything will colocalize\!

The spatial resolution of the light microscope is limited by the wavelength of the light (and the NA of the objective lens) according to Ernst Abbe's work. Molecules / proteins are an order of magnitude smaller than the wavelength of visible light, so they could be many nm apart, but still appear in the same image pixel. Is that true colocalization? Maybe, but it depends how you define it\!

Are the images spatially calibrated? If not then we need to calibrate them so we know the spatial sampling rate (think pixel or voxel size) in x, y and z. See the [SpatialCalibration](SpatialCalibration ) tutorial for how to do that. We need the images to be spatially calibrated in order for the Costes statistical significance test (below) method to work properly.

We need to think carefully about the correct or adequate spatial resolution in x, y and probably z. This depends on the {% include wikipedia title='Numerical aperture' text='Numerical Aperture'%} of the objective lens. You can calculate the correct pixel or voxel sizes for the objective lens you are using, to get the maximum resolution that that objective lens can really see: [Nyquist Calculator](http://support.svi.nl/wiki/NyquistCalculator)\]. Essentially the pixels / voxels should be about 3 times smaller then the resolution of the lens. On the other hand, if you are only interested in larger objects, and not the smallest details the objective can see, it makes sense to have larger pixels or voxels. Again, these should be about 3 times smaller than the smallest feature you want to resolve.

{% include wikipedia title='Nyquist–Shannon sampling theorem' text='Nyqvist'%} tells us the spatial sampling should be about three times smaller then the smallest object we want to resolve. Remember, spatial intensity correlation analysis, as we will perform here, can not tell you that 2 proteins are bound together in some biophysical bonding interaction. However, it might suggest that the 2 molecules occur with the same relative amounts when they are present in the set of spatial samples (pixels or voxels) with intensities above the thresholds we will calculate below. In any case, it might be a hint that "maybe they are binding partners or in the same macromolecular complex". You should follow up by determining true binding using {% include wikipedia title='FLIM' text='FLIM'%}, {% include wikipedia title='Förster resonance energy transfer' text='FRET'%} and biochemical methods like Immuno co-Precipitation etc.

## Further reading

  - [A practical guide to evaluating colocalization in biological microscopy](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3074624/?report=printable).

## Older colocalization plugins

### Colocalization Threshold

<span style="color: red">Note: this plugin is no longer under active development and support. Use [Coloc 2](Coloc_2 ) instead, which does the same thing, only better.</span>

The [Colocalization Threshold](Colocalization_Threshold ) plugin performs several functions for you in one go. With the "green" and "red" stacks of the [colocsample1bRGB\_BG.tif](https://fiji.sc/samples/colocsample1bRGB_BG.tif) dataset open and the channels split (see above) choose the menu item "Analyze-Colocalization-Colocalization Threshold". Next select the right stacks for the analysis in Channel1 and Channel2. You can use a region of interest (ROI) if you like, which should be defined before you run the plugin. Check on "Show Colocalized Pixels" and "Show Scatter Plot" (see also [Why scatter plots?](#Why_scatter_plots.3F "wikilink")), and others off. You can explore the options in Set options. Turn ALL the options on the first time you use it, so you see what it can do.

<img src="/images/pages/Coloc1.png" width="300"/> <img src="/images/pages/Coloc1b.png" width="300"/> <img src="/images/pages/Coloc2.png" width="300"/> \<\\p\>

1.  It generates a 2D Histogram / Scatterplot / Fluorogram. this is a really good way to visualise the correlation of the pixel intensities, over all pixels/voxels in the image, and can tell you immediately about problems such as intensity saturation/clipping, wrong offset, emission bleedthrough (fluorescence signal from the wrong dye in the detection channel), and even if there are multiple populations of colocalising species with different ratios of dyes in the same sample. Think of it just like a FACS or Flow cytometry scatter plot; indeed it is very similar.
2.  It makes a linear regression fit of the data in the scatter plot. That is the diagonal white line in the scatter plot, the gradient of which is the ratio of the intensities of the 2 channels.
3.  It does the Costes method auto threshold determination. The thresholds are the intensity levels above which for both channels you say the two dyes are "colocalised". This method uses an iterative procedure to determine what pair of thresholds for the 2 channels of the scatterplot give a {% include wikipedia title='Correlation' text='Pearson\'s correlation coefficient (r)'%} of zero for the pixels below the thresholds. That means that all the pixels which have intensities above the two thresholds have greater than zero correlation, and the pixels below the thresholds have none or anti correlated intensities. The method is pretty robust (so long as you don't stupidly defeat it, i.e. with image data with high offsets / background), and is fully reproducible, meaning you will always get the same thresholds for the same data set, and similar thresholds for similar datasets. Threshold setting is a big problem in colocalisation analysis. If you use a tool that allows you to manually set the thresholds, obviously you can get any result you like, since you are subjectively deciding what is colocalised and what isn't. This might please your boss, but it's not very scientific is it? So don't do that\! Use the Costes auto threshold instead\! Some people say the Costes method sets the thresholds too low, and lower than they would set them by eye. That might be true, but manual methods are subjective and totally unreliable. The thresholds that Costes method sets always mean the correlation below the thresholds is zero. That's a good thing.

<li>

The plugin finally sends a bunch of statistics and results to the results window. You need to turn them all on using the "set options" checkbox of the plugin GUI. Some of these results are pretty uninformative. They are listed here, in arguably order of usefulness:

1.  The thresholded Mander's split colocalisation coefficients (zero is no colocalisation, one means perfect colocalisation. There is one coefficient per channel, which tells you the proportion of signal in that channels that colocalises with the other channel. Of course this might be different for the two channels\! The thresholded Mander's coefficients are probably the numbers you would publish (not the Pearson's coefficients as these are less informative).
2.  The thresholds that were set by the Costes Auto threshold method.
3.  The Pearson's coefficients for: the whole image, image above thresholds (should be close to 1 for very good colocalizing dyes) and image below thresholds (should be around zero, because that's how the Costes auto threshold method tries to set them... so if it is not close to zero, something went wrong\!)
4.  Linear regression solution: If the image data are close to ideal, with comparable mean intensity in both channels and no problems exist with offset/background, then by definition, the gradient of the regression line will be close to unity (1) and the intercept will be close to zero. These are both good things for the quality of the result. This information might be less useful in other situations, e.g. where there is only sparse data in one channel, or there are multiple colocalising populations of signals (several clearly independent patches/blobs in the scatterplot).
5.  The % volume and intensity colocalised values are pretty useless, unless you use a biologically relevant region of interest, as they are very dependent on how much stuff and how much background there is in a certain image, and you will get totally different values for different images of the same sample, that happen to have different areas of background. In images that have no empty regions, like tissue samples, it might then be a useful number. Maybe % intensity above threshold colocalised might be a useful value in some cases? See section 6.3 at http://www.uhnresearch.ca/facilities/wcif/imagej/colour_analysis.htm for more explanation of what the different measurements exactly are, or look at the code.

### Colocalization Test

<span style="color: red">Note: This plugin is no longer actively developed or supported. Use [Coloc 2](Coloc_2 ) instead, which does the same thing, only more correctly, and as described in the original publication by Costes, instead of making a nasty assumption and shortcut.</span>

The [Colocalization Test](Colocalization_Test ) plugin performs the Costes test for statistical significance (which you should ALWAYS do after calculating the thresholded Manders coefficients and the scatterplot). It is in the menus at {% include bc content='Analyze | Colocalization | Colocalization Test'%}

<img src="/images/pages/ColocTestGUI1.png" width="400"/>

Choose the correct Channel 1 and Channel 2 images stacks from the drop down lists. Make sure "Current Slice Only" is off, and "Keep Example Randomized Image" and Show All R values" are on. Then click "OK"

<img src="/images/pages/ColocTestGUI2.png" width="200"/>

The results window will then display the calculated P-value, and some other details of the test calculation.

<img src="/images/pages/ColocTestResult.png" width="400"/>

The Costes method for {% include wikipedia title='Statistical significance' text='Statistical Significance'%} relies on the spatial calibration of the image, knowledge of the {% include wikipedia title='Numerical aperture' text='Numerical Aperture (N.A.)'%} of the objective lens, and the fluorescence emission wavelength to calculate how many pixels the {% include wikipedia title='Point spread function' text='point spread function'%} covers in the image. Then it takes the image in one of the channels, and randomizes it by moving PSF sized chunks of the image to random locations in a new random test image. Then it calculates the {% include wikipedia title='Correlation' text='Pearson\'s correlation coefficient (r)'%} between the randomized image and the original image of the other channel. If the correlation of the randomized image with the real image of the other channel is as good as or better than the correlation between the two real images, then any correlation that you measure is no better then what you would have got by chance for this image. This test is performed many (100) times, and the P-value is output, which is the proportion of random images that had better correlation than the real image. A P-value of 1.00 means that none of the randomised images had better correlation. 0.95 is the normal statistical confidence limit of 95%. Anything lower than that, and the correlation / colocalisation that you measure in the real images is not likely to be better than random chance, and thus is probably not interesting.

In this case the P-value should be 1.00. Since you told it to display the Pearson's correlation (r) values (R values here), they are in another window. You can see they are all close to zero, and in the results you can see that on average the randomized R value is about zero, meaning that the randomized images all had no correlation with the real image. Which is a good thing\!

Other available methods, such as Fay and Van Steensel, do the same thing but randomize the image in less rigorous but very simple ways, which may lead to errors such as over or under estimation of the P-value. However, they are faster than the Costes method.

Again, as for the [Colocalization Threshold](Colocalization_Threshold ) plugin, using an ROI here may well make very good sense, as you are only interested in the correlation between the 2 colour channels in parts of the image where the biology you are interested in is located. Background its typically uninteresting, and you can exclude it from the analysis. Its probably sensible to to use the same ROI as you used in the [Colocalization Threshold](Colocalization_Threshold ) plugin\!

   