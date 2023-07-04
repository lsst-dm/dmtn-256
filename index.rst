:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

Abstract
========
This brief study presents the outcomes of a test conducted on HSC (Hyper Suprime-Cam) images of the AP (Alert Production) pipeline, which incorporated a refactored version of ip_diffim DIA (Difference Image Analysis) implementations.
The main focus of this technical note is to evaluate the performance of the new possible configurations, also known as "flavors," of the DIA processing and deliver a status report.
The results are comparisons based on the number of detections, and the use of fake surce injections on the Visit images as well as Template images.
The results indicate no significant changes in the overall performance of the pipeline, our data is however not demonstrating that better accuracy and efficiency in detecting differences between multiple images is not possible, but instead, that the dataset used might not be the best scenario to uncover the advantages of these different configuration modes to the default standard DIA algorithm.
These findings could potentially benefit future test dataset construction for a correct evaluation of the DIA algorithm for identifying transient astronomical events, in particular, the design of new fake source injected samples.

Introduction and objectives
===========================

About this report
-----------------

This technical note objective is to report the resuls of a run of a recent version of the AP pipeline using the HSC "DIA Sprint" dataset.

The dataset is described in :ref:`2021 diffim sprint <section-dataset>`.

The objectives were made explicit in a series of JIRA tickets, stating multiple tasks and a final ticket asking for this present report, that rounds up the analysis and extracts final conclusions in order to plan future developments on `ip_diffim` and it's corresponant testing.

 - `DM-37957`_ (default mode processing)
 - `DM-37958`_ (default run analysis)
 - `DM-37959`_ (pre-convolution mode processing)
 - `DM-38186`_ (auto-mode processing)
 - `DM-38189`_ (pre-convolution run analisis)
 - `DM-38188`_ (auto-mode run analysis)
 - `DM-37960`_ (write this report)

.. _DM-37957: https://jira.lsstcorp.org/browse/DM-37957
.. _DM-37958: https://jira.lsstcorp.org/browse/DM-37958
.. _DM-37959: https://jira.lsstcorp.org/browse/DM-37959
.. _DM-38186: https://jira.lsstcorp.org/browse/DM-38186
.. _DM-38189: https://jira.lsstcorp.org/browse/DM-38189
.. _DM-38188: https://jira.lsstcorp.org/browse/DM-38188
.. _DM-37960: https://jira.lsstcorp.org/browse/DM-37960


Our main task is clear: to compare the diffrent flavors or DIA implementations, however, the actual metrics to use to obtain a meaningful comparison were not specified.

In the analysis we have included a standard analysis part inspired by notebooks in the repos of ap-pipe notebooks.
We have also included a few analysis bits not present in dm notebooks, and were obtained in a more exploratory work stage.

.. Topics and Contents
.. -------------------

.. Itemized list of analysis steps include:

..  -
..  - Overall description of the image results
..  - Filtered sources with flags.
..  - Visualization of images and display of visible features.
..    - G and R filters with injections and detections
..  - Example stamps -- zooniverse tool
..  - Estimation of Efficiency vs SNR and magnitude
..  - Estimation of contamination
..  - Estimation of the photometric properties of candidates
..  - Estimation of the crossmatched object properties like n_srcs and coord rms
..  - Duplicating the analisis items listed above with focus on the flags

.. .. TODO change this list at the end of esditing


`ip_diffim` Difference Image Analysis Algorithms
================================================

In the LSST Stack the subtractions are handled by `ip_diffim` package.

The `ip_diffim` package is a part of the LSST data processing pipelines, designed for Difference Image Analysis (DIA). It offers several options for processing, such as selecting different kernel sizes, matching PSFs with different techniques, or using different algorithms for object detection. It provides a set of tools for performing photometry on the difference images and generating catalogs of transient astronomical events. The `ip_diffim` package is a critical component of the LSST data processing pipeline and is expected to play a significant role in the discovery of new astronomical phenomena.

The package contains necesary tasks and configurations to detect and measure changes in the brightness of sources in the sky, including new bright sources undetected in archival images. In summary DIA takes two input images, a science image and a reference image (hereafter :math:`N` and :math:`R` respectively), and produces a "difference image" that contains the residual signal not accounted for with observing condition variations.

The image equalization is derived from point-spread function (PSF) matching the reference :math:`R` to the :math:`N` image, by means of a transformation in the form of a kernel convolution :math:`k` (convolution here is the operator :math:`*` ).

:math:`D = N - k * R`

In this setting, the problem is being solved by transforming R so its PSF matches the PSF from N image, and the solution for kernel k is obtained by means of a basis function expansion and a linear chi square minization. typically k can express kernels that involve broadening of the PSF, but any scenario in which the operation turns into a de-convolution the method faces great difficulties.
These scenarios take place when the PSF of N is smaller in size than R PSF. Two mitigation strategies can be used to avoid this:

 #. perform a re-assignemnt of the "target" image: :math:`D = k*N - R`, named the "auto-convolution" mode
 #. apply a previous convolution (i.e. "pre-convolution") to the N image with a given known kernel :math:`v`, so the effective PSF of it is broader than R image PSF and the solution for k it is no longer a de-convolution. As expressed in :math:`v*D = v*N - k*R` the final result of this method is the convolution of the difference image :math:`D` with the desired kernel :math:`v`.

The `ip_diffim` package is capable of performing subtractions using these different DIA "flavors": conventional A&L, `auto-mode` convolution and "pre-convolution".

.. image subtraction (:math:`N - kR`),
.. (where :math:`k` can be applied on either :math:`R` or :math:`N` depending on their relative PSF sizes)
.. where :math:`N` gets convolved beforehand, effectively broadening its PSF size to facilitate the transformation by :math:`k`.


Task preparation and run results
================================

We run a few basic steps in order to prepare for the run of the AP pipeline. The details are not extremely important, however we can mention three critical steps.

 - Setting up the pipeline `yaml` file with the task configurations (or the pipetask submit script in case we can set simple options and don't need a separate `yaml`)
 - Setting up the destination AP database location with apdb script `make_apdb.py`
 - If running with HTCondor resource manager, then allocate the needed resources, this is typically done in USDF (link to the htcondor help)

The run results can be found in the destination repository chosen when using the configuration. In our case we ran three different instances or "flavors" of subtractions, but we also changed some configuration for the pipeline in the case of pre-convoluution, as the source detection step needs a special configuration.



.. _section-dataset:

Dataset and results
-------------------


Input dataset
^^^^^^^^^^^^^

In the data processing we used the HSC dataset from the 2021 diffim sprint and
process both bands through the Alert Production pipeline at the USDF.

As defined in the sprint, the data include:

- g-band visits: [11690, 11692, 11694, 11696, 11698, 11700, 11702, 11704, 11706, 11708, 11710, 11712, 29324, 29326, 29336, 29340, 29350]
- r-band visits: [1202, 1204, 1206, 1208, 1210, 1212, 1214, 1216, 1218, 1220, 23692, 23694, 23704, 23706, 23716, 23718]
- For each visit above, only these detectors (`CCDS`): [49, 50, 57, 58, 65, 66]



Run result details
^^^^^^^^^^^^^^^^^^
* Used weekly `w_2023_07`
* Working directory on USDF. is `/sdf/group/rubin/user/bos/DM-37957`
* Templates are in `u/kherner/DM-33911/templates_bestThirdSeeing`
* Results for **Default mode**

  * Final processing of HSC COSMOS is in `u/bos/DM-37957/w_2023_07_default`
  * ApDB is `/sdf/home/b/bos/u/DM-37957/apdb_bosdm37597.sqlite`

* Results for **Auto convolution mode**

  * Final processing of HSC COSMOS is in `u/bos/DM-38186/w_2023_07_automode`
  * ApDB is `/sdf/home/b/bos/u/DM-38186/apdb_bosdm38186.sqlite`

* Results for **Pre-convolution mode**

  * Final processing of HSC COSMOS is in `u/bos/DM-37959-HTCondor-preconv`
  * ApDB is in the postgres DB, schema is `bos_dm37959_preconv_w2023_07`

.. _section-dataset-types:

Dataset type outputs
--------------------

The main corpus of the data products is composed of:

 - The difference images, named typically the `differenceExp` dataset, accessible through butler directly. In this case we use `fakes_goodSeeingDiff_differenceExp`, as we worked using `ApPipeWithFakes`
 - Detection and measurement tasks produce we can use the `diaSources` catalogs, containing the transient pixel brightness variation detections, accessible using the `apdb` tools.
 - Association of the `diaSources` result in `diaObject` catalogs of exactly 1 or more `diaSource` each
 - Additionally, we can query the fake injection catalogs through: `fakes_goodSeeingDiff_matchDiaSrc`, `fakes_ccdVisitFakeMagnitudes` and `fakes_goodSeeingDiff_assocDiaSrc`


Difference Image inspection
===========================

Inspection of Templates - Science - Difference
----------------------------------------

We inspected the results of image differencing by eye. First is to look at the group of Template-Science and Difference image planes.

.. figure:: /_static/figures/tern_default_g_11690_49/tern_images.png
    :name: fig-tern-default-g-11690-49
    :target: ../_images/tern_images.png
    :alt: Set of images for visit g-11690-49 default run

    Set of images Template-Science-Difference for **Default mode**, filter `g`, visit 11690 detector 49.

We can also check the variance plane of the difference image:

.. figure:: /_static/figures/tern_default_g_11690_49/dif_w_variance.png
    :name: fig-diff-variance-g-11690-49
    :target: ../_images/dif_w_variance.png
    :alt: Variance plane of diff image for visit g-11690-49 default run

    Variance plane of diff image for Default mode, filter `g`, visit 11690 detector 49.

By simple inspection we can see that the difference image features a flat background, no spatial correlation of noise, and almost no present edge effects.

Our inspection of the variance shows that there is some grid pattern that could mimic the template overlaps or the readout noise, that could have larger variance for some pixel columns. Additionally we spot the bright sources, that introduce extra variance, which seems correctly accounted for.

.. figure:: /_static/figures/tern_default_g_11690_49/psftern.png
    :name: fig-psftern-g-11690-49
    :target: ../_images/psftern.png
    :alt: The PSFs of the set of images for visit g-11690-49 default run

    Set of PSF stamps, for the Default mode run, filter `g`, visit 11690 detector 49.

Paying attention to the PSFs of the images we find that the Template PSF appears as higher in Signal-to-Noise Ratio (from here on SNR), as we cannot see the background noise. However it is easy to spot noise in the wings of the PSF, (it might be an artifact of the visualization, but it is unlikely). We can appreciate the Noise-Equivalent-Area circular radius in the panel titles for the template and science PSFs, which shows that for this case the PSF is broader in the science image.

This points that our subtraction with the default mode would be almost identical to the one in the auto-mode and that the pre-convolution procedure should converge to an equivalent kernel transformation, that yields almost the exact subtraction again.


We overlay the detections and fake coordinates. The plot includes the detections both good and bad (flagged detections), as well as the fake locations.

.. figure:: /_static/figures/tern_default_g_11690_49/tern_wfakes.png
    :name: fig-tern_wfakes-g-11690-49
    :target: ../_images/tern_wfakes.png
    :alt: The set of images including fake coordinates and detections for visit g-11690-49 default run

    Set of images Template-Science-Difference, with the detections overlayed both the ones that pass the flag cuts (in yellow) the ones that do not pass (red) and then the fake coordinates (in green crosses), together with their estimated SNR, for the Default mode run, filter `g`, visit 11690 detector 49.

We can find out that the detections group in the central columns, and these are flagged out. Also, fakes can be from templates and from science images, and those will look very differently in the difference images: template fakes are negative "holes" and science fakes are the normal expected transient candidate with positive counts.

The fakes that were found or lost are a bit hard to spot, but in the following figure we can clearly spot the transients in the images and how they were found

.. figure:: /_static/figures/tern_default_g_11690_49/tern_wfakes_found.png
    :name: fig-tern_wfakes-found-g-11690-49
    :target: ../_images/tern_wfakes_found.png
    :alt: The set of images including fake coordinates and detections for visit g-11690-49 default run

    Set of images Template-Science-Difference, with the detected fakes overlayed both the ones that were found and lost with their expected SNR, for the Default mode run, filter `g`, visit 11690 detector 49. In red circles the ones that were not found.


We could attempt to understand if the flavors make a visible difference by searching a pair of science-template images with a PSF relation that makes the auto-convolution mode and pre-convolution mode work in theory better than the default: this is when template PSF is bigger than the science PSF.

For this we pick the visit 11704, detector 58 in g band, being the image with the larger difference between the science and template PSF (in the desired direction). This particular image has a PSF area for the template of 58.1 pixels square and science PSF ENA of 54.6 pixel square. If we assume circular PSF shapes (very good approximation in principle for HSC), we obtain a PSF radius of 3.04 px for Template and 2.95 for the science exposure. Although this scenario is what we need to notice the effects of the different flavors, the difference in PSF sciece might be too subtle to make a difference.

.. figure:: /_static/figures/diff_11704_58_g_def_auto_preconv.png
    :name: fig-diff_11704_58_g_def_auto_preconv-11704-58
    :target: ../_images/diff_11704_58_g_def_auto_preconv.png
    :alt: The difference images for default, auto-mode and pre-convolution for visit g-11704-58 default run

    Image differences with the detected fakes overlayed for the Default mode run (left), auto-convolution mode (center) and pre-convolution mode (right panel), for filter `g`, visit 11704 detector 58.

We appreciate subtle differences, specially around the edges, but overall the algorithms seem to be handling the difficult cases such as saturated stars in a very similar way. Our normalization of the images uses a global zscale normalization, that might get different results due to the edge pixel properties, so the subtle difference in background noise it is not substantial and we think it is equivalent. We realize that maybe this case is an easy case, and the PSFs are not different enough to make the default value extremely suboptimal (like a straight deconvolution).



Dia Source detections
=====================

We can compare the sources that are detected in our images, we analyze the single detections or diaSources and the associations or diaObjects.

+-----------+----------+----------+
|           | N diaSrc | N diaObj |
+===========+==========+==========+
|   Default |    54799 |    33974 |
+-----------+----------+----------+
|  Pre-Conv |    60243 |    41343 |
+-----------+----------+----------+
| Auto Mode |    50049 |    30926 |
+-----------+----------+----------+

In the following figures we include the number of diaSource per CCD for each mode.

.. figure:: /_static/figures/number_diasrcs_default.png
    :name: fig-number_diasrcs_default
    :target: ../_images/number_diasrcs_default.png
    :alt: N diaSources per visit default mode

.. figure:: /_static/figures/number_diasrcs_convolutionauto.png
    :name: fig-number_diasrcs_convolutionauto
    :target: ../_images/number_diasrcs_convolutionauto.png
    :alt: N diaSources per visit auto-convolution mode

.. figure:: /_static/figures/number_diasrcs_preconvolution.png
    :name: fig-number_diasrcs_preconvolution
    :target: ../_images/number_diasrcs_preconvolution.png
    :alt: N diaSources per visit pre-convolution mode

The different distributions of diaSource detections per CCD for each mode, reveal that the number of artifacts seem to be lower for the default and auto-convolution modes, with respect to the number of sources in pre-convolution. These sources are including all the detections, and have no flag filtering.

If we apply some flag filtering we can clean this sample. This in principle is pruning out bad detections, like subtraction artifacts on saturated stars, edges or bad pixels in the CCD detector.
The set of flags is the commonly used throughout the analysis-ap notebooks:

.. code-block:: python

    badFlagList = [
        'base_PixelFlags_flag_bad',
        'base_PixelFlags_flag_suspect',
        'base_PixelFlags_flag_saturatedCenter',
        'base_PixelFlags_flag_interpolated',
        'base_PixelFlags_flag_interpolatedCenter',
        'base_PixelFlags_flag_edge'
        ]

The proportion of flagged sources and objects can be seen in the following figure.

.. figure:: /_static/figures/flags_combined.png
    :name: fig-distribution-flags
    :target: ../_images/flags_combined.png
    :alt: Flag bit distribution for each mode.

    The number of flagged diaSources per flag, for each respective mode run. The red bars correspond to the `badFlagList` mentioned earlier as the most conventional flag cuts.


After applying this cuts the table looks like this:

+-----------+----------+----------+-------------+----------------+
|           | N diaSrc | N diaObj | Good diaSrc | Good diaObject |
+===========+==========+==========+=============+================+
|   Default |    54799 |    33974 |       13244 |           6166 |
+-----------+----------+----------+-------------+----------------+
|  Pre-Conv |    60243 |    41343 |       10909 |           6381 |
+-----------+----------+----------+-------------+----------------+
| Auto Mode |    50049 |    30926 |       13727 |           6698 |
+-----------+----------+----------+-------------+----------------+

.. ---------------+--------------+-----------------------------+
..  N fakes Match | N Fakes Diff | N fakes Matched Not Flagged |
.. ===============+==============+=============================+
..         4627.0 |      50172.0 |                      2391.0 |
.. ---------------+--------------+-----------------------------+
..         4594.0 |      55649.0 |                      1732.0 |
.. ---------------+--------------+-----------------------------+
..         4321.0 |      45728.0 |                      2348.0 |
.. ---------------+--------------+-----------------------------+

The distribution of number of "good" diaSources per CCD now changes completely with respect to all the detections.
The typical number of detections drops down to around 100 or 150 per CCD image, and the distribution is less disperse, showing the cumulative a soft profile.

.. figure:: /_static/figures/number_good_diasrcs_default.png
    :name: fig-number_good_diasrcs_default
    :target: ../_images/number_good_diasrcs_default.png
    :alt: N diaSources per visit default mode

.. figure:: /_static/figures/number_good_diasrcs_convolutionauto.png
    :name: fig-number_good_diasrcs_convolutionauto
    :target: ../_images/number_good_diasrcs_convolutionauto.png
    :alt: N diaSources per visit auto-convolution mode

.. figure:: /_static/figures/number_good_diasrcs_preconvolution.png
    :name: fig-number_good_diasrcs_preconvolution
    :target: ../_images/number_diasrcs_preconvolution.png
    :alt: N diaSources per visit pre-convolution mode



If we analyze the number of `diaSources` per `diaObject` we obtain the following distributions:

.. figure:: /_static/figures/n_objects_goodObjects.png
    :name: fig-distribution-n-diaSources-per-object
    :target: ../_images/n_objects_goodObjects.png
    :alt: N diaSources per diaObject (flag filtered)

    The distribution of number of associated diaSources in the diaObjects for each respective mode run. The orange is a subset of the full diaObject distribution (in blue), after applying conventional flag cuts.

The conclusion that we can get from this is that most of the filtering is done for diaObjects that have less than 5 diaSources associated. This indicates a transient candidate that is not likely to be astrophysical in origin, although we are dealing with fakes in this situation it is acceptable.
Another conclusion from this plot is that the number of diaObjects in Pre-convolution is higher, but after filtering it ends up being lower (by a ~300 diaObjects margin) than the other flavors. Our plot also shows that a significant portion of these could be in the bin of 20 or more diaSources, which is interesting.

In the following figure we have a scatter plot of these diaObjects on sky coordinates.

.. figure:: /_static/figures/good_diaObj_zoomed_sky.png
   :name: good_diaObj_zoomed_sky
   :target: ../_images/good_diaObj_zoomed_sky.png
   :alt: Scatter of diaObjects in the sky

   Scatter plot showing the position of DIA source associations for each filter and each DIA flavor or mode. Size of the points is proportional to the number of associated diaSources.

We observe the spatial distribution of the diaObjects and their number of diaSources also displayed as the size of the scatter points. We can observe that the objects are clustered around what could be bright sources in the field. In contrast to the default mode we see that there are less points in the pre-convolution mode, although the difference is subtle.

We can understand that the associated diaSources in this plot should have already a significant cut, and they are mostly equivalent.




Fake source injection analysis
==============================

Number of matches
-----------------

We can expand the table that we built before to include the number of fake source matches, also including an estimation of the "contamination" -- this is the diaSources that have no match with the fakes, and also applying the conventional flag cuts.

.. _table-nfakematches:

.. table:: Table with number of fake matches per DIA flavor.

    +-----------+-------------+----------------+---------------+---------------+---------------------+
    |           | Good diaSrc | Good diaObject | N Fakes Match | contamination |  Matches w/flag cuts|
    +===========+=============+================+===============+===============+=====================+
    |   Default |       13244 |           6166 |        4627.0 |       50172.0 |              2391.0 |
    +-----------+-------------+----------------+---------------+---------------+---------------------+
    |  Pre-Conv |       10909 |           6381 |        4594.0 |       55649.0 |              1732.0 |
    +-----------+-------------+----------------+---------------+---------------+---------------------+
    | Auto Mode |       13727 |           6698 |        4321.0 |       45728.0 |              2348.0 |
    +-----------+-------------+----------------+---------------+---------------+---------------------+

We can see that the amount of matches decreases to almost a 50% when applying the flag cuts. Our conclusion is that maybe our set of flags is too agressive, and it is not really discriminating the real transients from the potential artifacts, but instead vettoing on information that might or might not be relevant.


CCD Illumination: X-Y distribution of detections
================================================

Looking at the distribution of the pixel coordinates of the detections we can try to obtain information about the algorithm performance around central and edge areas of the CCD. This contains valuable information about the capability of the algorithm to work in the full field of view and also can be of use when understanding the uniformity of the detections in the pixel field.

.. figure:: /_static/figures/chipillum_x_y_histogram_diffim_flavors.png
    :name: chipillum_x_y_histogram_diffim_flavors
    :target: ../_images/chipillum_x_y_histogram_diffim_flavors.png
    :alt: X-Y pixel coordinate distributions for diaSources

    Distribution of the X and Y coordinates of the diaSources.

Two important caveats of this plot are: the dimensions of the chip are 4096 by 2048, so X axis is smaller, and finally some detection centroids can end up in negative coordinates or, more generally coordinates that exceed the real CCD domain.

As a general note we understand that the number of transient candidate detections should not depend on the location on the image CCD domain up to a certain extent. This means, there will be some loss of sensitivy on algorithms such as pre-convolution, as the edges will have incomplete information due to kernel padding. In the following figure we can see a zoom in into the interesting edge areas, for x and y axes.

.. figure:: /_static/figures/chipillum_x_y_histogram_zoom_diffim_flavors.png
    :name: chipillum_x_y_histogram_zoom_diffim_flavors
    :target: ../_images/chipillum_x_y_histogram_zoom_diffim_flavors.png
    :alt: X-Y pixel coordinate distributions for diaSources

    Distribution of the X and Y coordinates of the diaSources, zoomed in the edges and central x-axis locations.

Clearly the distribution has several spikes at different locations, central columns along the x-axis shows an excess of sources detected as well as the edges of all the algorithms feature an incredible amount of excess of transient detections (the plot y-axis is on logarithmic scale), even on some ocasions an order of magnitude greater than the overall value of :math:`10^2`.


In the following figure we display the scatter of transient candidates.

.. figure:: /_static/figures/chipillum_scatter_diffim_flavors.png
    :name: chipillum_scatter_diffim_flavors
    :target: ../_images/chipillum_scatter_diffim_flavors.png
    :alt: Location of the detected sources in the CCD chip.

    The location of the detected transient candidates in the CCD chip. We call this the "illumination" of the CCD. It is clear again, an excess of detections in the central pixel columns and the chip edges.

The figure shows that there are columns with detection excess that can be removed, by using different flags or using information in the image mask plane. In the following figure we can see the scatter of the sources that pass the flag cuts.

.. figure:: /_static/figures/chipillum_scatter_goodsrc_diffim_flavors.png
    :name: chipillum_scatter_goodsrc_diffim_flavors
    :target: ../_images/chipillum_scatter_goodsrc_diffim_flavors.png
    :alt: Location of the detected sources in the CCD chip after common flag cuts.

    Position of the detected transient candidates that pass flag cuts in the CCD chip.

The flags cut the excess of detections up to certain level, and also leave areas in the CCD where now we have an artificial under-density of detections.

.. figure:: /_static/figures/chipillum_x_y_histogram_goodsrcs_diffim_flavors.png
    :name: chipillum_x_y_histogram_goodsrcs_diffim_flavors
    :target: ../_images/chipillum_x_y_histogram_goodsrcs_diffim_flavors.png
    :alt: X-Y pixel coordinate distributions for diaSources that pass flag cuts

    Distribution of the X and Y coordinates of the diaSources that pass the conventional flag cuts.

In the distribution of coordinates we can see that the edge excess and central excess get flattened and apparently the artifact sources are cut away.

We can study again the location distribution, but as a function to the distance to the center of the image, however since the image is rectangular this will not yield a flat distribution.
Instead we can weight each source detection by the area of the smallest central rectangle that contains the detection. This means, taking each pair of transient coordinate :math:`(x,y)` and transforming it to obtain the side of this rectangle: :math:`l = max(x', y')` with :math:`(x', y') = (x-x_c, y-y_c)`. The value :math:`\gamma` that we take finally into account is the fractional area with respect to the full CCD area: :math:`\gamma = l^2/(4096 \times 2048)`
This quantity will be distributed as a random uniform distribution for a true random position of our

.. figure:: /_static/figures/chipillum_areafraction_combined_diffim_flavors.png
    :name: chipillum_areafraction_combined_diffim_flavors
    :target: ../_images/chipillum_areafraction_combined_diffim_flavors.png
    :alt: Fractional area factor of illumination.

    Distribution of the fractional area :math:`\gamma` for different DIA flavors and for all the diaSources, the sources after flag cuts and the fakes that were detected in the substraction.

The distribution of this fractional value shows that instead of a flat profile, we get higher densities of detections closer to the edges of the CCD. The distribution changes if we look at the sources that pass flag cuts, however it doesn't become completely flat.

.. figure:: /_static/figures/chipillum_areafraction_combined_logscale_diffim_flavors.png
    :name: chipillum_areafraction_combined_logscale_diffim_flavors
    :target: ../_images/chipillum_areafraction_combined_logscale_diffim_flavors.png
    :alt: Fractional area factor of illumination.

    Distribution (with the y-axis log-scale) of the fractional area :math:`\gamma` for different DIA flavors and for all the diaSources, the sources after flag cuts and the fakes that were detected in the substraction.

The distribution for the fakes that were found instead show that the edges are less frequent and this could mean that the flag cuts are discarding events that are effectively fake injections (i.e. true transient sources).
This effect seems to be present to the same level on all the DIA flavors. This could point to some feature of the dataset from HSC or some other effect by the pipeline.


Efficiency of transient detection
=================================

We attempted to estimate the efficiency of detecting transients. This can be done as a function of the transient magnitude, which would depend on the filter and exposure, as well as the instrument noise properties, or instead we can do it as a function of the Signal-to-Noise ratio (SNR).

The main difficulty on the latter approach is to obtain an effective SNR even for fake injections that were not detected by the DIA pipeline. In that case, we must resource to a model of the noise properties of the image.


Signal-to-Noise estimation
--------------------------

The pipeline photometry module estimates a single SNR, however it measures fluxes and their uncertainties using several methods; here we will use the conventional PSF photometry one.

For a given point source at location :math:`(x_c, y_c)` its PSF flux estimation is: :math:`\rm{flux}(x_c, y_c) = \sum_{ij} \rm{Im}_{ij} \rm{PSF}(x_c, y_c)_{ij} / \sum \rm{PSF}^2`.

The PSF flux variance is :math:`\sigma^2_f` due to the sky variance will be the weighted sum of the variance of the pixels, following a similar formula as above, but instead of :math:`\rm{Im}_{ij}` we use the variance plane :math:`\rm{Var}_{ij}`. In the following :ref:`figure <fig-snr-pipe-vs-snrvariance>` we can see that the SNR from the pipeline and the one estimated by the described method are not equal.

.. figure:: /_static/figures/snr_pipe_vs_snrvariance.png
    :name: fig-snr-pipe-vs-snrvariance
    :target: ../_images/snr_pipe_vs_snrvariance.png
    :alt: SNR from the pipeline vs PSF weighted variance plane

    The estimated SNR from PSF weighted sum of the variance plane as a function of the SNR that the pipeline assigns. This plot is only possible for the found fakes, as the lost fake sample has no SNR estimation from the pipeline.

Our estimation of SNR is not in agreement with the reported SNR from the pipeline and instead we can use a different proxy SNR (heare after Model SNR), by modelling the uncertainties as done previously in other works such as `Sanchez et al 2022`_.

.. _Sanchez et al 2022: https://ui.adsabs.harvard.edu/link_gateway/2022ApJ...934...96S/doi:10.3847/1538-4357/ac7a37

To predict the SNR for a given point source of magnitude :math:`m` we first convert the magnitude value to flux in nanoJansky units, and make use of the calibration to obtain the pixel count values directly.
Next we use the image variance plane to obtain a median variance estimate around the object :math:`\sigma_{sky}`, and multiply this with the noise-equivalent area given by the factor :math:`[\sum \rm{PSF}^2]^{-1}`. Additional terms in the variance that we include are the zeropoint calibration error and the flux count variance (following a Poisson distribution law).

The obtained calculated SNR values are close to the pipeline SNR, but not exactly equal. In the folowing :ref:`figure <fig-snr-model-vs-snrflavors>` we find that the distributions do not agree completely, but do follow a similar shape profile.

.. figure:: /_static/figures/snr_model_vs_snrflavors.png
    :name: fig-snr-model-vs-snrflavors
    :target: ../_images/snr_model_vs_snrflavors.png

    The distributions of the several SNR estimated and reported by the pipeline: `SNR`, `psFluxSNR`, and `totFluxSNR` and the ones estimated independently using the variance `est. SNR` and the full model `Model SNR`.

When compared individually, as in the following :ref:`figures <fig-fake-snr_over-pred-snr-vs-fakemag-templt-and-science>`, we see that the estimated SNR using the variance plane is in better agreement to the pipeline reported SNR, although it is having problems at low-SNR for template fake sources.


.. figure:: /_static/figures/fake_snr_over_pred_snr_vs_fakemag_templt_and_science.png
    :name: fig-fake-snr_over-pred-snr-vs-fakemag-templt-and-science
    :target: ../_images/fake_snr_over_pred_snr_vs_fakemag_templt_and_science.png

    The ratio of the `Model SNR` to the pipeline reported `SNR`, for each subtraction method and as a function of the pipeline reported `SNR` and the Fake Magnitude. We also split between science and template fakes.

.. figure:: /_static/figures/fake_snr_over_est_snr_vs_fakemag_templt_and_science.png
    :name: fig-fake-snr_over-est-snr-vs-fakemag-templt-and-science
    :target: ../_images/fake_snr_over_est_snr_vs_fakemag_templt_and_science.png

    The ratio of the variance `estimated SNR`` to the pipeline reported `SNR`, for each subtraction method and as a function of the pipeline reported `SNR` and the Fake Magnitude. We also split between science and template fakes.



Efficiency estimation
---------------------

We calculate the efficiency of detecting the fake injected sources as a function of the Model and estimated SNR values.

Our fitting procedure is using the binomial distribution model for the detection of a fake transient, this results in a sigmoid function model :math:`\rm{eff}(x) = [1 + e^{-a (x - b)}]`, and our error estimation is according to the variance of this distribution.

.. figure:: /_static/figures/eff_estimation_diffim_flavors.png
    :name: fig-eff-estimation-diffim-flavors
    :target: ../_images/eff_estimation_diffim_flavors.png

    The efficiency of the pipelines for finding fake transient sources, for the different DIA flavors. The SNR used is the `Model SNR`.

.. figure:: /_static/figures/eff_estimation_diffim_flavors_estsnr.png
    :name: fig-eff-estimation-diffim-flavors-estsnr
    :target: ../_images/eff_estimation_diffim_flavors_estsnr.png

    The efficiency of the pipelines for finding fake transient sources, for the different DIA flavors. The SNR used is the `estimated SNR`.

We observe that for each of the SNR estimations the efficiency measurement lacks signal in the bins with low-SNR (from 0 to 5), where apparently we have not enough sources to obtain a real efficiency profile function.

We define the SNR at 50% efficiency :math:`SNR_{1/2}`, and from our modelling we obtain that it is oscillating just below :math:`SNR_{1/2} = 5`. We might be observing in this case the SNR detection threshold that is configured by default in the pipelines.

Our results point also that the efficiency is good at :math:`SNR>5`, but it is not reaching a stable value, and it oscilates dropping up to an 80%. This should instead be approaching to a 100% upper bound.

Future work on how to increase the statistical signficance of the efficiency measurements could prove very beneficial, specially by adding low-SNR fake sources in our sample, and this direction is already being explored.

Assesment of the photometric performance
========================================

We try to understand the resuls of photometric measurements using the fake injections.
For this we make use of the injection magnitude and the recovered or measured magnitude value their difference :math:`\Delta m = \rm{mag} - \rm{mag_{true}}` and the flux pull values :math:`(\rm{flux} - \rm{flux_{true}})/\sigma_{\rm{flux}}`.

.. figure:: /_static/figures/mag_offset_vs_SNR_diffim_flavors.png
    :name: fig-mag-offset-vs-SNR-diffim-flavors
    :target: ../_images/mag_offset_vs_SNR_diffim_flavors.png

    The offset magnitude :math:`\Delta m` for the found fakes as a function of SNR.

From Figure 23 we find that the scatter on the offsets is higher than expected, with offsets :math:`\Delta m \simeq \pm 1`. 
This is indistiguishable for DIA flavors, as they yield almost identical scatter results. 

For the analysis of the pull distribution we define an outlier set, that have an offset :math:`|\Delta m | > 0.1` and have a reported PSF flux SNR :math:`>0.5`.

.. figure:: /_static/figures/pull_distribution_w_outliers_diffim_flavors.png
    :name: fig-pull-distribution-w-outliers-diffim-flavors
    :target: ../_images/pull_distribution_w_outliers_diffim_flavors.png

    The pull distribution for the found fakes as a function of SNR. The plot is split in outliers and "inliers".

When looking at the pull distributions we see in the outliers a bump with large negative pull values. When inspecting this we find that this sample is the "Template Fake" set, that is to say, transients injected in the template images. In the next figure we can confirm this.

.. figure:: /_static/figures/pull_distribution_w_outliers_diffim_flavors_onlytemplatefake.png
    :name: fig-pull-distribution-w-outliers-diffim-flavors-onlytemplatefake
    :target: ../_images/pull_distribution_w_outliers_diffim_flavors_onlytemplatefake.png

    The pull distribution for the found fakes as a function of SNR only for the Template Fakes.


Template Seeing Analysis 
========================

We analyze the PSF sizes for the images, including their templates. This is relevant for our analysis, given that the different flavors of DIA will behave differently mainly as a function of these two parameters. 
For the PSF size we use again the PSF noise equivalent area factor and transform it to an equivalent radius, assuming circular shape: :math:`r_{\rm{eff}} = \sqrt{\rm{PSF_{NEA}}/(2\pi))}`.

In the following figure we show the distributions of the PSF :math:`r_{\rm{eff}}` sizes for Templates and Science images, for both used filters G and R. 

.. figure:: /_static/figures/psf_nea_circradius_filters_template_vs_science.png
    :name: fig-psf_nea_circradius_filters_template_vs_science
    :target: ../_images/psf_nea_circradius_filters_template_vs_science.png

    The PSF :math:`r_{\rm{eff}}` sizes for Templates and Science images, for both used filters G and R

The difference in radius between a Science image an its template would give us information on whether the DIA method needs to fit a kernel for deconvolution or not.

.. figure:: /_static/figures/psf_nea_circradius_filters_delta_template_science.png
    :name: fig-psf_nea_circradius_filters_delta_template_science
    :target: ../_images/psf_nea_circradius_filters_delta_template_science.png

    The delta PSF :math:`r_{\rm{eff}}` sizes for Templates and Science images, for both used filters G and R.

Our conclusion for this is that the used dataset is not exposing the DIA methods to the scenario where deconvolution of the science image is needed. In our final reccommendations we suggest that for testing the DIA pipeline is key to use datasets that have the (un)-desired scenarios that we expect the pipeline handle successfully during operations.

In our following figure we take a look at the number of detections per images.

.. figure:: /_static/figures/n_sources_vs_psf_delta_template_science.png
    :name: fig-n_sources_vs_psf_delta_template_science
    :target: ../_images/n_sources_vs_psf_delta_template_science.png

    Number of diaSource detections for each DIA flavor, as a function of the difference of PSF size between the Template and Science images. We overlay both filters used (G and R) and show the Number of sources before and after flag cuts. In the right panels we include the same plot but using logarithmic scale on the y axis.

We do not observe a dependency on the number of detections with the difference in PSF size. The number of detections is actually very similar for all the DIA flavors. 


Conclusions
===========

We run the ApPipeWithFakes pipeline on HSC G and R data, using three different setups for the DIA "flavor": the default Alard & Lupton, the convolution-auto mode, and the pre-convolution mode.
We can extract a few conclusions on this comparative analysis.

 * The number of fake injections is too low to get enough statistical signal for efficiency measurement.
 * The SNR of the fake injections is too high to probe the low-SNR regime of the image dynamic range.
 * The flag cuts conventionally applied are too aggressive and would potentially discard good transient sources. 
 * The photometric accuracy is around 0.2 magnitudes, and seems to be not up to the requirement specification.
 * Our dataset for this test does not contain some specific scenarios in which the various DIA flavors should out-perform the default Alard & Lupton technique. Probably incorporating some cases with bad seeing on the templates is of value for testing purposes.
 * The number of detections around the edges of CCDs are high, although correct flagging could potentially fix this issue.
 * Understanding correctly how to model the SNR for point sources of a given magnitude could yield value in the assessment of the limit of detections. Particularly identifying the correct variance model for source pixels, background and calibration would be beneficial. Additionally, this could be of use when optimizing the DIA source detection threshold. 

.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
..
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
