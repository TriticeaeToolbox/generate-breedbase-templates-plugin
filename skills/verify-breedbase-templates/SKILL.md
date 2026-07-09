---
name: verify-breedbase-templates
description: Verify the generated breedbase upload templates and check for errors against the original breeder data
allowed-tools: Bash(python:*) Bash(python3:*) Read
---

## Overview

This skill is used to check the validity of the generated upload templates that were created by a previous AI skill against the original breeder-provided data files.  The generated files are in the `upload_templates/` subdirectory and should contain an `accessions.csv`, `trials.csv`, and `observations.csv` file.  The original breeder data can be in any format and may be in one or more files and each file may contain one or more worksheets.

This skill should check each of the generated files to make sure the information in the upload templates is correct and is not missing any data that was in the original breeder-provided data files.

## Checks

### Accessions

The generated `accessions.csv` file contains the germplasm / variety metadata for all of the accessions observed in the trial data.

The following checks should be performed:

- Make sure all of the accessions from the original breeder data are included
- Make sure each accession entry is included only once in the upload template (no duplicates)
- Make sure every accession has the proper species set
- Make sure any synonyms found in the original data (such as alternate names included in parantheses) are included
- Make sure the purdy pedigree string is set if the pedigree is included in the original data
- If the pedigree is a simple A/B string, include that as parsed female_parent and male_parent in the upload template

### Trials

The generated `trials.csv` file contains the trial-level and plot-level metadata for each year/location combination in the original breeder data.

The following checks should be performed:

- Make sure every year/location combination in the original breeder data is included
- The `trial_name` follows the format of "experiment_year_location", where experiment is the user-provided experiment code, year is the 4 digit harvest year, and location is the town name of the location.  The `trial_name` should not include any spaces or any special characters other than letters, numbers, and underscores.
- Make sure the `breeding_program` is set to the name of the breeding program
- Make sure the `location` is set to the full "Town, ST" name (such as "Fargo, ND", "Ithaca, NY" or "Brandon, MB").
- Make sure the `year` is set to the 4-digit harvest year
- Make sure the `design_type` is set to one of the approved values
- Make sure the `description` is set and includes any additional notes from the breeder.
- Make sure the `accession_name` is set and matches the name of an accession in the `accessions.csv` file
- Make sure the `plot_number` is set and matches the plot number given in the breeder data or the standard convention for plot numbering: 1, 2, 3, etc for single rep trials OR 101, 102, 103, etc, 201, 202, 203, etc for multi-rep trials
- Make sure the `block_number` is set and the `rep_number` is set for multi-rep trials.
- Make sure the `plot_name` is set and includes the `trial_name` and `plot_number` and follows the convention of "trial_name-PLOT_plot_number".  The `plot_name` should not include any spaces or any special characters other than letters, numbers, and underscores.
- Make sure the `trial_type` is set, most likely to "phenotyping_trial"
- Double check the original breeding data for planting and harvest dates since these are very important to include if they are recorded.  The planting date should be added to the `planting_date` column and the harvest date should be added to the `harvest_date` column if included in the original data.
- If controls / checks are labelled in the original data, set the `is_a_control` column to 1 for each plot that contains a control / check accession.
- If the original data includes plot coordinates (such as range, row, or column positions), include that information in the `row_number` and `col_number` columns.

### Observations

The generated `observations.csv` file contains all of the trait observations for every plot in all of the trials.

The following checks should be performed:

- Make sure every plot from the original breeder's data is included in the `observations.csv` file in the `observationunit_name` column
- Make sure every plot name matches a plot in the generated `trials.csv` upload template
- Check the trait mapping (more details below)
- Check the trait conversions (more details below)

#### Trait Mapping

It is very important to check the mapping of trait terms used by the breeder to the standardized trait ontology terms for the specific crop that is being studied in this set of trials.

The following trait mapping checks should be made:

- For every trait in the original breeders data, check to make sure the correct trait ontology term is matched to the original trait term.  Check any definitions / descriptions used in the original data against the definitions used in the trait ontology.
- If you're not positive if the correct trait ontology term has been used, ask the user to double check the mapping
- If there is no good matching trait ontology term, use the original trait term from the breeder's data as the column header
- Generate a summary table of all trait terms used in the original data and their matching trait ontology term

#### Trait Conversions

It is very important that the trait values match the units / scale of the trait ontology term.  In some cases, the breeder data might be in a different scale than the trait ontology term.  In this case, the values have to be converted from the original scale to the trait ontology scale.

The following trait conversion checks should be made:

- Check if the trait values are assigned to the correct plot and trait column
- Check if the trait value has been converted.  If it has been converted, make sure the correct formula has been used to convert the trait
- If you are unsure what scale the original data is in, ask the user to confirm the scale
- If you are unsure of the formula to use to convert the data between different units, ask the user to confirm the formula
- Some conversion formulas differ depending on the crop - make sure the correct crop-specific conversion factor is being used
- Generate a summary table of all of the conversions that have been used for all of the traits
- The breeder may use a code or value to represent null / empty values (such as NA, ND, ., -9).  Remove these placeholders and keep the cell in the observations template empty for these values.  Generate a list of values that you removed.  If you are unsure if a value represents a null / empty value, ask the user to confirm.
