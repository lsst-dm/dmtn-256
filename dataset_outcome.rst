###############
Dataset types and results
################

In the data processing we used the HSC dataset from the 2021 diffim sprint and
process both bands through the Alert Production pipeline at the USDF.

As defined in the sprint, the data include:
g-band visits
    11690, 11692, 11694, 11696, 11698, 11700, 11702, 11704, 11706, 11708, 11710, 11712, 29324, 29326, 29336, 29340, 29350
r-band visits
    1202, 1204, 1206, 1208, 1210, 1212, 1214, 1216, 1218, 1220, 23692, 23694, 23704, 23706, 23716, 23718

For each visit above, only these detectors (ccds)
    49, 50, 57, 58, 65, 66


Dataset type outputs
====

The main corpus of the data products is built with

 * `diaSources`
 * `diaObjects`
 * `diffExp`
 * `fakes?`

