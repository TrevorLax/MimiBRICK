# MimiBRICK.jl

This repository will have the latest "standard" BRICK version and codes demonstrating its calibration in three configurations: (i) alone (forced by temperature and ocean heat), (ii) coupled to DOECLIM, and (iii) coupled to SNEASY. Standard calibration output will be provided as well as examples and scripts for downscaling the projections to a 1-degree grid for local sea-level rise.

<br>

## Statement of Need

MimiBRICK.jl is an implementation of the [Building Blocks for Relevant Ice and Climate Knowledge (BRICK) semi-empirical model for sea-level change](https://doi.org/10.5194/gmd-10-2741-2017) in the Mimi integrated modeling framework (https://www.mimiframework.org/). The Mimi modeling framework is a coding platform that facilitates coupling models and running coupled modeling experiments. MimiBRICK.jl is flexible, efficient, and modular, to facilitate incorporating BRICK into coupled models and integrated assessments of climate impacts in a modular fashion to provide global average as well as local sea-level projections. This focus on tight model coupling and integrated modeling is a key feature of MimiBRICK.jl and broader Mimi modeling framework.

This implementation includes examples for using observational data to calibrate the model, as well as various configurations in which MimiBRICK.jl is coupled to other climate model components. For users who do not wish to re-run computationally intensive model calibration algorithms, this implementation also includes scripts for using existing calibration output for standard future climate change scenarios, and examples downscaling these global projections for assessments of local impacts. Pre-run model calibration and simulation output can be found in the [accompanying Zenodo repository](https://zenodo.org/record/6626335).

<br>

## Recommended Use

* If you would like to use previously published BRICK results as part of a new analysis, but do not necessarily want to re-run the model, then we recommend to go to the [accompanying Zenodo repository](https://zenodo.org/record/6626335). In the near future, we will compile a library of other previously published studies using BRICK, including any other relevant outputs. 
* If you would like to run the model yourself, then you are in the right place!
  * It is assumed that users will not clone/download this Git repository. Instead, you can add and use the package as described below. Those commands and the commands to set up and run the model can be executed from the directories that contain the rest of your project codes.
  * Load the MimiBRICK package (or if you wish to edit the package either fork the master branch from this repository or download the zipped file of codes)
  * Do your analysis
  * Also please let us know of any model modifications and/or bug fixes that might usefully be incorporated into the main BRICK codes. Creating an "Issue" here is a great way to do that (top horizontal menu bar in the GitHub browser).

## How To Install MimiBRICK

This code was created using [Julia v1.6](https://julialang.org/downloads/) and requires several Julia packages. It is recommended that you use Julia v1.6 (or later). Julia may be downloaded from http://julialang.org/downloads/.

(1) Run the following line to install the Mimi implementation of BRICK:

```julia
]
add MimiBRICK
```

(2) To exit back to Julia, hit the `backspace` key.

<br>

## Running baseline cases with default parameters and unit tests

If you would like to take a look at the unit tests run with Continuous Integration for this package, feel free to take a look at the `runtests.jl` file, and the separate testing scripts it calls.

### More detailed examples

Below, we review the three main configurations of the model that we anticipate being used. However, in the `examples` directory, you can find further examples conducting the statistical calibration, creating posterior model hindcasts and projections, and downscaling these hindcasts and projections to local sea-level changes. Those examples are in Jupyter notebooks. To open and run one, you will need to clone or fork the MimiBRICK.jl repository. Then, navigate to it in the Julia terminal and run:
```julia
using Pkg; Pkg.add(IJulia)
using IJulia; notebook(dir="examples")
```
You can then select and run the Jupyter notebook of your chosing in the Jupyter browser window (which will open in a web browser).

### BRICK standalone (with temperature and ocean heat uptake exogenous forcing)

This is the first test that is done in `test/runtests/test_default.jl`. Since it does not require DOECLIM or SNEASY, you can run BRICK using temperature and ocean heat uptake forcing data by running in the Julia console:
```julia
using MimiBRICK
m = MimiBRICK.get_model()
run(m)
```

You can plot the output fields in the model object `m` using (for example) the `Plots` Julia plotting package. First, let's grab the years over which the model was run. This is a dimension in the model. We can retrieve it by using the `dim_keys()` function, from the `Mimi` package.
```julia
using Mimi
years = dim_keys(m, :time)
```

Then we can load the `Plots` package and make a figure of the global mean sea-level change. Note that the first argument into the `m` object specifies the component of our model, and the second argument specifies the field. Here, we are grabbing the `sea_level_rise` field from the `global_sea_level` component.
```julia
using Plots
plot(years, m[:global_sea_level, :sea_level_rise])
```

Mimi also offers an explorer window to check these model output fields out. To use this, we need to load the `Mimi` package (if you haven't already).
```julia
using Mimi
```

Then, we can open the explorer.
```julia
explore(m)
```

This should open a window labeled "Mimi Explorer Window". On the left, there should be two vertically-stacked boxes. The top box is labeled "Components" and the bottom box is labeled "Data". To view some of the model output from our out-of-box BRICK simulation, you must first pick one of the Components from the top box, and then a Variable field out of the bottom box. For example, to view the global mean sea level model projections, select `global_sea_level` from the Components box, and `global_sea_level : sea_level_rise` from the Variables box. A plot of this variable should appear in the right column of plot boxes. The top box is static; the bottom box you can interact with to zoom in on different portions of the figure.

More information about exploring Mimi model results can be found in the [Mimi Framework How-To guides online](https://www.mimiframework.org/Mimi.jl/stable/howto/howto_2/).

### DOECLIM-BRICK (with radiative forcing)

A simulation using DOECLIM to model temperature and ocean heat uptake, coupled to BRICK for sea-level rise can be constructed and run using the `create_brick_doeclim()` function. This function uses the `MimiSNEASY` package, which includes the DOECLIM model. Here we construct the model object `m` using the `create_brick_doeclim()` constructor, and run the model using the standard Mimi `run(m)`.

```julia
m = MimiBRICK.create_brick_doeclim()
run(m)
```

These are using the default arguments in the model constructor for the DOECLIM-BRICK model:
* `rcp_scenario = "RCP85"` - using Representative Concentration Pathway 8.5 as a default; other options include `RCP26`, `RCP45`, and `RCP60`
* `start_year = 1850` - starting year of the model simulation
* `end_year = 2020` - ending year of the model simulation

So, if you wanted to instead run DOECLIM-BRICK using RCP 6.0 from 1800 to 2100, you could run:
```julia
m = MimiBRICK.create_brick_doeclim(rcp_scenario="RCP60", start_year=1800, end_year=2100)
run(m)
```

And of course, you can use `explore(m)` to check out the model outputs attached to the model object `m` to verify that we have in fact changed RCP scenario and time periods. (Reminder: you'll need to have loaded the `Mimi` package using `using Mimi` to access the `explore()` function.)

### SNEASY-BRICK (with emissions forcing)

Running a coupled model using SNEASY and BRICK proceeds in the same way as DOECLIM-BRICK. We can use the constructor for the coupled SNEASY-BRICK model, create, then run the coupled model.
```julia
m = MimiBRICK.create_sneasy_brick()
run(m)
```

The `create_sneasy_brick()` constructor has the same arguments as the DOECLIM-BRICK constructor, so you can change the RCP scenario and the time period.

<br>

## Author Contributions

* TW: initial model development, software development, model calibration and validation, conceptualization, projection direction and overall management
* LR: software development, package maintenance, conceptualization
* FE: software development, model calibration and validation, conceptualization
* VS: software testing, model calibration and validation, conceptualization
* AB: initial model development, software testing, conceptualization
* KK: software testing, conceptualization
* DA: software development, package maintenance, conceptualization, project direction

<br>

## License

Copyright 2022 Tony Wong, Lisa Rennels, Frank Errickson, Vivek Srikrishnan, Alexander Bakker, Klaus Keller, and David Anthoff

This file and codes in this repository are part of MimiBRICK.jl (Building blocks for Relevant Ice and Climate Knowledge). MimiBRICK.jl is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

MimiBRICK.jl is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with MimiBRICK.jl (`LICENSE.md)`). If not, see http://www.gnu.org/licenses/.

