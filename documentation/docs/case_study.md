# Case Study

This section contains a tutorial-style presentation of a case study, which provides a useful demonstration of typical usage of the
toolbox. We have included ToF-SIMS imaging data to get you started using this toolbox.

We have provided a ToF-SIMS data set to get you started [here](https://doi.org/10.26181/25648905.v1), with information about how the data was collected at the bottom of this page. 

Note that this data set is relatively large. If you would like to reduce its size, you can use the following command in MATLAB:

```MATLAB 
X = imresize(X, scale, ‘method’, ‘box’);
```

Where scale must be between 0-1 to reduce the data set size. Other interpolation methods are also possible – 
see [here](https://www.mathworks.com/help/matlab/ref/imresize.html) for more details.

## Creating a model
Data are required to be in the MATLAB workspace as a 3D (or 4D, if three spatial dimensions) hyperspectral array. Our data variable is called **X**. 
The first step in using the SOM-RPM toolbox is to instantiate the SOM-RPM model:
```MATLAB
 mdl = SOMRPMModel;
```
This uses the SOMRPMModel class to create the ```mdl``` object. Then, we need to fit a SOM model by calling the corresponding class method:

```MATLAB
 mdl = mdl.fit_toroidal_som(X, 6, 500);
```
Note that the ```mdl``` object is returned, which is now modified and contains the fitted SOM model. In this case we have chosen to make a 6 x 6 
neuron model with 500 epochs. This is quite a small model for our simple data set. 

We suggest running a range of model sizes and epochs before deciding the best model for your system. In general, there is no best size – this depends 
on the aims of the analysis and must be considered with reference to domain expertise. 

There are a number of additional settings that can be optionally added. For the full list, 
check the [Toolbox Structure](toolbox_structure.md) section. For usage suggestions check [Usage guidelines](usage_guidelines.md).  

If, for example you want your model to have square neurons instead of the default hexagonal, then you can use the below: 
 
```MATLAB
 mdl = mdl.fit_toroidal_som(X, 6, 500, 'topol','square');
```
Or, if you want to change ```sigma_min``` from the default 0.05 to 0.1 then you can use the below. 


```MATLAB
 mdl = mdl.fit_toroidal_som(X, 6, 500, 'sigma_min', 0.1);
```

The ```fit_toroidal_som``` function will output the alterd ```mdl``` object and a figure indicating the convergence of the quantisation and 
topographical error for your chosen model size and number of epochs. 

## Applying rpm algorithm

The second step in using the SOM-RPM toolbox is applying the RPM algorithm. 
This function has no required inputs, and be achieved using the following;

```MATLAB
mdl = mdl.fit_rpm;
```
Optional inputs can be added as above, for the full list, check the [Toolbox Structure](toolbox_structure.md) section. 
The rigidity is the parameter you will want to investigate most often, to change it from its default 0 to -0.5, see below. 


```MATLAB
mdl = mdl.fit_rpm('rigidity',-0.5);
```

```fit_rpm``` outputs several figures. The first figures are the Flat RPM (if set to true) and the toroidal RPM energy plots. 
Check the [Usage guidelines](usage_guidelines.md) for tips on when and how to optimise these values. The other figures are the similarity map, the SOM and the histogram SOM (indicating how many pixels are within each neuron),
 should you wish to access them at any other time, you can use the below commands, respectively;

```MATLAB
mdl.show_color_som;

mdl.show_histogram_som;

mdl.show_similarity_map;
```

## Selecting regions of interest (ROIs)

The next step in using the SOM-RPM algorithm is to investigate the data using the region selection function. 
This can also be used to remove background pixels from the data set. 
The only required input is the name of the region to be selected. 

Note: If you make multiple selections using the same name, each selection will override the previous one by default. 
So, take care when naming your regions as it can become confusing very quickly. 


```MATLAB
mdl = mdl.select_similarity_map_roi('roi_1');
```

This command will return the similarity map, and allow you to draw the regions you wish to select. If in default polygon mode, double clicking will close the shape. 
When the last region is drawn (1 by default), a new similarity map indicating only the selected pixels, the SOM with only the selected pixels present and the 
histogram SOM of the same area, will appear.   

Setting the number of ROIs to draw to zero (using n_roi ) allows you to use previously drawn regions with the same name, 
while changing other settings such as top_p or distance_threshold. 
This allows for a quick exploration of other settings without having to manually re-draw each time. 

```MATLAB  
mdl = mdl.select_similarity_map_roi('roi_1','n_roi', 0,'top_p', 0.98, 'dist_threshold', 0.15);
```

The ```show_similarity_map```, ```show_color_som``` and ```show_histogram_som``` methods can also be used with 
optional inputs to view ROI selections using roi_name.

```MATLAB
mdl.show_similarity_map('roi_name','roi_1');
```

###Retrieving ROI data 

Using the ```get_roi_info```, ```get_roi_pixel_data``` methods gives users the option to easily export information 
associated with the ROI selection.
The ```get_roi_info``` method creates a structure in the workspace that holds all the meta information on the selection, 
such as which settings were used and the mask area for the ROI. 
While, ```get_roi_pixel_data``` exports an unfolded array with dimensions
 (number of X pixels x number of Y pixels) x number of spectral positions/intensities and plots the average weights for 
a given ROI against the feature labels. These functions can be accessed using the following;

```MATLAB
roi_1_info = mdl.get_roi_info('roi_1');

roi_1_data = mdl.get_roi_pixel_data('roi_1');
```
Note: the generated plot uses a generic x-axis ranging to one to the number of spectral features. To plot with feature labels, 
an optional argument 'feature_labels' can be added which must be a numeric vector of same length as the number of columns in **X**.
In our example the array is called pk_lbls, representing the ToF-SIMS peak labels.
```MATLAB
roi_1_data = mdl.get_roi_pixel_data('roi_1', 'feature_labels', pk_lbls);
```
For exporting roi data, the following comman can be used;  

```MATLAB
 [pixel_weights, pixel_spectra] = mdl.get_roi_pixel_data('roi_1', 'X', X, 'feature_labels', feature_labels);
``` 
The rigidity can also be added as an optinal setting. 

This produces plots of the pixel speactra and weights and returns a matrix of size [no. of pixels within ROI, no. spectral features] 
###Removing the substrate and running a new SOM on the reduced data set

Assuming the roi_1 is the selected substrate, the mask input can be used to subtract the selected pixels from the data set.
First, make a new ```SOMRPMModel```, in this case we will create mdl_masked, then use the ``fit_torodial_som`` function as before, 
with the mask input set to the pixel index of roi_1_info. 

```MATLAB
mdl_masked = SOMRPMModel;

mdl_masked = mdl_masked.fit_torodial_som(X, 6, 500, 'mask', roi_1_info.pixel_ind);
```

To invert the selection, you will need to redo the roi selecion using the same settings, with the addition of ```invert_selection```, 
then repeat process. 
  
```MATLAB  
mdl = mdl.select_similarity_map_roi('roi_1','n_roi', 0,'top_p', 0.98, 'dist_threshold', 0.15, 'invert_selection', true);

roi_1_info = mdl.get_roi_info('roi_1');

mdl_masked = SOMRPMModel;

mdl_masked = mdl_masked.fit_toroidal_som(X, 6, 500, 'mask', roi_1_info.pixel_ind);
```
Save the workspace using the following 
```MATLAB
save('Data_name','-v7.3');
```

## About included data set
We have provided a ToF-SIMS hyperspectral data cube **X**. Its dimensions are 960x800x963, which represent the height, width 
and spectral dimension, respectively. 

ToF-SIMS data is acquired by measuring secondary ions emitted from the sample surface, based on their mass to charge ratio (m/z) inferred by their time-of-flight. 
The spectral dimension of **X** is therefore intensity value, reflecting the area under a mass peak (i.e. total number of ions collected) at a given m/z value. 
The m/z values are given in a vector called **pk_lbls** (short for peak labels).

To prepare the sample, an Inkjet printer (Hp Officejet Pro 8100) was used to print a file from PowerPoint, using the with the 
media set to ‘thick plain paper’ and the quality set to ‘best’. Pure black, yellow, cyan and magenta ink were 
printed as well as a mixed purple region at 100%, 50% and 10% opacity on 200g/m2 Colour Copy paper. The paper 
sample was then analysed using an IONTOF V ToF-SIMS instrument. Using a Bi3+ 30keV primary ion, in negative ion mode, 
the stage was rastered in random mode, over a 2000 x 2400 mm area collecting 800 x 950 pixels, with a total dose density 
of 1.20e+11 (1/cm2). This was collected at 20 frames per patch, with 1 shot per frame over 1 scan. Argon gas 
flooding was set to 1x10-6 mbar and the flood gun was used to for charge compensation. Ion peaks were 
automatically selected with minimum threshold of 100 counts, resulting in 963 peaks, that we then exported to MATLAB 
using the BiF6 file format, for analysis with the SOM-RPM toolbox.  