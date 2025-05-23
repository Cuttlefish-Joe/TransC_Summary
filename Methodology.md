## Methodology (for Trans-C Quadratic Code)
### Code Storage Protocols
All code used for this mini-project is stored locally at `C:\Users\thele\Dropbox\PC\Desktop\Honours\python\Trans_C_Quadratic`, and available online to reasonable accuracy at [github](https://github.com/Cuttlefish-Joe/trans-c-quadratics/tree/main). The base document is Trans_C_Quadratic, though Trans_C_Quadratic_0orderwork should be considered the default document, having been updated to include 0th order polynomial analysis. The variations present in the other files are alluded to via their names. 

### Standard Code Operation
The code operates in four key parts: Generating the Synthetic Data, Setting up for the Sampling, Running the Sampler, and Analysing the Code. Before this, several settings must be confirmed: 

> plotting = True provides diagnostic plotting outputs. 
> parallel = False determines whether parallel computation will be used for the sampling. Currently, this feature is not available on Windows systems. 
> autopseudo = True creates pseudo-priors automatically based on the input prior and likelihood data. Alternatively, the user can specify their own Gaussian functions. 
> autothin = False will thin within-state posterior samples by their auto-correlation, useful when input ensembles may not be independent - but comes at the cost of additional compute time. 

#### Generating the Synthetic Data
Here, the initial "true" polynomial is established, including true order, true parameter values, true noise level, and number of samples. 

#### Setting up for the Sampling
This section consists of the problem setup; establishing the prior, likelihood, and posterior functions; running an optimisation code to find starting parameter estimates; and defining the pseudo-prior. 

For the problem setup, the number of states and their dimensions must be defined. For these examples, 5 states were used with dimensions 1 - 5 (representing 0th to 4th order polynomials). Each of these states is assigned a $\sigma$, which correlates to the estimated amount of data noise present in each state. As the code is currently non-hierarchical, this stays a fixed estimate, and has a strong control over the ultimate state-acceptance ratios. Unless prior information is known, $\sigma$ seems to operate best when assigned equally across all states. Each state is also assigned a $\mu$, a list of initial estimates for the parameters in each state (i.e. length 1 for dimension 1, length 2 for dimension 2). Initially, these were selected at random, however the code was found to produce much more accurate final estimates when these initial random estimates were refined to be close to the values found post-optimisation. Parameter covariance matrices and data covariance matrices can then be constructed for each state, with this code assuming independent values and thus using diagonal matrices. The code is made more efficient by calculating the determinants of these matrices here also. Finally, a list of G matrices should be constructed, placing the parameters into the data context ($data = Gm$ , with m as the model parameters of a given state). This was constructed bespokely. 

The (log) prior function in this code assumes normalised normal distribution prior PDFs based on $\sigma$ and $\mu$ are desired - useful when solving for functions like polynomials, where it is not necessarily reasonable to provide upper and lower bounds for a flat uninformative prior. If a different distribution is desired, the problem setup may require the definition of other variables. Effectively, it is finding: 

\[ \text{log(prior)} =  -0.5\text{log}(2\pi) -0.5\text{log}(\sigma^2) - 0.5\frac{(x-\mu)^2}{\sigma^2} \]

The (log) likelihood function in this code assumes Gaussian data also. Unlike the prior, the likelihood function is based on the experimental data and places it in comparison to values calculated using the G matrices, following the same basic form as the prior function above. The (log) posterior function simply sums the log prior and log likelihood functions together. 

In order to estimate starting parameter values, a standard optimisation inversion code is run for each state being tested. The optimal parameters will occur for the largest posterior value, as this is the most commonly sampled state of the data. Thus, the `scipy.minimize` code is run targeting the negative of the posterior. The output values are used to refine the $\mu$ estimates as described above. 

The pseudo-prior can either be automatically generated (recommended) or input manually as a series of Gaussians. To automatically generate the pseudo-prior, a small number of walkers walk a small number of steps in a randomly chosen initial state around the optimised starting parameter values, forming PDFs around these expected values. Manually input Gaussian pseudo-priors require inputs of $\mu$ and $\sigma$, as well as covariance matrices for each state. 

#### Running the Sampler
As an "ensemble resampler" example of Trans-C analysis, running the sampler involves establishing and running initial samplers within each state, then running an ensemble resampler to allow comparison between acceptance of model states. 

The number of samples taken in each state should ideally be a lot higher than the expected amount of variation, giving each of the chains a long time to settle down to appropriate values after their burn-in periods (where the sampler is not yet close to an optimum point). For these examples, 32 chains with 50 000 samples each were ran in each of the five tested states. This step is the most time-intensive part of the code (taking between 10 and 30 minutes typically), though further optimisation may be found (at the cost of generalisation) by analytically inputting expressions for the covariance matrices and their determinants, given the simple nature of this problem. The optimised parameters for each state can be visualised after this stage with a corner plot, where the outside histograms represent the distribution of values for a single parameter, and the bullseye targets represent the correlation between each pair of parameters within the state. This function is not available for 1-dimensional states (the 0th order polynomial), though the properties of this distribution can be called using ensemble_per_state[0]. These optimised parameters per state are not changed during the resampling process. 

In this example, the ensemble resampler runs 32 chains with 100 000 samples each, to determine the acceptance ratios of each of the five states - with higher values representing greater support from the data. These chains can be checked to ensure they completed their burn in periods through visualisations of relative evidence in each chain over time. This stage of the Trans-C analysis determines the "conceptual model" best suited to explaining the data from the options provided. 

#### Analysing the Code
The primary axes of analysis for these Trans-C results are the acceptance rates between states, and the selected optimum parameters within the most-supported state. The relative evidences of each state can be directly compared to one-another numerically or via a column graph. For an example like this, a very high relative evidence is expected for the conceptual state that matches the original data (the quadratic model state). The efficacy of the McMC sampling can be confirmed through visualising the predicted curves from each of the runs in comparison to the truth curve, which should be near the centre of the sample curves. The optimised parameters in each state can be called through the ensemble_per_state or the transc_ensemble functions, for comparison with truth values and other inversion results. The ideal results would show a high acceptance rate for the correct conceptual model, with optimised parameters similar to the truth values.

### Code Iterations
For this mini-project, several different iterations of starting assumptions were made to test their effect on the code outputs, in the following way: 

| Iteration Name    | $\mu$ values                                                                                  | $\sigma$ values (linear to quartic) |
|-------------------|-----------------------------------------------------------------------------------------------|-------------------------------------|
| TransC Quadratic  | Optimised                                                                                     | All 10                              |
| badmus            | 0 except for 0th order term, which was 40                                                     | All 10                              |
| bigmus            | Optimised, however the initial parameters were an order of magnitude larger than other trials | All 10                              |
| badsigmas         | Optimised                                                                                     | 10, 3, 1, 1                         |
| badsigmas(1111)   | Optimised                                                                                     | 1, 1, 1, 1                          |
| badsigmas(4344)   | Optimised                                                                                     | 4, 3, 4, 4                          |
| badsigmas(131010) | Optimised                                                                                     | 1, 3, 10, 10                        |
| bigsigmas         | Optimised                                                                                     | 20, 20, 20, 20                      |
