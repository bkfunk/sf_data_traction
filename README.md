SF Data Traction
========

SF provides a lot of interesting data [online](https://data.sfgov.org/), including data on all incidents reported to SFPD (including incidents reported by citizens and by officers themselves).[\*](#note1) The actual data file, which contains incidents since 2003, can be found [here](http://apps.sfgov.org/datafiles/view.php?file=Police/sfpd_incident_all_csv.zip). Note that this is a broader universe than, say, the FBI's [Uniform Crime Reporting](http://www.fbi.gov/about-us/cjis/ucr/ucr) database, since it includes reported incidents that were deemed by officers to not, in fact, be criminal activity (as well as warnings, etc.). In particular, the data includes latitude and longitude fields for all incidents.

I wanted to see what I could learn from this data, in particular how crime reporting relates to a number of other characteristics. Here, I match incidents for 2012 to Census-Tract-level data from the US Census's 2012 American Community Survey. This allows me to turn incidents into _incident rates_, since I have population by census tract. It also allows me to look at, say, the relationship between poverty levels and crime rates, or the relationship between the percentage of people who don't have a car and the car theft rate. In order to visualize this disparate data, I create a combination scatter plot and map using d3.js. The final visualization can be viewed here: <http://bkfunk.github.io/sf_data_traction/crimestats.html> [(source)](https://github.com/bkfunk/sf_data_traction/blob/master/crimestats.html).

## Cleaning the Data
The data cleaning scripts are located in the [`/scripts/`](https://github.com/bkfunk/sf_data_traction/tree/master/scripts) directory.
+ `clean_census.py` loads several data files downloaded from the Census, renaming variables and merging the datasets together.
+ `prep_incidents.py` loads the CSV files downloaded from data.sfgov.org and uses the latitude and longitude fields to associate the appropriate Census tract ID, using the FCC's Census Bock API to do the geocoding.
+ `analyze_crimes.py` merges the SFPD data with the Census data, cleans certain variables, drops other variables, and then creates 3 output CSV fles:
  - `inc_2012.csv` -- all incidents in 2012
  - `res_2012.csv` -- all _resolved_ incidents in 2012
  - `arr_2012.csv` -- all _arrests_ in 2012
  Currently, I only look at the incidents file (`inc_2012`)

## Visualizing the Data
I create some plot templates (located in [`lib/charts/`](https://github.com/bkfunk/sf_data_traction/tree/master/lib/charts)) to ease plotting. `crimestats.html` uses these templates to create a scatter plot, a brush-able distribution plot for each axis on the scatter, and a map showing a choropleth of the data.

I got a shapefile for the SF Census Tracts from the [US Census site](http://www2.census.gov/geo/tiger/GENZ2010/gz_2010_06_140_00_500k.zip). I then converted the shapefile to TopoJSON using the `bin/create_topojson` bash script. I load that TopoJSON file, along with the incidents data, into d3, and merge those two in the javascript so that both geo data and the incident data are bound to each tract element.

Using the visualization, you can select variables to be plotted on:
+ The scatter plot's x-axis
+ The scatter plot's y-axis
+ The map's choropleth

All three variables can be set indipendently, meaning you can visualize three separate variables in three ways, or one variable on the scatter plot and another on both the scatter plot and choropleth. This allows you to describe some complex relationships in a number of ways.

Mousing over a point in the scatter plot will highlight the corresponding Census tract on the map, and vice versa. Using the brushes on the axis density plots, you can select only a portion of each of the scatter's variables' distributions. For example, you could zoom in on the y-values in the lower part of the distribution. On the choropleth at the right, any tracts that are no longer visible on the scatter plot will switch from a green color scale to a gray color scale. This allows you to, say, zoom in on tracts with high rates of assault and view the distribution of percentage of households with young residents within those high-assault areas and outside those areas.

The d3 code is intentionally pretty agnostic about the precise nature of the data. In order to reuse the visualization with different data, you would just need to get other census-tract-level data with a "tract_id" and another TopoJSON file with the tracts for a different city or region. With a little modification, you could use any type of data associated with any type of map boundary.



<a id="note1">*</a>__NB__: Per the dataset owner's post in the discussion on the [data site](https://data.sfgov.org/Public-Safety/SFPD-Reported-Incidents-2003-to-Present/dyj4-n68b), "The only data not included in this database is the homicide data."
