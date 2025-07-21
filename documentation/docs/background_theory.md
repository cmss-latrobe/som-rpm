# Background Theory

For a comprehensive description of the background theory underpinning SOM-RPM, please refer to the [paper associated
with the release of this toolbox](https://doi.org/10.1016/j.chemolab.2025.105383). For a condensed version, please see below.

The self-organizing map (SOM) creates a low-dimensional topology-preserving model of high-dimensional data, revealing the way
in which data points are arranged in high-dimensional space. This toolbox is an implementation of the SOM combined with a secondary
algorithm, called the relational perspective map (RPM), which together form self-organizing map with relational perspective mapping (SOM-RPM).

## SOM

Given a data set comprising $p$ features, the SOM learns the weights for a set of $N$ neurons, $\mathcal{N}=\{n_1,n_2,…,n_N\}$ (with the weights matrix given by 
$\textbf{W}∈\mathbb{R}^{N×p}$), to model data topology. The result is a 2D grid of neurons whose topological relationships model the high-dimensional data topology,
through a combination of the neuron positions in the grid and their weight vectors.

####Batch vs Online training 

The SOM can be fit using either batch of online training. Briefly, batch training updates neuron weights by considering all data points simultaneously, whereas online
training involves passing each data point to the neurons sequentially. In general, batch training is much more efficient and, given that this toolbox is intended for
hyperspectral data with many samples (pixels), this is the default and highly recommended training mode.

In either case, neuron weights are updated based on a distance function, $d(n_r,n_{BMU})$, which determines the distance between each neuron, $n_r$, and the best matching unit (BMU), $n_{BMU}$,
for a given data point. The BMU is the neuron whose weight vector is closest (typically based on Euclidean distance) to the data point vector. In our implementation, we use
the Euclidean distance between neurons on the grid as a distance measure, with toroidal topology. The toroidal topology ensures that edge effects are mitigated during training.
For more information about these fundamental aspects of the SOM, we strongly suggest referring to our [paper](https://doi.org/10.1016/j.chemolab.2025.105383) and the broader SOM literature.


#### Learning rate decay function

When using online training mode, the toolbox allows specification of a learning rate decay (batch training mode does not use a learning rate).

The two options for this decay provided are cosine (default) and linear decay. For the cosine decay, the learning rate at epoch t is given by EQ1 and linear decay it is given by EQ2

$\eta(t) = \frac{1}{2} \cdot (\eta_{max} - \eta_{min}) \cdot (1 + \cos (\frac{t}{t_{max}}\pi))+ \eta_{min}$<span style="float:right;">EQ1</span>

$\eta(t) = (\eta_{max} - \eta_{min}) \ast (1- \frac{t}{t_{max}}) + \eta_{min}$<span style="float:right;">EQ2</span>

where $\eta_{max}$ and $\eta_{min}$ are the maximum and minimum learning rates, respectively, and $t$ and $t_{max}$ are the current and 
maximum epochs, respectively.

#### Neuron neighborhood function

In order for the SOM to self-organize, it is necessary to use a neighborhood function in addition to the distance
function described earlier. This function is responsible for determining how neurons close to the BMU are updated at each epoch.

Two common functions are Gaussian and triangle, which are the two options currently provided in the toolbox. 

The Gaussian (EQ3) and the triangle (EQ4) neighborhood functions are given by

$u(n_r,n_{BMU},t) = \exp(-\frac{d(n_r,n_{BMU})^2}{2\sigma(t)^2})$		<span style="float:right;">EQ3</span>

$u(n_r,n_{BMU},t) = \mbox{max} (0,1 - \frac{d(n_r,n_{BMU})}{\sigma(t)})$ <span style="float:right;">EQ4</span>

where $n_r$, $n_{BMU}$, $t$ and and $d(n_r,n_{BMU})$ are defined as before. The function $\sigma(t)$ is the 
neighborhood radius, which decays with respect to $t$ (both cosine and linear decay are options in the toolbox, as 
for the learning rate). This decay is a key component of the SOM algorithm, resulting in increased focus on 
neurons closer to the BMU as training progresses.

Intuitively, both neighborhood functions cause neurons that are close to the BMU to be adjusted more heavily than those that
are further away on the SOM. This controls how the SOM self-organizes into a topological model of the data.

## SOM Evaluation Metrics

The toolbox currently automatically calculates two commonly applied metrics to evaluate SOM fit quality: quantization error, $QE$, and 
topographic error, $TE$. 

The hyperparameters of the SOM can be adjusted to favour either $QE$ or $TE$, by trading off fine-grained fitting 
(low $QE$) with better topology preservation (low $TE$).

### Quantisation Error

$QE$ measures how close each data point is to its BMU, with smaller values indicating a better fit. Mathematically, it is given by 

$QE = \frac{1}{m} \sum^m_{i=1} \big\| \textbf{w}_{BMU} - \textbf{x}_i \big\|$  <span style="float:right;">EQ5</span>

where $\textbf{w}_{BMU}$ is the weight vector associated with the BMU for the i-th data point, $\textbf{x}_i$. 
Note that the value of $QE$ cannot be used as an absolute indicator of quality, as it depends on the data set.

### Topographic Error

$TE$ is a measure of the local topology preservation of the SOM. Mathematically, it is given by 

$TE = \frac{1}{m} \sum^m_{i=1} \tau(\textbf{x}_i)$ <span style="float:right;">EQ6</span>

$\tau (\textbf{x}) = \biggl\{^{0 \mbox{ if } n_{BMU} \mbox{ and } n_{SBMU} \mbox{ are neighbours on the SOM}}_{1 \mbox{ else}}$ <span style="float:right;">EQ7</span>


where $n_{BMU}$ and $n_{SBMU}$ are the best-matching and second best-matching units (neurons), respectively. 

Intuitively, $TE$ applies an error if the two closest neurons to $\textbf{x}_i$ (based on Euclidean distance) are not neighbors, indicating a lack of local topology preservation.

## RPM

The RPM algorithm is a non-linear dimensionality reduction (NLDR) algorithm designed to model high-dimensional distance information in a low-dimensional space.
To do this, it takes as input a set of abstract data points $\textbf{x}_i,i=1,2,…n,$ with a distance matrix $D^*$ with corresponding distance elements $d_{ij}^*,i,j=1,2,…,n$.
These points are mapped by the RPM algorithm to a 2D map space $t_i,i=1,2,…n,$ with a new distance matrix $D$ 
with elements $d_{i,j},i,j=1,2,…,n$. RPM aims to preserve distance information by identifying a configuration 
such that $D$ (in the 2D map space) is similar to $D^*$.
 
The RPM algorithm models the data points as a multiparticle system on a continuous toroidal surface. Each pair of particles $(i, j)$ 
experiences a symmetric repulsive force, determined by both $d_{i,j}$ and $d_{i,j}^*$.

Specifically, the pairwise particle force is given by 

$f_{i,j} = \frac{\delta E_p}{\delta d_{ij}} = -\frac{d_{ij}^*}{d^{p+1}_{ij}}$ <span style="float:right;">EQ8</span>

where $E_p$ is the potential energy of the system and is defined by

$E_p = \Biggl\{^{-\sum_{i<j}d^*_{ij}\ln(d^*_{ij}), \mbox{ if }p=0}_{\sum_{i<j}\frac{d^*_{ij}}{pd^*_{ij}}, \mbox{ else } }$ <span style="float:right;">EQ9</span>

The rigidity, $ρ$, is set by the user and determines whether the mapping is biased toward local 
(larger values) or global (smaller values) distances. 

For our implementation of SOM-RPM for hyperspectral images, we apply the RPM algorithm to the neurons of a trained SOM. In this way, the topology of the neurons determines
the starting coordinates of the low-dimensional RPM map, and their weight vectors are used for force calculations. Intuitively, the addition of RPM to the trained SOM incorporates
distance information into the model to complement the topological information. Finally, a CIELAB color scheme is used to show the similarity between hyperspectral pixels 
(based on the SOM-RPM model) using color.

The descriptions above are very brief and intended only to give an overview of the theory underpinning SOM-RPM. We again emphasize that users should read our original
[SOM-RPM paper](https://doi.org/10.1021/acs.analchem.0c00986), and the [paper associated with the release of this toolbox](https://doi.org/10.1016/j.chemolab.2025.105383).
