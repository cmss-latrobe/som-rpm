# Toolbox Structure

The SOM-RPM toolbox is built around a central ```class```, called ```SOMRPMModel```, which enables interactive analyses and data exploration using SOM-RPM. 

With this object-oriented design, almost of all of the functionality comes from direct interaction with instances of this ```class```, which are called ```objects```. For
example, we can instantiate an object called ```mdl``` from the ```SOMRPMModel``` class like this:

```MATLAB
mdl = SOMRPMModel;
```

The ```SOMRPMModel``` class has ```properties``` and ```methods```. The properties store all of the data associated with the model (e.g. the SOM model, RPM models, 
hyperparameters etc.), while the methods act on these properties in some way. 

For example, the ```fit_toroidal_som``` method fits a SOM to the inputted data, with outputs stored as a ```SOMModel``` property in the ```SOMRPMModel``` object. 
The ```fit_rpm``` method then uses the stored ```SOMModel``` to fit an RPM model, and so on.

Briefly, the workflow consists of the following steps (explained in more detail in the [case study](case_study.md)).

1. Instantiate a SOM-RPM model object 

2. Fit a toroidal SOM model using the ```fit_toroidal_som``` method

3. Fit an RPM model to the toroidal SOM using the ```fit_rpm``` method

4. Interact with the models using the following methods:

	a. ```show_similarity_map```

	b. ```show_color_som```

	c. ```show_histogram_som```

	d. ```select_similarity_map_roi```

	e. ```get_roi_info```

	f. ```get_roi_pixel_data``` 

## SOMRPMModel Methods

### fit_toroidal_som

This method fits a toroidal SOM to the data, assigning each pixel (or voxel) a neuron.
Below is a table of inputs, possible values and default values. Inputs denoted with an * are optional. 

|Inputs |Description | Possible values | Default |
|-------|------------|------------------|--------|
|X  |Hyperspectral data cube | 3D numeric array |NA |
|nsize |Number of neurons on each side of SOM | Integer scalar greater than zero |NA |
|epochs |Number of training epochs | Integer scalar greater than zero | NA |
|topol* |Neuron topology |‘square’ or ‘hexagonal’ |‘hexagonal’ |
|training* |Training algorithm |‘batch’ or ‘online’ |'batch' |
|use_gpu* |Indicates whether to use GPU for computations |Boolean (logical) |false |
|lr_max* |Maximum (starting) learning rate |Real scalar greater than zero |0.5 |
|lr_min* |Minimum (ending) learning rate |Real scalar greater than zero and less than or equal to lr_max |0.01 |
|lr_decay_func* |Learning rate decay function |'cosine' or 'linear'|'cosine' |
neighbor_func* | Neuron neighborhood function | 'gaussian' or 'triangle' | 'gaussian'|
sigma_max* | Maximum (starting) sigma for neighborhood function| Real scalar greater than zero and less than or equal to one|0.2
sigma_min* | Minimum (ending) sigma for neighborhood function| Real scalar greater than zero and less than or equal to sigma_max|0.05
sigma_decay_func* | Decay function for sigma |'cosine' or 'linear' |'cosine'|
|init* |Weights initialisation method |‘random’ or ‘pca’ |‘random’ |
|init_weights* |Initial weights for the SOM neurons |3D numeric array of size [nsize, nsize, size(X, 3)] |[] |
|shuffle_data* |Controls whether to shuffle data during training |Boolean (logical) |false |
|plot_metrics* |Plot quality metrics after fitting |Boolean (logical)|true |
|mask* |Mask for hyperspectral data cube pixels |Boolean (logical) matrix of size [size(X, 1), size(X, 2)] OR a Boolean (logical) vector of length size(X, 1)*size(X, 2) |false(0) |
|verbose*|Verbosity of print statement updates |Integer scalar either 0 (no updates), 1 (update every 10 epochs) or 2 (update every epoch) |2 |

The only output is the completed model mdl. The quantisation and topographic error are printed in the command window. Plots showing these metrics
as a function of epochs are also created automatically.
 
### fit_rpm

This method applies the RPM algorithm to the trained SOM Model, colouring the SOM. 
There are no required input arguments for fit_rpm, so the below table shows only the optional input arguments and the outputs. 
In this case, the input arguments control both the RPM fitting and the resulting visualizations.

|Inputs |Description |Possible values |Default |
|-------|--------|--------------|--------------|
|init_lr* |Initial learning rate for 3D toroidal RPM |Real scalar greater than zero |0.1 |
|vanishing_rate* |Vanishing rate for 3D toroidal RPM |Real scalar greater than zero |0.9999 |
|rigidity* |Rigidity for RPM force calculations |Real scalar in range [-1, ∞] |0 |
|metric* |Distance metric for high-dimensional distances |‘euclidean’, ‘mahalanobis’, ‘cityblock’, ‘cosine’, … |'euclidean' |
|n_iter* |Maximum number of iterations for 3D toroidal RPM |Integer greater than or equal to zero |10000 |
|n_rot* |Number of rotations around 3D torus to try |Integer greater than or equal to zero |20 |
|patience* |Stopping iteration patience for 3D toroidal RPM |Integer greater than or equal to zero |50 |
|flat_approx* |Use a flat RPM initially to approximate the best solution |Boolean (logical) |true |
|init_lr_flat* |Initial learning rate for flat RPM |Real scalar greater than or equal to zero |0.5 |
|vanishing_rate_flat* |Vanishing rate for flat RPM |Real scalar greater than or equal to zero |0.9999 |
|n_iter_flat* |Maximum number of iterations for flat RPM|Integer greater than or equal to zero |10000 |
|patience_flat* |Stopping iteration patience for flat RPM |Integer greater than or equal to zero |10 |

This again outputs the completed model mdl as well as 3-4 figures depending on your settings. 
In this case, the outputted ```SOMRPMModel``` will contain the trained RPM model. Where a ```SOMRPMModel``` object can only contain a single SOM, 
it can contain infinitely many RPM models with different rigidities.
 
### select_similarity_map_roi

This method is used to select regions of interest either for background removal or for investigation of hyperspectral data. 

|Inputs |Description |Possible values |Default |
|-------|--------|-----------|----------|
|roi_name |Name of the region of interest (ROI) |String|NA |
|rigidity*|Rigidity used for RPM model|Real scalar in range [-1, ∞]|0 |
|n_roi* |Number of ROIs to draw|Integer greater than or equal to zero |1| 
|roi_type* |Type of ROI, either similar or selection |'similar' or 'selection'|'similar'|
|draw_shape* |Shape of ROI to draw|‘Polygon’, ‘Circle’, ‘Freehand’, ‘Rectangle’, … |'Polygon'|
|top_p*|Cumulative pixel fraction to select|Real scalar in range [0, 1] |1|
|dist_threshold*|Distance threshold when using similar roi_type|Real scalar greater than or equal to zero|0| 
|invert_selection*|Flag to invert the drawn ROI mask|Boolean (logical) |false |
|color_scheme* |Color scheme to use for visualisation|'cielab' or 'rgb'|'cielab'|
|cmap*|Colormap to use for histogram SOM |‘parula’, ‘turbo’, ‘hsv’, ‘hot’, ‘cool’, ‘spring’, … or 2D numeric RGB array |'gray' (inverted) | 