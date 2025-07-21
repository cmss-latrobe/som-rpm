# Usage Guidelines 

###Helpful tips and shortcuts

- Use the **Tab** key to auto fill optional settings and functions in the command window. 
- Always explore your data using multiple models to get a full appreciation for the 
complexity of your data sets.
- When selecting regions of interest, setting ```n_roi = 0``` allows you to use previously drawn regions with the same name, 
while changing other settings such as ```top_p``` or ```distance_threshold```. This allows for a quick exploration of other settings 
without having to manually re-draw each time.
- Using ```invert_selection``` will invert the selection. This is a useful tool for removing the 
background or if your data exists in two key regions.  

## Data pre-processing and requirements
To begin, the input data used to train the SOM-RPM model, $X$, must be hyperspectral with either two or three spatial dimensions,
i.e. either a 3D or 4D array (either single or double precision). This toolbox does not perform any data preprocessing, so this must be considered separately. In general,
there is no one-size-fits-all in this regard, and it must be performed in careful consideration of the data, preferably with domain expertise.

## SOM fitting guidelines

### What is the best SOM size and number of Epochs?

We suggest running a range of model sizes (using the ```nsize``` input argument) and epochs to explore your data. 
In general, there is no best size – this depends on the aims of the analysis and must be considered 
with reference to domain expertise. Adjusting the SOM size can provide a a hierarchical view of similarity, 
where smaller maps show what is broadly similar and larger maps indicate more subtle changes. 

The outcome of the SOM is dependent on many factors such as data size and quality, noise levels, number of expected components etc.
As a rule of thumb, for simple systems, models up to size 10 x 10 should be sufficient. For more complex systems, 20 x 20 is often large enough. 
Alternatively, consider multiplying the number of expected components/classes by 4 as a starting point. This allows ample room for the components 
to be separated on the SOM.

A suitable number of epochs can be chosen by monitoring for convergence of the [quantisation error](background_theory.md) 
metric, which should reduce (until plateau) with increasing epochs. Additionally, the [topographic error](background_theory.md) 
is a measure of the topological preservation of the model, whereby lower values indicate better preservation.

As a starting point we suggest testing a range of values between 1 and 2000. Most data sets we have tested converge 
within this range. For example, a typical range of epochs may look something like [10, 100, 200, 500, 1000, 2000].

Note that, because the SOM is designed to be trained for a set number of epochs, separate models will need to be fit from 
scratch using different epochs. This is different from many machine learning algorithms which can be stopped early 
upon convergence.

Note also that the number of epochs is linearly proportional to computation time i.e. Epochs has time complexity of *O*(*n*), 
while SOM size input has time complexity of *O*(*n*<sup>2</sup>).  

###Neuron topology

The neuron topology alters the number of neighbours each neuron has. 
Our toolbox offers the option between square and hexagonal.
We have generally found hexagonal to be preferable as it has 6 neighbours instead of 4, and therefore tesselates
with even spacing across the SOM (although our recent [SOM hyperparameters paper](https://doi.org/10.1116/6.0002788)
did not show any major differences between the two).

### Training algorithm
The training algorithm can be set to either batch (default) or online. 
Batch mode is recommended as it considers the entirety of the data set simultaneously and is therefore much more computationally efficient,
which is particularly important for large hyperspectral data sets.
 
In contrast, online mode updates the neurons weights one sample at a time, and this is significantly slower. We have 
included this for completeness, but it is likely to be too slow to be practical for most hyperspectral data sets.
For a detailed and mathematical explanation read [here](background_theory.md). 

### Trade-off between quantization error and topographic error. 
The quantisation measures how close each data point is to its BMU while topographic error
measures the local topology of the SOM. Mathematical descriptions are provided [here](background_theory.md). 

In general this becomes a trade-off, in terms of model selection. That is as hyperparameters are adjusted 
(in particular the ```sigma_min``` argument), the two errors will tend to move in opposite directions. Selecting
an optimal model using these metrics alone is therefore impossible, and must be done based on domain expertise and
the specific aims of the analysis.

### Random versus PCA initialisation 

PCA initialisation initialises the weights of the SOM neurons based on the eigenvectors of the two largest principal components of the data. 
This will output the same SOM and RPM result in repeated calculations (when batch training mode is used). 
As the random initialisation is random, it results in a unique SOM being generated with repeated calculations. We suggest exploring
both options to identify which is best for your specific data set.  

###Adjusting sigma_min

One of the most influential optional arguments to consider when fitting a toroidal som is ```sigma_min```. 

As the SOM fitting progresses, the neighborhood radius (used to update neuron weights) decreases. ```sigma_min```
specifies the **minimum** radius. As we demonstrate briefly in [our paper](https://doi.org/10.1016/j.chemolab.2025.105383), changing this hyperparameter
controls the trade-off between quantization error and topographic error. Specifically, **increasing** ```sigma_min```
will create more topologically accurate SOMs by restricting the freedom of individual neurons to move more closely
to individual data points. Conversely, **decreasing** ```sigma_min``` will reduce the quantization error and therefore
better capture the fine-grained structure in the data. Again, the optimal value will depend on the data set and the
aims of the analysis.

## RPM fitting guidelines

### Flat and toroidal energy plots

These plots describe how the energy of the system converges during the RPM calculation, 
with the flat energy describing the initial flat approximation (if used) and the toroidal energy describing the 3D toroidal model. 

Both of these plots should converge smoothly. Using the flat approximation significantly reduces the computational time required for the RPM to run, 
with no significant drawbacks (that we have identified). Experimentaion is encouraged to see how each approach works with your data. 

#### How do I improve/reduce the energy?

In general, energy should converge smoothly. Sharp jumps indicate initial learning rate is too high and/or learning decay is too low. 
Conversely, setting the learning rate too low and/or decay too high will cause underfitting and lead to convergence at a suboptimal energy.
We therefore recommend experimenting with these hyperparameters to achieve a minimum energy with a smooth convergence.
Once a minium is identified, a final model should be re-fit using the optimal number of iterations.

### Rigidity
The rigidity hyperparameter changes the distance modelling to focus on either global (low values) or local (high values) distance information. 
While a ```SOMRPMModel``` can only contain a single SOM, it can contain infinitely many RPM models with different rigidities, 
whereby each RPM model with a unique rigidity is stored as a field within the ```RPMModels``` property.

As a rule-of-thumb, rigidities between -0.5 and 2 tend to show a good variation in terms of global versus local information. We recommend exploring
this range (and beyond) with each data set.

## Selecting regions of interest (ROIs) on the similarity map

### How to select areas 

The region select tool, by default, uses one drawn polygon to select pixels. This can be changed using the ```n_roi```and ```draw_shape``` 
optional inputs. Adjusting ```n_roi``` will change the number of regions you must draw to select pixels, while ```draw_shape``` can be changed to 
draw circular, rectangular or freehand regions. When using polygon selection, click once to begin the pixel selection, add vertices with 
additional clicks, then double click to close the shape. When using other selection shapes, click and hold/drag to select the area.

When ```n_roi``` is set to zero, this will prompt to search for previously created ROIs with the same name as inputted. This is useful when modifying optional
arguments, i.e. so that you do not need to redraw the selection each time.

### Selection modes
The selection mode, ```roi_type```, is set to ```similar``` by default, this mode will select all the pixels within your drawn ROI as well as 'similar' pixels, 
where similar is defined as those pixels sharing a best matching unit (BMU) with those pixels within the selection. This allows for all pixels associated with 
selected BMUs to be collected within one region. The ```roi_type``` can be changed to ```selection``` mode, which will only include pixels in the drawn ROI region 
without regard for RPM colouring. 

The ```dist_threshold``` (default = 0) can be increased to select pixels with BMU's close to the BMUs of pixels within the selection, 
whereby close is given by Euclidian distance in the RPM model. This mode is best used when the ```n_roi``` is set to zero, allowing for small changes without re-drawing regions

### top_p 
Using ```top_p``` allows for the exclusion of low-abundance pixels. For example if there is one red pixel in an area of 100 blue pixels, the red BMUs and associated 
pixels will be included in the selection. Using ```top_p``` selects the most abundant neurons from the drawn region until the cumulative fraction of pixels 
in the region reaches ```top_p```. We suggest setting ```top_p``` to 0.98, but again advise that this will depend on your specific data set and application. 



   

