---
title: "LandR _Biomass_validationKNN_ Manual"
subtitle: "v.0.0.2.9001"
author: "Ceres Barros"
date: "Last updated: 2022-02-24"
output:
  bookdown::html_document2:
    toc: true
    toc_float: true
    theme: sandstone
    number_sections: false
    df_print: paged
    keep_md: yes
editor_options:
  chunk_output_type: console
  markdown: 
    wrap: 80
bibliography: citations/references_Biomass_validationKNN.bib
citation-style: citations/ecology-letters.csl
link-citations: true
always_allow_html: true
---

# LandR *Biomass_validationKNN* Module



[![made-with-Markdown](https://img.shields.io/badge/Made%20with-Markdown-1f425f.svg)](http://commonmark.org)
[![Generic
badge](https://img.shields.io/badge/Get%20help-Report%20issues-%3CCOLOR%3E.svg)](https://github.com/PredictiveEcology/Biomass_validationKNN/issues)

**This documentation is work in progress. Potential discrepancies and omissions
may exist for the time being. If you find any, do contact us using the link
above\^\^**

## Module Overview

### Module summary

An approach to validating outputs from LandR Biomass - notably the
*Biomass_core* vegetation simulation module - using publicly available data for
Canadian forests. This module produces both visual and statistical validation of
*Biomass_core* outputs that are related to species abundance and
presence/absence in the landscape. To do so, it downloads and prepares all
necessary data (observed and simulated), calculates validation statistics and
produces/saves validation plots.

### Module inputs and parameters at a glance

*Biomass_validationKNN* requires access to outputs of simulations from
*Biomass_core*, and internet access to retrieve the observed kNN datasets used
for validation. We advise future users to run *Biomass_validationKNN* with
defaults and inspect what the objects are like before supplying their own data,
or alternative data URLs. We expect the number of validation modules to increase
as other validation approaches are developed based on project needs.

The module is able to compile all simulation output data provided that the user
supplies the object names and their file paths via the `simulationOutputs` input
object. Alternatively, the user may pass the pre-compiled outputs (namely the
`cohortData` and `pixelGroupMap` objects) via the `allCohortData` and
`pixelGroupMapStk` input objects. See [Input objects] for more detail.

Key parameters are those defining simulation years and replicates,
(`validationYears`, `validationReps`) and plot control (`.plots`). Here's the
full list of parameters:

Table \@ref(tab:moduleInputsBiomassValid) shows a full list of input objects that
*Biomass_validationKNN* expects.

<table class="table" style="margin-left: auto; margin-right: auto;">
<caption>(\#tab:moduleInputsBiomassValid)List of _Biomass_validationKNN_  input objects and their description.</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> objectName </th>
   <th style="text-align:left;"> desc </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> allCohortData </td>
   <td style="text-align:left;"> All `cohortData` tables saved during the simulation, particularly for the validation years. If not supplied, the module will attempt to retrieve them using the 'simulationOutputs' table </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biomassMap </td>
   <td style="text-align:left;"> total biomass raster layer in study area (in g/m^2), filtered for pixels covered by `cohortData`. Only used to calculate total no. of pixels being simulated If not supplied, will default to `rawBiomassMapStart` </td>
  </tr>
  <tr>
   <td style="text-align:left;"> firePerimeters </td>
   <td style="text-align:left;"> A map of fire perimeters in the study area that can be used to exclude pixels that have been burnt during the validation period. If burnt pixels are not to be excluded Provide an empty `sf` object with the same properties as the default. Defaults to the latest Canadian Wildland Fire Information System National Burned Area Composite, subset to fires occuring up to last validation year (inclusively). Source URL determined by `fireURL` </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fireURL </td>
   <td style="text-align:left;"> A URL to a fire database, such as the Canadian National Fire Database, that is a zipped shapefile with fire polygons, an attribute (i.e., a column) named 'Year'. If supplied (omitted with NULL or NA), this will be used to 'update' age pixels on `standAgeMap` with 'time since fire' as derived from this fire polygons map </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pixelGroupMapStk </td>
   <td style="text-align:left;"> A stack of `pixelGroupMap`s saved during the simulation, particularly for the validation years. If not supplied, the module will attempt to make it using the 'simulationOutputs' table </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rawBiomassMapStart </td>
   <td style="text-align:left;"> observed total biomass raster layer in study area at the first year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived total aboveground biomass map from 2001 (in ton/ha). See https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 for metadata. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rawBiomassMapEnd </td>
   <td style="text-align:left;"> observed total biomass raster layer in study area at the last year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived total aboveground biomass map from 2011 (in ton/ha) See https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rasterToMatch </td>
   <td style="text-align:left;"> A raster of the `studyArea` in the same resolution and projection as `rawBiomassMapStart`. This is the scale used for all outputs for use in the simulation. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rstLCChange </td>
   <td style="text-align:left;"> A mask-type map of land cover changes in the study area that can be used to exclude pixels that have been disturbed during the validation period. If disturbed pixels are not to be excluded Provide an empty sf object with the same properties as the default. Defaults to Canada's forest change map between 1985-2011 (CFS), filtered for years 2001-2011 (inclusively) and all disturbances collapsed (map only has values of 1 and NA). See `P(sim)$LCChangeYr` parameter to change the period of disturbances, and https://opendata.nfis.org/mapserver/nfis-change_eng.html for more information. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rstLCChangeYr </td>
   <td style="text-align:left;"> An OPTIONAL map of land cover change years in the study area used to exclude pixels that have been disturbed during the validation period. It defaults to Canada's forest change year national map between 1985-2011 (CFS). If `P(sim)$LCChangeYr` is not `NULL`, this layer is used to filted disturbed pixels that fall within the years specified by `P(sim)$LCChangeYr`. If `P(sim)$LCChangeYr` is `NULL` this layer is not used. See https://opendata.nfis.org/mapserver/nfis-change_eng.html for more information. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> simulationOutputs </td>
   <td style="text-align:left;"> An OPTIONAL table listing simulation outputs (as passed to `spades()`, or `experiment) that will be used to make `allCohortData`, `pixelGroupMapStk`, if these are not provided. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> speciesLayersStart </td>
   <td style="text-align:left;"> observed cover percentage raster layers by species in Canada species map, at the first year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived species cover maps from 2001, using a cover threshold of 10% - see https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 for metadata </td>
  </tr>
  <tr>
   <td style="text-align:left;"> speciesLayersEnd </td>
   <td style="text-align:left;"> observed percent cover raster layers by species in Canada used for validation at the last year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived species cover maps from 2011 - see https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 for metadata </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sppColorVect </td>
   <td style="text-align:left;"> named character vector of hex colour codes corresponding to each species </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sppEquiv </td>
   <td style="text-align:left;"> table of species equivalencies. See `LandR::sppEquivalencies_CA`. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> standAgeMapStart </td>
   <td style="text-align:left;"> observed stand age map in study area, at the first year of the validation period Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived biomass map from 2001 - see https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 for metadata </td>
  </tr>
  <tr>
   <td style="text-align:left;"> standAgeMapEnd </td>
   <td style="text-align:left;"> observed stand age raster layer in study area, at the last year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived stand age map from 2011. See https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studyArea </td>
   <td style="text-align:left;"> Polygon to use as the study area. Must be provided by the user </td>
  </tr>
</tbody>
</table>

Below is the full list of input objects that *Biomass_validationKNN* expects
(Table \@ref(tab:moduleParamsBiomassValid)). Of these, the only input that **must** be
provided (i.e. *Biomass_validationKNN* does not have a default for) is
`studyArea`. Objects suffixed with `*Start` correspond to the same objects in
the simulation without this suffix (e.g. `rawBiomassMapStart` is `rawBiomassMap`
in the simulation). Order objects like `studyArea` and `rasterToMatch` have the
same names in the simulation.

We advise users to **supply the *exact same objects*** as those used in the
simulations wherever possible - see example below.

<table class="table" style="margin-left: auto; margin-right: auto;">
<caption>(\#tab:moduleParamsBiomassValid)List of _Biomass_validationKNN_ parameters and their description.</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> paramName </th>
   <th style="text-align:left;"> paramDesc </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> coverThresh </td>
   <td style="text-align:left;"> The minimum % cover a species needs to have (per pixel) in the study area to be considered present. Should be the same as the one used to obtain the species cover layers for simulation set up. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deciduousCoverDiscount </td>
   <td style="text-align:left;"> This was estimated with data from NWT on March 18, 2020 and may or may not be universal. Should be the same as the one used when preparing `cohortData` in the simulation set up. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> LCChangeYr </td>
   <td style="text-align:left;"> OPTIONAL. An integer or vector of integers of the validation period years, defining which years of land-cover changes (i.e. disturbances) should be excluded. `NULL` by default, which presumes no subsetting based on years is done internally (either the user supplies a pre-filtered `rstLCChange`, or no filtering is desired). If not `NULL` `rstLCChangeYr` is used to filter disturbed pixels within the specified years. See https://opendata.nfis.org/mapserver/nfis-change_eng.html for more information. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> minCoverThreshold </td>
   <td style="text-align:left;"> Cover that is equal to or below this number will be omitted from the dataset Should be the same as the one used when preparing `cohortData` in the simulation set up. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> obsDeltaAgeB </td>
   <td style="text-align:left;"> When TRUE, the observed changes in biomass and age (deltaB, deltaAge) between the two validation years will be plotted as maps and scatterplots </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pixelGroupBiomassClass </td>
   <td style="text-align:left;"> When assigning `pixelGroup` membership, this defines the resolution of biomass that will be considered 'the same pixelGroup', e.g., if it is 100, then 5160 and 5240 will be the same Should be the same as the one used when preparing `cohortData` in the simulation set up. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sppEquivCol </td>
   <td style="text-align:left;"> The column in `sim$specieEquivalency` data.table to use as a naming convention </td>
  </tr>
  <tr>
   <td style="text-align:left;"> validationReps </td>
   <td style="text-align:left;"> The simulation repetitions for the validation. Defaults to 1:10. Set to NA if not using repetitions (i.e. only one run) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> validationYears </td>
   <td style="text-align:left;"> The simulation years for the validation. Defaults to 2001 and 2011. Must select two years </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .plotInitialTime </td>
   <td style="text-align:left;"> If NA plotting is off completely (this includes saving). </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .plots </td>
   <td style="text-align:left;"> Passed to `types` in Plots (see ?Plots). There are a few plots that are made within this module, if set. Note that plots (or their data) are saved in file.path(outputPath(sim), 'figures'). If `NA`, plotting is off completely (this includes plot saving). </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .saveInitialTime </td>
   <td style="text-align:left;"> This describes the simulation time at which the first save event should occur </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .saveInterval </td>
   <td style="text-align:left;"> This describes the simulation time interval between save events </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .studyAreaName </td>
   <td style="text-align:left;"> Human-readable name for the study area used. If `NA`, a hash of `studyArea` will be used. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .useCache </td>
   <td style="text-align:left;"> Controls cache; caches the init event by default </td>
  </tr>
</tbody>
</table>

## Events

The following events take place during a *Biomass_validationKNN* run. Note that
this module only runs once (in one "timestep").

-   Module initiation (`init` event) - prepares both the observed and simulated
    data to be compared

-   Calculation of summary variables for validation (`calculateValidVars` event)

-   Calculation of validation statistics (`validationStats` event)

-   Optional diagnostic plots of biomass and age changes ($\Delta$B,
    $\Delta$Age) in the observed data (`obsDeltaMaps` event).

-   Plotting events:

    -   Plots of landscape-wide (`landscapeWidePlots` event), pixel-level
        (`pixelLevelPlots` events) comparisons of all summary variables

    -   Plots of biomass and age changes ($\Delta$B, $\Delta$Age) in observed
        and simulated data, with respect to the first year. (`deltaBComparisons`
        event)

### Module outputs

The module produces the following outputs (Table \@ref(tab:moduleOutputsBiomassValid)):

<table class="table" style="margin-left: auto; margin-right: auto;">
<caption>(\#tab:moduleOutputsBiomassValid)List of _Biomass_validationKNN_ output objects and their description.</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> objectName </th>
   <th style="text-align:left;"> desc </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> logLikelihood </td>
   <td style="text-align:left;"> A table of negative sum log-likelihood values calculated for different variables and averaged across repetitions. At the moment, log-likelihood values are calculated for biomass (landscape- and pixel-level), species presences and dominance (lanscape-level) and deltaB (landscape- and pixel-level. For biomass and count data (presences/dominance, we assume an underlying multinomial distribution, and for deltaB a multivariate Gaussian distribution - note that the later is still under development. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> landscapeMAD </td>
   <td style="text-align:left;"> Mean absolute deviance values calculated on landscape-level relative abundances, species presences and dominance, and deltaB, per repetition and year (except for deltaB, which is integrated across years) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> landscapeVars </td>
   <td style="text-align:left;"> A table containing observed and simulated landscape-averaged variables used for validation (by year and repetition, 'rep', in the case of simulated data), namely: species relative abundances ('relAbund'), species presenses ('count'), species dominance (as in no. pixels where a given species, has higher 'relAbund'; 'countDom') and species changes in biomass, as 2011 minus 2001 ('deltaB'). Observed data rows are labelled as 'observed' in 'dataType' column. In species dominance, pixels with &gt;= 2 species with max(B) and pixels with no B are classified as 'Mixed' and 'No veg.', respectively in the 'speciesCode' column - note that this is 'vegType' column in `pixelCohortData`. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pixelCohortData </td>
   <td style="text-align:left;"> A table containing observed and simulated pixel-level data (by year and repetition, 'rep', in the case of simulated data) on species biomass (summed across cohorts, 'B'), total pixel biomass ('pixelB'), average biomass-weighted pixel age ('pixelAge'), species relative abundance (calculated as B/pixelB, 'relativeAbund'), species dominance (the species with max(B), 'vegType'), and lanscape-wide biomass ('landscapeB'). Observed data columns are suffixed with 'Obsrvd'. In species dominance, pixels with &gt;= 2 species with max(B) (i.e. 'noDoms' &gt;= 2) are classified as 'Mixed'. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pixelMAD </td>
   <td style="text-align:left;"> Mean absolute deviance values calculated on pixel-level relative abundances and deltaB, per repetition and year (except for deltaB, which is integrated across years) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pixelVars </td>
   <td style="text-align:left;"> The same as `landscapeVars`, but variables are calculated at the pixel-level </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rstDisturbedPix </td>
   <td style="text-align:left;"> Raster of pixel IDs (as a mask) that have been disturbed by fire or suffered land-cover changes during the validation period. These pixels are excluded form the validation. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rawBiomassMapStart </td>
   <td style="text-align:left;"> observed total biomass raster layer in study area at the first year of the validation period. Filtered to exclude pixels that were disturbed during the validation period </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rawBiomassMapEnd </td>
   <td style="text-align:left;"> observed total biomass raster layer in study area at the last year of the validation period. Filtered to exclude pixels that were disturbed during the validation period </td>
  </tr>
  <tr>
   <td style="text-align:left;"> speciesLayersStart </td>
   <td style="text-align:left;"> observed percent cover raster layers by species in Canada at the first year of the validation period. Filtered to exclude pixels that were disturbed during the validation period </td>
  </tr>
  <tr>
   <td style="text-align:left;"> speciesLayersEnd </td>
   <td style="text-align:left;"> observed percent cover raster layers by species in Canada at the last year of the validation period. Filtered to exclude pixels that were disturbed during the validation period </td>
  </tr>
  <tr>
   <td style="text-align:left;"> standAgeMapStart </td>
   <td style="text-align:left;"> observed stand age map in study area, at the first year of the validation period Filtered to exclude pixels that were disturbed during the validation period </td>
  </tr>
  <tr>
   <td style="text-align:left;"> standAgeMapEnd </td>
   <td style="text-align:left;"> observed stand age map in study area, at the last year of the validation period Filtered to exclude pixels that were disturbed during the validation period </td>
  </tr>
</tbody>
</table>

and saves the validation figures in the output path defined in
`getPaths(sim)$outputPath` (which was defined as `paths$outputPath`). Tables are
not saved unless specified via `spades(outputs = data.frame(...), ...)`. If not
saving objects to disk (such as tables), these can be looked at using, e.g.,
`mySimValidation$logLikelihood`.

### Links to other modules

Intended to be used with *Biomass_core* and any other modules that link to it
and affect cohort biomass (e.g., via `cohortData` table). You can see all
*potential* module linkages within the LandR ecosystem
[here](https://rpubs.com/PredictiveEcology/LandR_Module_Ecosystem). Select
*Biomass_validationKNN* from the dropdown menu to see linkages with
*Biomass_validationKNN*.

### Getting help

-   <https://github.com/PredictiveEcology/Biomass_validationKNN/issues>

## Module manual

### Detailed description

This module compares simulated outputs of two years (across replicates), with
corresponding years of observed data. It was designed to compare the observed
data for years 2001 (start point for the simulation) and 2011 (i.e., after 10
years of simulation) of the kNN forest layers of the Canadian National Forest
Inventory -- these are currently the only available FAIR datasets [*sensu*
@StallEtAl2019]) on stand biomass and species % cover changes across Canada.
However, the user can supply other sources of observed data, as long as they
have an identical format.

The validation is done both visually (using barplots and boxplots) and using two
statistics: mean absolute deviation of simulated biomass (per species) and the
sum of negative log-likelihoods (SNLL) of predictions with respect to observed
data for species biomass, species presences/absences and changes in biomass
($\Delta$B) - the later is still under development.

This module assumes that the simulation data preparation was carried out by
*Biomass_borealDataPrep*, and so, to ensure that the comparison and the
simulated datasets are built with the same assumptions, the data treatment steps
in *Biomass_borealDataPrep* are repeated here. The module may also excludes
disturbed pixels coded in `rstLCCChange` and fire perimeter data
(`firePerimeters`). If this is not intended, pass a `rstLCCChange` with `NA`'s
only and/or an empty `firePerimeters` `sf` object. *Biomass_validationKNN* then
compares simulated species biomass, presences, dominance, and changes in biomass
against observed data available for the starting conditions (2011 by default)
and a second time point (e.g. 2011, or after 10 years of simulation). To do so,
for each year and replicate, and for both the simulated and observed data, the
module calculates:

-   species relative abundances at the pixel- and landscape-level (across all
    pixels)

-   species presences and dominance at the landscape level

-   changes in species biomass ($\Delta$B) at the pixel- and landscape-level for
    both the simulated and observed data. Biomass units respect those used in
    *Biomass_core* ($g/m^2$).

Pixel-level relative abundances are calculated as the species biomass (summed
across cohorts) divided by the total pixel biomass (summed across cohorts and
species), while landscape-wide relative abundances are calculated as the sum of
a species biomass across all pixels divided by the sum of total biomass across
all pixels. Species presences are calculated as the number of pixels where a
given species is present, and species dominance is calculated as the number of
pixels where a species has the highest relative biomass in a given pixel. Pixels
where two or more species share the highest biomass value are classified as
'mixed forest', and pixels without any biomass are classified as 'no veg.'.
Finally, ($\Delta$B) is calculated per species as the final biomass (e.g., year
2011) minus the initial biomass (e.g., year 2001), either at the pixel- or
landscape-level. All calculations were done per replicate.

### Validation approaches

#### Visual validation

The module plots the above metrics as barplots showing landscape-level values
(averaged across replicates for the simulated data) or boxplots showing
pixel-level values. Plotting can be live and/or in the form of exported images
(or both turned off completely).

#### Mean absolute deviation

Mean absolute deviance (MAD) values are calculated on landscape- and pixel-level
species relative abundances and $\Delta$B, and landscape-level species presences
and dominance, per replicate and year (except for $\Delta$B, which is integrated
across years). Output tables with MAD values are exported as `landscapeMAD` and
`pixelMAD`, and the module also produces visual inspection of these values as
dot-and-whisker plots.

#### Sum of negative log-likelihood (SNLL)

To provide a measure of overall goodness of fit of the simulation model, given a
given set of starting conditions and simulation mechanisms (i.e., the
combination of inputs to *Biomass_core* but also other modules that may be
associated in affecting vegetation dynamics), *Biomass_validationKNN* estimates
sum of negative log-likelihoods (SNLL) of simulated species biomasses, $\Delta$B
(both at the landscape and pixel-level), and species presences (at the
landscape-level), with respect to their observed counterparts. More precisely,
let $\ell$ be the log-likelihood function denoting the probability of observing
$x$ of $X$ (a random variable following a continuous probability distribution
$f(x)$), given a parameter $\theta$: 

\begin{equation}
  \ell(\theta \mid x) = f(x)
  (\#eq:loglik)
\end{equation}

In our case, $\theta$ is equivalent to the model's starting conditions and
structure, $X$ is the observed data with $x$ being the simulated values, and
$f(x)$ the continuous probability distribution of $X$. For each variable that we
wanted to evaluate and for each simulation replicate, Equation \@ref(eq:loglik)
is applied to calculate the SNLL estimated for each value of $x$ at the pixel or
landscape-level, $i$: 

\begin{equation}
  -\sum_{i = 1}^{N} \ell(\theta \mid x_{i})
  (\#eq:negsumloglik)
\end{equation}

where $N$ is equal to total number of pixels. At the landscape scale $N = 1$.

For species biomass and species presences, we draw the probability of observing
$x_{i}$ (a vector of species biomasses/presences in pixel/landscape $i$) from a
multinomial density distribution
($f(x_{i}) = {\sf Multi}(n_{i}, \mathrm{p}_{i})$), where
$n_{i} = \sum_{j = 1}^{K} X_{i,j}$ ($X$ being the observed values of biomass of
$j = 1, ..., K$ species in a pixel/landscape $i$) and $\mathrm{p_{i}}$ is the
vector of simulated values $x_{i,j}$.

**The computation of SNLL for $\Delta$B is still under development**. We have implemented the following, approach:
For $\Delta$B, we draw the
probability of observing $x_{i,j}$ (the simulated $\Delta$B of $j = 1, ..., K$
species in a pixel/landscape $i$) from a multivariate Gaussian distribution,
$f(x_{i}) = \mathcal {N}(\mu_{i}, \mathrm{M}_{i})$, where $\mu_{i}$ is the
vector of observed mean $\Delta$B for each species $j = 1, ..., K$, and
$\mathrm{M}$ is the observed $K * K$ variance-covariance matrix of species
$\Delta$B. Unfortunately this is presenting problems, due to
$\mathrm{M}$ not being strictly positive definite.

After calculating SNLL across pixels (or for a landscape), values are averaged
across replicates for an overall model estimate and exported in the
`logLikelihood` table.

We refer to the Wikipedia pages on the [multinomial
distribution](https://en.wikipedia.org/wiki/Multinomial_distribution) and on the
[multivariate Gaussian
distribution](https://en.wikipedia.org/wiki/Multivariate_normal_distribution#Density_function)
for a good summary of these two distributions and their use in SNLL estimation.

### Initialization, inputs and parameters

*Biomass_validationKNN* initializes itself and prepares all inputs provided that
it has access to outputs of simulations from Biomass_core, and internet access
to retrieve the observed kNN datasets used for validation. Future users should
run *Biomass_validationKNN* with defaults and inspect what the objects are like
before supplying their own data, or alternative data URLs. Alternatively, users
may develop their own validation modules using *Biomass_validationKNN* as a
template.

#### Input objects

*Biomass_validationKNN* requires the following input data layers: land-cover
change (change type and year), fire perimeters, % species cover, stand age and
stand biomass. By default, the module will take these from National Forest
Inventory kNN layers for years 2001 and 2011. We recommend that the user
supplies layers used to initialise the simulation as the starting input layers
(2001 if that is the starting point) to guarantee that they match. Table
\@ref(tab:moduleInputs2BiomassValid) shows the full list of input objects used by the
module.

<table class="table" style="margin-left: auto; margin-right: auto;">
<caption>(\#tab:moduleInputs2BiomassValid)List of _Biomass_validationKNN_ input objects and their description.</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> objectName </th>
   <th style="text-align:left;"> objectClass </th>
   <th style="text-align:left;"> desc </th>
   <th style="text-align:left;"> sourceURL </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> allCohortData </td>
   <td style="text-align:left;"> data.table </td>
   <td style="text-align:left;"> All `cohortData` tables saved during the simulation, particularly for the validation years. If not supplied, the module will attempt to retrieve them using the 'simulationOutputs' table </td>
   <td style="text-align:left;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> biomassMap </td>
   <td style="text-align:left;"> RasterLayer </td>
   <td style="text-align:left;"> total biomass raster layer in study area (in g/m^2), filtered for pixels covered by `cohortData`. Only used to calculate total no. of pixels being simulated If not supplied, will default to `rawBiomassMapStart` </td>
   <td style="text-align:left;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> firePerimeters </td>
   <td style="text-align:left;"> sf </td>
   <td style="text-align:left;"> A map of fire perimeters in the study area that can be used to exclude pixels that have been burnt during the validation period. If burnt pixels are not to be excluded Provide an empty `sf` object with the same properties as the default. Defaults to the latest Canadian Wildland Fire Information System National Burned Area Composite, subset to fires occuring up to last validation year (inclusively). Source URL determined by `fireURL` </td>
   <td style="text-align:left;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> fireURL </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:left;"> A URL to a fire database, such as the Canadian National Fire Database, that is a zipped shapefile with fire polygons, an attribute (i.e., a column) named 'Year'. If supplied (omitted with NULL or NA), this will be used to 'update' age pixels on `standAgeMap` with 'time since fire' as derived from this fire polygons map </td>
   <td style="text-align:left;"> https://cwfis.cfs.nrcan.gc.ca/downloads/nfdb/fire_poly/current_version/NFDB_poly.zip </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pixelGroupMapStk </td>
   <td style="text-align:left;"> RasterStack </td>
   <td style="text-align:left;"> A stack of `pixelGroupMap`s saved during the simulation, particularly for the validation years. If not supplied, the module will attempt to make it using the 'simulationOutputs' table </td>
   <td style="text-align:left;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rawBiomassMapStart </td>
   <td style="text-align:left;"> RasterLayer </td>
   <td style="text-align:left;"> observed total biomass raster layer in study area at the first year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived total aboveground biomass map from 2001 (in ton/ha). See https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 for metadata. </td>
   <td style="text-align:left;"> http://ftp.maps.canada.ca/pub/nrcan_rncan/Forests_Foret/canada-forests-attributes_attributs-forests-canada/2001-attributes_attributs-2001/NFI_MODIS250m_2001_kNN_Structure_Biomass_TotalLiveAboveGround_v1.tif </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rawBiomassMapEnd </td>
   <td style="text-align:left;"> RasterLayer </td>
   <td style="text-align:left;"> observed total biomass raster layer in study area at the last year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived total aboveground biomass map from 2011 (in ton/ha) See https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 </td>
   <td style="text-align:left;"> http://ftp.maps.canada.ca/pub/nrcan_rncan/Forests_Foret/canada-forests-attributes_attributs-forests-canada/2011-attributes_attributs-2011/NFI_MODIS250m_2011_kNN_Structure_Biomass_TotalLiveAboveGround_v1.tif </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rasterToMatch </td>
   <td style="text-align:left;"> RasterLayer </td>
   <td style="text-align:left;"> A raster of the `studyArea` in the same resolution and projection as `rawBiomassMapStart`. This is the scale used for all outputs for use in the simulation. </td>
   <td style="text-align:left;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rstLCChange </td>
   <td style="text-align:left;"> RasterLayer </td>
   <td style="text-align:left;"> A mask-type map of land cover changes in the study area that can be used to exclude pixels that have been disturbed during the validation period. If disturbed pixels are not to be excluded Provide an empty sf object with the same properties as the default. Defaults to Canada's forest change map between 1985-2011 (CFS), filtered for years 2001-2011 (inclusively) and all disturbances collapsed (map only has values of 1 and NA). See `P(sim)$LCChangeYr` parameter to change the period of disturbances, and https://opendata.nfis.org/mapserver/nfis-change_eng.html for more information. </td>
   <td style="text-align:left;"> https://opendata.nfis.org/downloads/forest_change/C2C_change_type_1985_2011.zip </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rstLCChangeYr </td>
   <td style="text-align:left;"> RasterLayer </td>
   <td style="text-align:left;"> An OPTIONAL map of land cover change years in the study area used to exclude pixels that have been disturbed during the validation period. It defaults to Canada's forest change year national map between 1985-2011 (CFS). If `P(sim)$LCChangeYr` is not `NULL`, this layer is used to filted disturbed pixels that fall within the years specified by `P(sim)$LCChangeYr`. If `P(sim)$LCChangeYr` is `NULL` this layer is not used. See https://opendata.nfis.org/mapserver/nfis-change_eng.html for more information. </td>
   <td style="text-align:left;"> https://opendata.nfis.org/downloads/forest_change/C2C_change_year_1985_2011.zip </td>
  </tr>
  <tr>
   <td style="text-align:left;"> simulationOutputs </td>
   <td style="text-align:left;"> data.table </td>
   <td style="text-align:left;"> An OPTIONAL table listing simulation outputs (as passed to `spades()`, or `experiment) that will be used to make `allCohortData`, `pixelGroupMapStk`, if these are not provided. </td>
   <td style="text-align:left;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> speciesLayersStart </td>
   <td style="text-align:left;"> RasterStack </td>
   <td style="text-align:left;"> observed cover percentage raster layers by species in Canada species map, at the first year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived species cover maps from 2001, using a cover threshold of 10% - see https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 for metadata </td>
   <td style="text-align:left;"> http://ftp.maps.canada.ca/pub/nrcan_rncan/Forests_Foret/canada-forests-attributes_attributs-forests-canada/2001-attributes_attributs-2001/ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> speciesLayersEnd </td>
   <td style="text-align:left;"> RasterStack </td>
   <td style="text-align:left;"> observed percent cover raster layers by species in Canada used for validation at the last year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived species cover maps from 2011 - see https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 for metadata </td>
   <td style="text-align:left;"> http://ftp.maps.canada.ca/pub/nrcan_rncan/Forests_Foret/canada-forests-attributes_attributs-forests-canada/2011-attributes_attributs-2011/ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sppColorVect </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:left;"> named character vector of hex colour codes corresponding to each species </td>
   <td style="text-align:left;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sppEquiv </td>
   <td style="text-align:left;"> data.table </td>
   <td style="text-align:left;"> table of species equivalencies. See `LandR::sppEquivalencies_CA`. </td>
   <td style="text-align:left;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left;"> standAgeMapStart </td>
   <td style="text-align:left;"> RasterLayer </td>
   <td style="text-align:left;"> observed stand age map in study area, at the first year of the validation period Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived biomass map from 2001 - see https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 for metadata </td>
   <td style="text-align:left;"> http://ftp.maps.canada.ca/pub/nrcan_rncan/Forests_Foret/canada-forests-attributes_attributs-forests-canada/2001-attributes_attributs-2001/NFI_MODIS250m_2001_kNN_Structure_Stand_Age_v1.tif </td>
  </tr>
  <tr>
   <td style="text-align:left;"> standAgeMapEnd </td>
   <td style="text-align:left;"> RasterLayer </td>
   <td style="text-align:left;"> observed stand age raster layer in study area, at the last year of the validation period. Defaults to the Canadian Forestry Service, National Forest Inventory, kNN-derived stand age map from 2011. See https://open.canada.ca/data/en/dataset/ec9e2659-1c29-4ddb-87a2-6aced147a990 </td>
   <td style="text-align:left;"> http://ftp.maps.canada.ca/pub/nrcan_rncan/Forests_Foret/canada-forests-attributes_attributs-forests-canada/2011-attributes_attributs-2011/NFI_MODIS250m_2011_kNN_Structure_Stand_Age_v1.tif </td>
  </tr>
  <tr>
   <td style="text-align:left;"> studyArea </td>
   <td style="text-align:left;"> SpatialPolygonsDataFrame </td>
   <td style="text-align:left;"> Polygon to use as the study area. Must be provided by the user </td>
   <td style="text-align:left;"> NA </td>
  </tr>
</tbody>
</table>

Of the inputs in Table \@ref(tab:moduleInputs2BiomassValid), the following are particularly
important and deserve special attention:

-   **Spatial layers**

    -   `biomassMap` -- a map of simulated stand biomass (in $g/m^2$ ) filtered
        for the pixels where cohort dynamics were simulated. This corresponds to
        the `sim$biomassMap` object produced by *Biomass_borealDataPrep* or to
        the `sim$simulatedBiomassMap` produced by *Biomass_core*.

    -   `firePerimeters` -- a fire perimeters polygon map that should be used to
        exclude recently burned pixels from the analysis. If this is not desired
        the user needs to provide an empty `sf` object (e.g.,
        `sf::st_polygon()`) .

    -   `rawBiomassMapStart` -- raw biomass data used to initialize and
        parametrize `Biomass_core`. By default, the module uses the stand
        biomass map from KNN for the year 2001. The user must make sure this
        appropriate for their use case, or else supply the correct raster layer.

    -   `rawBiomassMapEnd` -- raw biomass data used to validate the model after
        several simualtion years. By default, the module uses the stand biomass
        map from KNN for the year 2011, which is compared with the 10th year of
        a simulation initialised using the KNN 2001 data. The user must make
        sure this appropriate for their use case, or else supply the correct
        raster layer.

    -   `rstLCChange` -- a binary raster layer with disturbed pixels that should
        be removed from the analyses. Can be combined with `rstLCChangeYr` to
        filter pixels disturbed in a given time period defined by
        `P(sim)$LCChangeYr`. Defaults to [Canada's forest change national map
        between 1985-2011
        (CFS)](https://opendata.nfis.org/downloads/forest_change/C2C_change_type_1985_2011.zip).

    -   `rstLCChangeYr` -- a raster layer with year of disturbance. This is an
        optional layer that can be combined with `rstLCChange` and
        `P(sim)$LCChangeYr` to filter disturbed pixels by year of disturbance.
        Not used by default. Defaults to [Canada's forest change year national
        map between 1985-2011
        (CFS)](https://opendata.nfis.org/downloads/forest_change/C2C_change_year_1985_2011.zip).

    -   `speciesLayersStart` -- same as `rawBiomassMapStart`, but with respect
        to species % cover data.

    -   `speciesLayersEnd` -- same as `rawBiomassMapEnd`, but with respect to
        species % cover data.

    -   `studyArea` -- shapefile. A `SpatialPolygonsDataFrame` with a single
        polygon determining the where the simulation will take place. This is
        the only input object that **must be supplied by the user**.

-   **Simulation-related objects**

    -   `allCohortData` -- OPTIONAL. A `data.table` containing all `cohortData`
        objects relevant for the validation (e.g., as many `cohortData` objects
        as simulation replicates times 2, for the beginning and end year). If
        not supplied, *Biomass_validationKNN* attempts to produce this object
        using the `cohortData` object file listed in `simulationOutputs` .
        Hence, the user must either supply **both** `allCohortData` and
        `pixelGroupMapStk` **or** `simulationOutputs.`

    -   `pixelGroupMapStk` -- OPTIONAL. As `allCohortData` but with respect to
        `pixelGroupMap` objects.

    -   `simulationOutputs` -- OPTIONAL. A `data.frame` that has the same
        structure as the `data.frame`'s specifying outputs to be saved in
        `spades(…, outputs = data.frame(…))`. We advise passing the same
        `data.frame` that was supplied to `spades` during the simulation call,
        but filtered by the relevant `cohortData` and `pixelGroupMap` objects
        and, potentially, with file paths corrected to match the current working
        directory (see [Usage example]). Only used if `allCohortData` and
        `pixelGroupMapStk` are not supplied.

    -   `pixelGroupMap` -- a raster layer with *pixelGroup* IDs per pixel.
        Pixels are always grouped based on identical *ecoregionGroup*,
        *speciesCode*, *age* and *B* composition, even if the user supplies
        other initial groupings (e.g., this is possible in the
        *Biomass_borealDataPrep* data module).

#### Parameters

Table \@ref(tab:moduleParams2BiomassValid) lists all parameters used in
*Biomass_validationKNN* and their detailed information.

<table class="table" style="margin-left: auto; margin-right: auto;">
<caption>(\#tab:moduleParams2BiomassValid)List of _Biomass_validationKNN_ parameters and their description.</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> paramName </th>
   <th style="text-align:left;"> paramClass </th>
   <th style="text-align:left;"> default </th>
   <th style="text-align:left;"> min </th>
   <th style="text-align:left;"> max </th>
   <th style="text-align:left;"> paramDesc </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> coverThresh </td>
   <td style="text-align:left;"> integer </td>
   <td style="text-align:left;"> 10 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> The minimum % cover a species needs to have (per pixel) in the study area to be considered present. Should be the same as the one used to obtain the species cover layers for simulation set up. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> deciduousCoverDiscount </td>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> 0.8418911 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> This was estimated with data from NWT on March 18, 2020 and may or may not be universal. Should be the same as the one used when preparing `cohortData` in the simulation set up. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> LCChangeYr </td>
   <td style="text-align:left;"> integer </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 1900 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> OPTIONAL. An integer or vector of integers of the validation period years, defining which years of land-cover changes (i.e. disturbances) should be excluded. `NULL` by default, which presumes no subsetting based on years is done internally (either the user supplies a pre-filtered `rstLCChange`, or no filtering is desired). If not `NULL` `rstLCChangeYr` is used to filter disturbed pixels within the specified years. See https://opendata.nfis.org/mapserver/nfis-change_eng.html for more information. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> minCoverThreshold </td>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> 5 </td>
   <td style="text-align:left;"> 0 </td>
   <td style="text-align:left;"> 100 </td>
   <td style="text-align:left;"> Cover that is equal to or below this number will be omitted from the dataset Should be the same as the one used when preparing `cohortData` in the simulation set up. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> obsDeltaAgeB </td>
   <td style="text-align:left;"> logical </td>
   <td style="text-align:left;"> TRUE </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> When TRUE, the observed changes in biomass and age (deltaB, deltaAge) between the two validation years will be plotted as maps and scatterplots </td>
  </tr>
  <tr>
   <td style="text-align:left;"> pixelGroupBiomassClass </td>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> 100 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> When assigning `pixelGroup` membership, this defines the resolution of biomass that will be considered 'the same pixelGroup', e.g., if it is 100, then 5160 and 5240 will be the same Should be the same as the one used when preparing `cohortData` in the simulation set up. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sppEquivCol </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:left;"> Boreal </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> The column in `sim$specieEquivalency` data.table to use as a naming convention </td>
  </tr>
  <tr>
   <td style="text-align:left;"> validationReps </td>
   <td style="text-align:left;"> integer </td>
   <td style="text-align:left;"> 1, 2, 3,.... </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> The simulation repetitions for the validation. Defaults to 1:10. Set to NA if not using repetitions (i.e. only one run) </td>
  </tr>
  <tr>
   <td style="text-align:left;"> validationYears </td>
   <td style="text-align:left;"> integer </td>
   <td style="text-align:left;"> 2001, 2011 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> The simulation years for the validation. Defaults to 2001 and 2011. Must select two years </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .plotInitialTime </td>
   <td style="text-align:left;"> integer </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> If NA plotting is off completely (this includes saving). </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .plots </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:left;"> object, png </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> Passed to `types` in Plots (see ?Plots). There are a few plots that are made within this module, if set. Note that plots (or their data) are saved in file.path(outputPath(sim), 'figures'). If `NA`, plotting is off completely (this includes plot saving). </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .saveInitialTime </td>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> This describes the simulation time at which the first save event should occur </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .saveInterval </td>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> This describes the simulation time interval between save events </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .studyAreaName </td>
   <td style="text-align:left;"> character </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> Human-readable name for the study area used. If `NA`, a hash of `studyArea` will be used. </td>
  </tr>
  <tr>
   <td style="text-align:left;"> .useCache </td>
   <td style="text-align:left;"> logical </td>
   <td style="text-align:left;"> init </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> Controls cache; caches the init event by default </td>
  </tr>
</tbody>
</table>

Of the parameters listed in Table \@ref(tab:moduleParams2BiomassValid), the following are
particularly important:

-   `LCChangeYr` -- integer. Optional parameter defining the years of
    disturbance that should be filtered out of the analysis using the
    `rstLCChangeYr` layer. This parameter is set to `NULL` by default, meaning
    that `rstLCChangeYr` will not be used.

-   `sppEquivCol` -- character. the column name in `specieEquivalency`
    data.table that defines the naming convention to use throughout the
    simulation. The user needs to make sure the is

-   `validationReps` -- integer. which simulation replicates should be used for
    the validation.

-   `validationYears` -- integer. What simulation years should be used for the
    validation - the year number needs to match the observed data year. For
    instance, if the first observed data year is 2001, that must be the first
    simulation year.

### Simulation flow

The general flow of *Biomass_validationKNN* processes is:

1.  Preparation of all necessary objects, namely obtaining the observed data
    layers from online repositories (or if available stored local copies) and
    the compiling simulated data if the user has not done so previously (see
    [Input objects]).

2.  Calculation of summary variables for validation, namely:

    -   relative biomass per species per pixel and across the landscape (per
        year and per replicate)

    -   changes in species biomass per pixel and across the landscape (per
        replicate), with respect to the first year.

    -   species dominance across the landscape

    -   species presences across the landscape

3.  Calculation of validation statistics, namely mean absolute deviations (MAD)
    and sum of negative log-likelihoods (SNLL).

4.  Assessment of the relationship between observed $\Delta$B and observed
    $\Delta$Age -- this is an optional visual diagnostic of the observed data
    that produces scatterplots of $\Delta$B \~ $\Delta$Age of three types:

    -   With raw observed values of $\Delta$B and $\Delta$Age

    -   With $\Delta$B and $\Delta$Age calculated on observed data *after*
        pre-processing (i.e., the data clean-steps done in
        `Biomass_borealDataPrep`, which are also done to the observed data
        before validation)

    -   With the data shown in 2) above, but filtered by pixels where there was
        only a stand age increment corresponding to the number of years of
        between the two validation time points. This is not necessarily a
        *correct* filter, as stands may have suffered an age reduction due to
        the loss of old cohorts from background mortality (i.e., not coming from
        disturbances. However, if using the default input datasets, it is
        unlikely that this is a widespread phenomenon in only 10 years. We
        remind the user that disturbed pixels should be removed from the
        analyses when validating succession dynamics in the absence of
        disturbance - the default option.

5.  Plots:

    -   Barplots of landscape-wide and pixel-level comparisons between observed
        and simulated data, with respect to relative biomass, dominance and
        presences.

    -   Boxplots of biomass changes ($\Delta$B) in observed and simulated data,
        with respect to the first year.

    -   Maps of biomass and age changes ($\Delta$B, $\Delta$Age) with respect to
        the first year, in observed and simulated data.

All module default outputs are in the form of plots, but the user can chose to
save any objects (see Table \@ref(tab:moduleOutputsBiomassValid)).

## Usage example

### Load `SpaDES` and other packages.


```r
library(SpaDES)
library(SpaDES.install)
library(SpaDES.experiment)
library(future)
```

### Get the modules

Because *Biomass_validationKNN* is meant to validate simulation outputs against
observed data, we need to first run a simulation of forest dynamics with
*Biomass_core*. To do that we get both modules' code from the PredictiveEcology
GitHub repository. Notice that we are placing all module code, inputs and
outputs in temporary directories.


```r
tempDir <- tempdir()
spadesModulesDirectory <- file.path(tempDir, "modules")
paths <- list(inputPath = file.path(tempDir, "inputs"), 
                 cachePath = file.path(tempDir, "cache"), 
                 modulePath = spadesModulesDirectory, 
                 outputPath = file.path(tempDir, "outputs"))

getModule("PredictiveEcology/Biomass_core", modulePath = spadesModulesDirectory, overwrite = TRUE)
getModule("PredictiveEcology/Biomass_validationKNN", modulePath = spadesModulesDirectory, overwrite = TRUE)

## by default the repository branch name is appended to the module folder name.
## so we change the folder name to remove the "-master" suffix.
file.rename(c(file.path(spadesModulesDirectory, "Biomass_core-master"),
              file.path(spadesModulesDirectory, "Biomass_validationKNN-master")),
            c(file.path(spadesModulesDirectory, "Biomass_core"),
              file.path(spadesModulesDirectory, "Biomass_validationKNN")))
```

### Setup simulation


```r
times <- list(start = 2001, end = 2011)

studyArea <- Cache(randomStudyArea, size = 1e7) # cache this so it creates a random one only once on a machine

# Pick the species you want to work with -- using the naming convention in "Boreal" column of LandR::sppEquivalencies_CA
speciesNameConvention <- "Boreal"
speciesToUse <- c("Pice_Gla", "Popu_Tre", "Pinu_Con")

sppEquiv <- LandR::sppEquivalencies_CA[get(speciesNameConvention) %in% speciesToUse]
# Assign a colour convention for graphics for each species
sppColorVect <- LandR::sppColors(sppEquiv, speciesNameConvention,
                                 newVals = "Mixed", palette = "Set1")

## Usage example
modules <- as.list("Biomass_core")
objects <- list(studyArea = studyArea, sppEquiv = sppEquiv, sppColorVect = sppColorVect)

successionTimestep <- 20L

## keep default values for most parameters 
## (ommitted from this list)
parameters <- list(
  Biomass_core = list(
    "sppEquivCol" = speciesNameConvention
    , "successionTimestep" = successionTimestep
    , ".plotInitialTime" = times$start
    , ".plotInterval" = 1L
    , ".plots" = "png"
    , ".saveInitialTime" = times$start
    , ".useCache" = "init"
    , ".useParallel" = FALSE
  )
)

outputs <- data.frame(expand.grid(objectName = "cohortData",
                                  saveTime = unique(seq(times$start, times$end, by = 1)),
                                  eventPriority = 1,
                                  stringsAsFactors = FALSE))
outputs <- rbind(outputs, data.frame(objectName = "pixelGroupMap",
                                     saveTime = unique(seq(times$start, times$end, by = 1)),
                                     eventPriority = 1))
```

### Run simulation

Here we run a simulation with three replicates using the `experiment2` function
of the `SpaDES.experiment` R package [@McIntireChubaty2021], which builds a
folder structure where simulation outputs are conveniently organised.


```r
graphics.off()
mySimInit <- simInit(times = times,
                     params = parameters, 
                     modules = modules, 
                     objects = objects, 
                     paths = paths,
                     outputs = outputs)

plan(sequential)
mySimExperiment <- experiment2(
  sim1 = mySimInit,
  clearSimEnv = FALSE,
  replicates = 3)
```

### Validate simulation outputs with *Biomass_validationKNN*

Note that because we ran *Biomass_core* by itself using theoretical input data,
we can expect the validation to reveal that the module didn't do a great job at
reproducing observed patterns.


```r
simulationOutputs <- lapply(mySimExperiment, FUN = function(x, localSimPaths) {
  oldPath <- dirname(outputPath(x)) ## exclude sim*_rep* folder
  DT <- as.data.table(outputs(x))
  DT[, file := sub(oldPath, localSimPaths$outputPath, file)]
  DT
}, localSimPaths = as.list(normPath(paths)))
simulationOutputs <- rbindlist(simulationOutputs)

validationPaths <- as.list(normPath(paths))
validationPaths$outputPath <- file.path(validationPaths$outputPath, "validation")

validationTimes <- list(start = 1, end = 1)
validationParams <- list(
  Biomass_validationKNN = list(
    "sppEquivCol" = params(mySimInit)$Biomass_core$sppEquivCol
    , "validationReps" = as.integer(1:3)  ## or length of simLists
    , "validationYears" = as.integer(c(2001, 2011))
    , ".plots" = c("png")
  )
)

## make an empty fire polygon object to bypass removing fire-disturbed pixels
noFires <- sf::st_polygon()
validationObjects <- list(
  "biomassMap" = mySimExperiment$sim1_rep1$biomassMap
  , "firePerimeters" = noFires
  , "rasterToMatch" = mySimExperiment$sim1_rep1$rasterToMatch
  , "rawBiomassMapStart" = mySimExperiment$sim1_rep1$biomassMap
  , "simulationOutputs" = simulationOutputs
  , "speciesLayersStart" = mySimExperiment$sim1_rep1$speciesLayers
  , "sppColorVect" = mySimExperiment$sim1_rep1$sppColorVect
  , "sppEquiv" = mySimExperiment$sim1_rep1$sppEquiv
  , "studyArea" = mySimExperiment$sim1_rep1$studyArea
)

mySimValidation <- simInitAndSpades(times = validationTimes
                                    , params = validationParams
                                    , modules = "Biomass_validationKNN"
                                    , objects = validationObjects
                                    , paths = validationPaths
                                    , .studyAreaName = SAname)
```

Here are some of the output figures automatically produced by
*Biomass_validationKNN*

<div class="figure">
<img src="figures/LandscapeComparisons_PresAbs.png" alt="_Biomass_validationKNN_ automatically generates plots showing a visual comparison between simulated and observed species presences (right) across the landscape, and relative species biomass per pixel (left)." width="50%" /><img src="figures/PixelComparisons_relB.png" alt="_Biomass_validationKNN_ automatically generates plots showing a visual comparison between simulated and observed species presences (right) across the landscape, and relative species biomass per pixel (left)." width="50%" />
<p class="caption">(\#fig:figBiomassValidOutPlots)_Biomass_validationKNN_ automatically generates plots showing a visual comparison between simulated and observed species presences (right) across the landscape, and relative species biomass per pixel (left).</p>
</div>

<div class="figure">
<img src="figures/landscapeMAD.png" alt="A plot of landscape-wide mean absolute deviations (MAD) from (top to bottom) observed mean relative abundance, no. of presences, no. of pixels where the species is dominant and $\Delta$B." width="672" />
<p class="caption">(\#fig:figBiomassValidOutPlots2)A plot of landscape-wide mean absolute deviations (MAD) from (top to bottom) observed mean relative abundance, no. of presences, no. of pixels where the species is dominant and $\Delta$B.</p>
</div>

<div class="figure">
<img src="figures/observedDeltaBDeltaAge.png" alt="Diagnostic plot of observed changes in biomass and age $\Delta$B and $\Delta$Age, respectively)." width="672" />
<p class="caption">(\#fig:figBiomassValidOutPlots3)Diagnostic plot of observed changes in biomass and age $\Delta$B and $\Delta$Age, respectively).</p>
</div>

## References