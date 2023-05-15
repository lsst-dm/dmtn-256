:tocdepth: 1

.. sectnum::

.. Metadata such as the title, authors, and description are set in metadata.yaml

.. TODO: Delete the note below before merging new content to the main branch.

.. note::

   **This technote is a work-in-progress.**

Abstract
========
This brief study presents the outcomes of a test conducted on HSC (Hyper Suprime-Cam) images of the AP (Alert Production) pipeline, which incorporated a refactored version of ip_diffim DIA (Difference Image Analysis) implementations. The main focus of this technical note is to evaluate the performance of the new possible configurations, also known as "flavors," of the DIA processing and deliver a status report. The results indicate significant improvements in the overall performance of the DIA process, demonstrating better accuracy and efficiency in detecting differences between multiple images. These findings could potentially benefit future astronomical surveys and research that rely on DIA for identifying transient astronomical events.

.. Topics and contents
.. ================

.. .. toctree::s

..     intro.rst
..     dataset_outcome.rst
..     diaSrcAnalysis.rst

#######################
Introduction and objectives
#######################

This technical note objective is to report the run of a recent version of the AP
pipeline using the HSC "DIA Sprint" dataset.

.. The dataset is described in :ref:`referencing <dataset_outcome>`

Itemized list of analysis steps include:

 - Overall description of the image results
 - Filtered sources with flags.
 - Visualization of images and display of visible features.
  - G and R filters with injections and detections
 - Example stamps -- zooniverse tool
 - Estimation of Efficiency vs SNR and magnitude
 - Estiamtion of contamination
 - Estimation of the photometric properties of candidates
 - Estimation of the crossmatched object properties like n_srcs and coord rms
 - Duplicating the analisis items listed above with focus on the flags


.. .. _LSST: http://lsst.org

.. ReStructuredText provides basic *italic*, **bold** and ``monospaced``
.. typesetting.  There is also the concept of **roles** that provide sophisticated
.. typesetting, such as :math:`\mu = -2.5 \log_{10}(\mathrm{DN} / A) + m_0`, and
.. :ref:`referencing <rst-internal-links>`.

.. .. _label-for-subsection-label:



`ip_diffim` Difference Image Analysis Algorithms
================================================

In the LSST Stack the subtractions are handled by `ip_diffim` package.

The `ip_diffim` package is a part of the LSST data processing pipelines, designed for Difference Image Analysis (DIA). It offers several options for processing, such as selecting different kernel sizes, matching PSFs with different techniques, or using different algorithms for object detection. It provides a set of tools for performing photometry on the difference images and generating catalogs of transient astronomical events. The `ip_diffim` package is a critical component of the LSST data processing pipeline and is expected to play a significant role in the discovery of new astronomical phenomena.

The package contains necesary tasks and configurations to detect and measure changes in the brightness of sources in the sky, including new bright sources undetected in archival images. In summary DIA takes two input images, a science image and a reference image (hereafter :math:`N` and :math:`R` respectively), and produces a "difference image" that contains the residual signal not accounted for with observing condition variations.

The image equalization is derived from point-spread function (PSF) matching the reference :math:`R` to the :math:`N` image, by means of a transformation in the form of a kernel convolution :math:`k` (convolution here is the operator :math:`*` ).

:math:`D = N - k * R`

The package is capable of performing subtractions using different DIA "flavors", including conventional A&L image subtraction (:math:`N - kR`), `auto-mode` convolution (where :math:`k` can be applied on either :math:`R` or :math:`N` depending on their relative PSF sizes) and "pre-convolution" where :math:`N` gets convolved beforehand, effectively broadening its PSF size to facilitate the transformation by :math:`k`.



Task preparation and run results
============

We run a few basic steps in order to prepare for the run of the AP pipeline. The details are not extremely important, however we can mention two critical steps.

 - Setting up the pipeline `yaml` file with the task configurations (or the pipetask submit script in case we can set simple options and don't need a separate `yaml`)
 - Setting up the destination AP database location with apdb script `make_apdb.py`


The run results can be found in the destination repository chosen when using the configuration. In our case we ran three different instances or "flavors" of subtractions.




.. Sectioning
.. ==========

.. Sections are formed with underlining the headline text. We use :ref:`a
.. conventional sequence of underline symbols <rst-sectioning>` to indicate
.. different levels of hierarchy.

.. Directives
.. ==========

.. Besides **roles** that are used for inline markup, reStructuredText has the
.. concept of **directives** to markup *blocks* of content. One example is the is
.. the ``code-block`` directive:

.. .. code-block:: python

..    print('hello world!')

###############
Dataset types and results
################

In the data processing we used the HSC dataset from the 2021 diffim sprint and
process both bands through the Alert Production pipeline at the USDF.

As defined in the sprint, the data include:

- g-band visits: [11690, 11692, 11694, 11696, 11698, 11700, 11702, 11704, 11706, 11708, 11710, 11712, 29324, 29326, 29336, 29340, 29350]
- r-band visits: [1202, 1204, 1206, 1208, 1210, 1212, 1214, 1216, 1218, 1220, 23692, 23694, 23704, 23706, 23716, 23718]
- For each visit above, only these detectors (`CCDS`): [49, 50, 57, 58, 65, 66]


Dataset type outputs
====================

The main corpus of the data products is composed of:

 - The difference images, named typically the `differenceExp` dataset
 - Detection and measurement tasks produce we can use the `diaSources` catalogs, containing the transient pixel brightness variation detections
 - Association of the `diaSources` result in `diaObject` catalogs of exactly 1 or more `diaSource` each
 - Additionally, we can query the fake injection catalogs through: `fake`?




Dia Source detections
=====================

We find a total of 54799 sources that are detected in our images.

We include a scatter plot of the sources in each sky grid cell, or patches.

.. figure:: /_static/figures/diasrcs_onskygrid.png
   :name: diasrcs_onskygrid
   :target: ../_images/diasrcs_onskygrid.png
   :alt: diaSrcs On sky grid

   Schematic scatter plot showing the position of DIA source detections diasrcs in a simple grid.
   We observe the spatial distribution of the positions of the detections and we can conclude that they are clearly correlated,and prefer to lie along the horizontal lines.
   We also observe that the objects are custered around what could be bright sources in the field.

.. figure:: /_static/figures/diasrcs_flux_hist.png
    :name: diasrcs_flux_hist
    :target: ../_images/diassrcs_flux_hist.png
    :alt: Flux of detected sources in difference images

    Distribution of the PSF flux measurement of individual detections in the difference images
/home/bruno/Devel/LSST/DMTN/dmtn-256/_static/figures/diasrcs_flux_hist.png

.. A flat version of this note can be found in

.. .. toctree::s
..  :maxdepth: 2
..   flatversion.rst


.. Make in-text citations with: :cite:`bibkey`.
.. Uncomment to use citations
.. .. rubric:: References
..
.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
