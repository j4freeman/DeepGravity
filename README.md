# A Deep Gravity model for mobility flows generation

## Table of contents
1. [Citing](#citing)
2. [Abstract](#abstract)
3. [Architecture of Deep Gravity](#architecture)
4. [Running Deep Gravity](#running)
	  - [Setup](#setup)
	  - [Experiments](#experiments)
    - [Plot the results](#plot)
    - [Additional data](#additional_data)


<a id='citing'></a>
## Citing

If you use the code in this repository, please cite our paper:

F. Simini, G. Barlacchi, M. Luca, L. Pappalardo, *A Deep Gravity model for mobility flows generation*, Nature Communications 12, 6576 (2021). https://doi.org/10.1038/s41467-021-26752-4

```
@article{Simini2021,
author = {Simini, Filippo and Barlacchi, Gianni and Luca, Massimilano and Pappalardo, Luca},
doi = {10.1038/s41467-021-26752-4},
issn = {2041-1723},
journal = {Nature Communications},
number = {1},
pages = {6576},
title = {{A Deep Gravity model for mobility flows generation}},
url = {https://doi.org/10.1038/s41467-021-26752-4},
volume = {12},
year = {2021}}
```

and the official code repository: [![DOI](https://zenodo.org/badge/379271033.svg)](https://zenodo.org/badge/latestdoi/379271033)

<a id='abstract'></a>
## Abstract
The movements of individuals within and among cities influence critical aspects of our society, such as well-being, the spreading of epidemics, and the quality of the environment. When information about mobility flows is not available for a particular region of interest, we must rely on mathematical models to generate them. We propose Deep Gravity, an effective model to generate flow probabilities that exploits many features (e.g., land use, road network, transport, food, health facilities) extracted from voluntary geographic data, and uses deep neural networks to discover non-linear relationships between those features and mobility flows. Our experiments, conducted on mobility flows in England, Italy, and New York State, show that Deep Gravity achieves a significant increase in performance, especially in densely populated regions of interest, with respect to the classic gravity model and models that do not use deep neural networks or geographic data. Deep Gravity has good generalization capability, generating realistic flows also for geographic areas for which there is no data availability for training. Finally, we show how flows generated by Deep Gravity may be explained in terms of the geographic features and highlight crucial differences among the three considered countries interpreting the model’s prediction with explainable AI techniques.

![Performances of DG vs G in an highly populated area in England](https://github.com/scikit-mobility/DeepGravity/blob/master/imgs/plot.png?raw=true)
_Figure 1. Performances in terms of Common Part of Commuters (CPC) of Deep Gravity (DG) vs the gravity model (G) in an highly populated area in England_


<a id='architecture'></a>
## Architecture of Deep Gravity
To generate the flows from a given origin location (e.g., <img src="https://render.githubusercontent.com/render/math?math=l_i">), Deep Gravity uses a number of input features to compute the probability <img src="https://render.githubusercontent.com/render/math?math=p_{i,j}"> that any of the <img src="https://render.githubusercontent.com/render/math?math=n"> locations in the region of interest (e.g., <img src="https://render.githubusercontent.com/render/math?math=l_j">) is the destination of a trip from <img src="https://render.githubusercontent.com/render/math?math=l_i">. Specifically, the model output is a n-dimensional vector of probabilities <img src="https://render.githubusercontent.com/render/math?math=p_{i,j}"> for <img src="https://render.githubusercontent.com/render/math?math=j = 1, ..., n">. These probabilities are computed in three steps (see figure below).

![Architecture of Deep Gravity](https://github.com/scikit-mobility/DeepGravity/blob/master/imgs/architecture.png?raw=true)
_Figure 2. Architecture of Deep Gravity_

1. The input vectors <img src="https://render.githubusercontent.com/render/math?math=x(l_i, l_j) = concat[x_i, x_j, r_{i,j}]"> for <img src="https://render.githubusercontent.com/render/math?math=j =1, \dots, n"> are obtained performing a concatenation of the following input features: <img src="https://render.githubusercontent.com/render/math?math=x_i">, the feature vector of the origin location <img src="https://render.githubusercontent.com/render/math?math=l_i">; <img src="https://render.githubusercontent.com/render/math?math=x_j"> the feature vector of the destination location <img src="https://render.githubusercontent.com/render/math?math=l_j">; and the distance between origin and destination <img src="https://render.githubusercontent.com/render/math?math=r_{i, j}">. 
For each origin location (e.g. <img src="https://render.githubusercontent.com/render/math?math=l_i">), <img src="https://render.githubusercontent.com/render/math?math=n"> input vectors <img src="https://render.githubusercontent.com/render/math?math=x(l_i, l_j)"> with <img src="https://render.githubusercontent.com/render/math?math=j = 1, \dots, n"> are created, one for each location in the region of interest that could be a potential destination. 

2. The input vectors <img src="https://render.githubusercontent.com/render/math?math=x(l_i, l_j)"> are fed in parallel to the same feed-forward neural network. The network has 15 hidden layers of dimensions 256 (the bottom six layers) and 128 (the other layers) with LeakyReLu activation function, <img src="https://render.githubusercontent.com/render/math?math=a">. Specifically, the output of hidden layer <img src="https://render.githubusercontent.com/render/math?math=h"> is given by the vector <img src="https://render.githubusercontent.com/render/math?math=z^{(0)}(l_i, l_j) = a(W^{(0)} \cdot x(l_i, l_j))"> for the first layer (<img src="https://render.githubusercontent.com/render/math?math=h=0">) and <img src="https://render.githubusercontent.com/render/math?math=z^{(h)}(l_i, l_j) = a(W^{(h)} \cdot z^{(h - 1)}(l_i, l_j))"> for <img src="https://render.githubusercontent.com/render/math?math=h>0">, where <img src="https://render.githubusercontent.com/render/math?math=W"> are matrices whose entries are parameters learned during training. 

3. The output of the last layer is a scalar <img src="https://render.githubusercontent.com/render/math?math=s(l_i, l_j) \in[-\infty, +\infty]"> called score: the higher the score for a pair of locations <img src="https://render.githubusercontent.com/render/math?math=(l_i, l_j)">, the higher the probability to observe a trip from <img src="https://render.githubusercontent.com/render/math?math=l_i"> to <img src="https://render.githubusercontent.com/render/math?math=l_j"> according to the model. Finally, the scores are transformed into probabilities using a softmax function, <img src="https://render.githubusercontent.com/render/math?math=p_{i,j} = e^{s(l_i, l_j)} / \sum_{k} e^{s(l_i, l_k)}">, which transforms all scores into positive numbers that sum up to one. The generated flow between two locations is then obtained by multiplying the probability (i.e., the model's output) and the origin's total outflow.

The location feature vector <img src="https://render.githubusercontent.com/render/math?math=x_i"> provides a spatial representation of an area, and it contains features describing some properties of location <img src="https://render.githubusercontent.com/render/math?math=l_i">, e.g., the total length of residential roads or the number of restaurants therein. Its dimension, <img src="https://render.githubusercontent.com/render/math?math=d">, is equal to the total number of features considered. The location features we use include the population size of each location and geographical features extracted from OpenStreetMap belonging to the following categories:

- Land use areas (5 features): total area (in squared km) for each possible land use class, i.e., residential, commercial, industrial, retail and natural;
- Road network (3 features): total length (in km) for each different types of roads, i.e., residential, main and other; 
- Transport facilities (2 features): total count of Points Of Interest (POIs) and  buildings related to each possible transport facility, e.g., bus/train station, bus stop, car parking;
- Food facilities (2 features): total count of POIs and  buildings related to food facilities, e.g., bar, cafe, restaurant;
- Health facilities (2 features): total count of POIs and  buildings related to health facilities, e.g., clinic, hospital, pharmacy;
- Education facilities (2 features): total count of POIs and  buildings related to education facilities, e.g., school, college, kindergarten; 
- Retail facilities (2 features): total count of POIs and  buildings related to retail facilities, e.g., supermarket, department store, mall.

In addition, Deep Gravity includes as feature the geographic distance, <img src="https://render.githubusercontent.com/render/math?math=r_{i, j}">, between two locations <img src="https://render.githubusercontent.com/render/math?math=l_i"> and <img src="https://render.githubusercontent.com/render/math?math=l_j">, which is defined as the distance measured along the surface of the earth between the centroids of the two polygons representing the locations. All values of features for a given location (excluding distance) are normalized dividing them by the location's area.

Each flow in Deep Gravity is hence described by 39 features (18 geographic features of the origin and 18 of the destination, distance between origin and destination, and their populations). 

The loss function of Deep Gravity is the cross-entropy: 

<img src="https://render.githubusercontent.com/render/math?math=H = - \sum_{i} \sum_j \frac{y(l_i, l_j)}{O_i} \ln p_{i,j}">

where <img src="https://render.githubusercontent.com/render/math?math=y(l_i, l_j) / O_i"> is the fraction of observed flows from <img src="https://render.githubusercontent.com/render/math?math=l_i"> that go to <img src="https://render.githubusercontent.com/render/math?math=l_j"> and <img src="https://render.githubusercontent.com/render/math?math=p_{i, j}"> is the model's probability of a unit flow from <img src="https://render.githubusercontent.com/render/math?math=l_i"> to <img src="https://render.githubusercontent.com/render/math?math=l_j">. 
Note that the sum over <img src="https://render.githubusercontent.com/render/math?math=i"> of the cross-entropies of different origin locations follows from the assumption that flows from different locations are independent events, which allows us to apply the additive property of the cross-entropy for independent random variables. 

The network is trained for 20 epochs with the RMSprop optimizer with momentum 0.9 and learning rate <img src="https://render.githubusercontent.com/render/math?math=5 \cdot 10^{-6}"> using batches of size 64 origin locations. To reduce the training time, we use negative sampling and consider up to 512 randomly selected destinations for each origin location. 

<a id='running'></a>
## Running Deep Gravity

<a id='setup'></a>
### Setup
Make sure you have the following dependencies installed:

- `pytorch 1.7.1`
- `numpy 1.19.2`
- `pandas 1.2.4`
- `geopandas 0.9.0`
- `scikit-mobility 1.1.0`
- `area`

<a id='experiments'></a>
### Experiments

Once you installed all the packages correctly, you can run the experiments.

We expect to find some datasets in a path named `data/<country_name>` where country name is a parameter that can be passed to the model. In particular, we expect to find:

- `tessellation.geojson` or `tessellation.shp`. The tessellation can also be generated by using the parameters `tessellation-area` and `tessellation-size` when the model is called.
- `output_areas.geojson` or `output_areas.shp`. A file containing the location code and the geometry of the output areas. the column containing the location code can be specified using the parameter `oa-id-column` when calling the model.
- `flows.csv` containing three columns indicating the origin, destination and the actual flow of people. The columns with the information can be called specifying the parameters `flow-origin-column`, `flow-destination-column` and `flow-flows-column`. Due to GitHub policy, the file containing the flows for the running example of New York have to be downloaded from [here](here)
- `features.csv` containing at least a column named like `oa-id-column` and a set of other columns representing the features of the model

An example of dataset collected in New York is already loaded in the repository and the following examples are based on that. Note that when main.py is launched for the first time, a set of additional files are generated in a folder called `processed`. These files should not be removed.

The model can be run with the following command:

`python main.py --dataset new_york --oa-id-column GEOID --flow-origin-column geoid_o --flow-destination-column geoid_d --flow-flows-column pop_flows --epochs 1 --device cpu --mode train`

you can also include some parameters related to the model:

- `batch-size` to specify the input batch size for training. Deafult is 1 
- `test-batch-size` to specify the batch size at test time. Default is 1
- `epochs` default is 10 
- `lr` that is the learning rate. Default is 5e-6 
- `momentum`  default is 0.9
- `seed` 
- `device` can be `cpu` or `gpu` 
- `mode` that can be `train` or `test` 

There are also some parameters related to the 

Once your model is trained, you will find the results of the test phase in a file in the results directory. The file will be named `tile2cpc_<model-type>_<country>_<no-round>.csv`. In the same folder, you will also find the trained model named `model_<model-type>_<country>_<no-round>.pt`

<a id='plot'></a>
### Plot of the results

Once you have the results for all the four models in at least a country and at least for one no-round, you can reproduce Figure 3 and Table 1 of the paper using the notebook `plot_results.ipynb`

<a id='additional_data'></a>
### Additional Data

The datasets used in the experiments can be found at:
- England
  - [https://census.ukdataservice.ac.uk/use-data/guides/flow-data.aspx](https://census.ukdataservice.ac.uk/use-data/guides/flow-data.aspx)
  - [https://census.ukdataservice.ac.uk/use-data/guides/boundary-data](https://census.ukdataservice.ac.uk/use-data/guides/boundary-data)
- Italy
  - [http://datiopen.istat.it/datasetPND.php](http://datiopen.istat.it/datasetPND.php)
  - [https://www.istat.it/it/archivio/104317#accordions](https://www.istat.it/it/archivio/104317#accordions)
- New York
  - [https://github.com/GeoDS/COVID19USFlows](https://github.com/GeoDS/COVID19USFlows)
  - [https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2020&layergroup=Census+Tracts](https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2020&layergroup=Census+Tracts)

Data related to POIs should be retrieved from appropriate services. Examples are Overpass API, HOTosm or - suggested - by downloading a local copy of the OSM database in a PostgreSQL instance and by running appropriate queries. The query we used to retrieved POIs information is available in `osm_query.yaml`
