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
  - G and R filters separated. Injections and detections
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

The `ip_diffim` package is a part of the LSST data processing pipelines, designed for Difference Image Analysis (DIA).

The package contains necesary tasks and configurations to detect and measure changes in the brightness of sources in the sky, including new bright sources undetected in archival images. In summary DIA takes two input images, a science image and a reference image (hereafter $N$ and $R$ respectively), and produces a "difference image" that contains the residual signal not accounted for with observing condition variations. The image equalization is derived from point-spread function (PSF) matching the reference $R$ to the $N$ image, by means of a transformation in the form of a kernel convolution $k$. The package is capable of performing subtractions using different DIA "flavors", including conventional image subtraction (A&L $N - kR$), `auto-mode` convolution (where $k$ can be applied on either $R$ or $N$ depending on their relative PSF sizes) and "pre-convolution" where $N$ gets convolved beforehand, effectively broadening its PSF size to facilitate the transformation by $k$.

The package offers several options for processing, such as selecting different kernel sizes, matching PSFs with different techniques, or using different algorithms for object detection. It provides a set of tools for performing photometry on the difference images and generating catalogs of transient astronomical events. The `ip_diffim` package is a critical component of the LSST data processing pipeline and is expected to play a significant role in the discovery of new astronomical phenomena.


Task preparation and run results
============

We run a few basic steps in order to prepare for the run of the AP pipeline. The details are not extremely important, however we can mention two critical steps.

 - Setting up the pipeline yaml with the task configurations (or the pipetask submit script in case we can set simple options and don't need a full fleshed yaml)
 - Setting up the destination AP database location with apdb script `make_apdb.py`


The run results can be found in the destination repository chosen when using the configuration. In our case we ran three different instances or "flavors" of subtractions.

[]


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