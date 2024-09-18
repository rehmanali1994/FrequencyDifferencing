# FrequencyDifferencing
Frequency differencing method that generates low-frequency data for kickstarting full-waveform inversion

Ultrasound tomography (UST) is a medical imaging system that uses the transmission of ultrasound through tissue to create images of the speed of sound.  Currently, the main algorithm used to provide high-resolution images of the speed of sound is full-waveform inversion (FWI).  This work builds on our previous work on frequency-domain FWI for ring-array UST ([rehmanali1994/WaveformInversionUST](https://github.com/rehmanali1994/WaveformInversionUST)).  

The main challenge that this work aims to address is the problem of false-local minima via cycle skipping. When the initial sound speed guess is far from the ground truth, simulated ultrasound signals may be shifted by more than a half-cycle away from the measured waveform, causing the simulation to lock onto the wrong cycle of the measured waveform.  This causes FWI to converge to an erroneous sound speed image with several cycle-skipping artifacts.  One way to avoid cycle skipping is to use low-frequency data.  By lengthening the cycle, the same time shift would fall inside the desired half-cycle window.  However, most ultrasound transducer lack the adequate low-frequency response.  

Instead, we borrow a trick from sonar known as frequency differencing, which was originally developed in a beamforming context (frequency-difference beamforming).  We use frequency differencing to synthesize the desired low-frequency signals from the available high-frequency data.  The low-frequency data synthesized via frequency differencing is then used to kickstart FWI.

We provide sample data and algorithms presented in

> Rehman Ali; Trevor Mitcham; Sarah McConnell; Redi Rahmani; G. Edward Vates; Nebojsa Duric, "A Frequency-Differencing Strategy to Kickstart Full-Waveform Inversion Without Cycle Skipping", JASA Express Letters, _In Review_

If you use the code/algorithm for research, please cite the above paper and the prior work it was built on ([rehmanali1994/WaveformInversionUST](https://github.com/rehmanali1994/WaveformInversionUST)). 

You can also reference a static version of this code by its DOI number:
[![DOI](https://zenodo.org/badge/859300408.svg)](https://zenodo.org/doi/10.5281/zenodo.13785187)

# Experimental Datasets

**Please download the sample data (VSX256_YezitronixPhantom22.mat; VSX256_YezitronixPhantom44.mat; VSX256_MacaqueBrain_HF_1_24_24_Slice02.mat; VSX256_MacaqueBrain_HF_1_24_24_Slice03.mat; VSX512_AlcoholBrain_6_14_24_Axial.mat; VSX512_AlcoholBrain_6_14_24_Sagittal.mat) under the [releases](https://github.com/rehmanali1994/FrequencyDifferencing/releases) tab for this repository, and place that data in the [SampleData](https://github.com/rehmanali1994/FrequencyDifferencing/tree/main/SampleData/) folder.**

The following scripts correspond to each dataset:
1) _VSX256_YezitronixPhantom22.mat_ and _VSX256_YezitronixPhantom44.mat_ - ([MultiFrequencyFreqDiffWaveformInvVSX256Yezitronix.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX256Yezitronix.m)) - These datasets correspond to Verasonics acquisitions from two different slices of a breast phantom. In the full paper, we show results for (1024 emitters) x (1024 receivers), but to conserve space, we downsampled this data to (256 emitters) x (256 receivers).
2) _VSX512_AlcoholBrain_6_14_24_Axial.mat_ and _VSX512_AlcoholBrain_6_14_24_Sagittal.mat_ - ([MultiFrequencyFreqDiffWaveformInvVSX512AlcoholBrain.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX512AlcoholBrain.m)) - These datasets correspond to Verasonics acquisitions from the axial and sagittal slices of a cadaveric human brain sample preserved in alcohol. In the full paper, we show results for (1024 emitters) x (1024 receivers), but to conserve space, we downsampled this data to (512 emitters) x (512 receivers).
3) _VSX256_MacaqueBrain_HF_1_24_24_Slice02.mat_ and _VSX256_MacaqueBrain_HF_1_24_24_Slice03.mat_ - ([MultiFrequencyFreqDiffWaveformInvVSX256MacaqueBrain.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX256MacaqueBrain.m)) - These datasets correspond to Verasonics acquisitions from two different slices of a macaque brain inside a plastic bag placed within a replica human skull. In the full paper, we show results for (1024 emitters) x (1024 receivers), but to conserve space, we downsampled this data to (256 emitters) x (256 receivers).

# k-Wave Simulation

The code used to run the [k-Wave](http://www.k-wave.org/) simulation involves a 3-step process:

1) [GenKWaveSimInfo.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Simulations/GenKWaveSimInfo.m) creates a MAT file that is stored in the [Simulations/sim_info](https://github.com/rehmanali1994/FrequencyDifferencing/tree/main/Simulations/sim_info) folder. This MAT file contains all the information (medium, ring array geometry, and pulse excitation) needed to simulate the UST system. [GenKWaveSimInfo.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Simulations/GenKWaveSimInfo.m) uses [sampled_circle.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Simulations/phantoms/sampled_circle.m) to create the ring-array transducer and [soundSpeedPhantom2D.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Simulations/phantoms/soundSpeedPhantom2D.m) to produce the material properties used in the simulation. 
2) After generating the MAT file in [Simulations/sim_info](https://github.com/rehmanali1994/FrequencyDifferencing/tree/main/Simulations/sim_info), we run the actual [k-Wave](http://www.k-wave.org/) simulation using [GenRFDataSingleTxKWave.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Simulations/GenRFDataSingleTxKWave.m). [GenRFDataSingleTxKWave.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Simulations/GenRFDataSingleTxKWave.m) loops through each single-element transmit. The simulated data for each transmit is then stored in MAT files in the [Simulations/scratch](https://github.com/rehmanali1994/FrequencyDifferencing/tree/main/Simulations/scratch) folder.
3) Lastly, [AssembleUSCTChannelData.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Simulations/AssembleUSCTChannelData.m) assembles the simulated data from each indivdual transmit/MAT-file in the [Simulations/scratch](https://github.com/rehmanali1994/FrequencyDifferencing/tree/main/Simulations/scratch) folder into a single MAT file _kWave_SheppLogan.mat_ containing the full UST dataset in the [Simulations/datasets](https://github.com/rehmanali1994/FrequencyDifferencing/tree/main/Simulations/datasets) folder.

# Code

The key functions/classes used in the waveform inversion scripts ([MultiFrequencyFreqDiffWaveformInvkWave.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvkWave.m); [MultiFrequencyFreqDiffWaveformInvVSX256Yezitronix.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX256Yezitronix.m); [MultiFrequencyFreqDiffWaveformInvVSX512AlcoholBrain.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX512AlcoholBrain.m)); [MultiFrequencyFreqDiffWaveformInvVSX256MacaqueBrain.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX256MacaqueBrain.m)) are: 
1) [HelmholtzSolver.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/HelmholtzSolver.m) - Implements the Helmholtz equation solver as a class. For a given set of medium properties, the HelmholtzSolver forms the discretized system of equations that needs to be solved either on CPU or GPU. If an NVIDIA GPU is available, a block LU factorization is performed and stored in memory for subsequent solves using this factorization.
2) [stencilOptParams.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/stencilOptParams.m) - Helper function called by [HelmholtzSolver.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/HelmholtzSolver.m) to generate the stencil used to discretize the Helmholtz equation.
3) [decompBlockLU.cu](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/decompBlockLU.cu) - This is the MEX CUDA code called by [HelmholtzSolver.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/HelmholtzSolver.m) to perform the block LU factorization. Must be compiled in MATLAB using `mexcuda -lcusolver decompBlockLU.cu` using the cuSOLVER option.
4) [applyBlockLU.cu](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/applyBlockLU.cu) - This is the MEX CUDA code called by [HelmholtzSolver.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/HelmholtzSolver.m) to apply the block LU factorization computed by [decompBlockLU.cu](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/decompBlockLU.cu) to set of given sources (or adjoint sources). Must be compiled in MATLAB using `mexcuda -lcublas applyBlockLU.cu` using the cuBLAS option.
5) [ringingRemovalFilt.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Functions/ringingRemovalFilt.m) - Helper function called during waveform inversion scripts ([MultiFrequencyFreqDiffWaveformInvkWave.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvkWave.m); [MultiFrequencyFreqDiffWaveformInvVSX256Yezitronix.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX256Yezitronix.m); [MultiFrequencyFreqDiffWaveformInvVSX512AlcoholBrain.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX512AlcoholBrain.m)); [MultiFrequencyFreqDiffWaveformInvVSX256MacaqueBrain.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX256MacaqueBrain.m)) to remove ringing artifacts in the images.

Note that the above codes are identical to our previous work ([rehmanali1994/WaveformInversionUST](https://github.com/rehmanali1994/WaveformInversionUST)). Please also cite the works listed in the previous repository if your work involves the codes above. These codes ran successfully with an NVIDIA GeForce RTX 3060 GPU (12 GB of GPU RAM) on a CPU with 48 GB of RAM in MATLAB versions 2021b, 2022b, and 2024a. We therefore recommend running this code on a CPU with at least 48 GB of RAM and a GPU with at least 12 GB of RAM.

# Sample Results
Each waveform inversion script ([MultiFrequencyFreqDiffWaveformInvkWave.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvkWave.m); [MultiFrequencyFreqDiffWaveformInvVSX256Yezitronix.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX256Yezitronix.m); [MultiFrequencyFreqDiffWaveformInvVSX512AlcoholBrain.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX512AlcoholBrain.m)); [MultiFrequencyFreqDiffWaveformInvVSX256MacaqueBrain.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/MultiFrequencyFreqDiffWaveformInvVSX256MacaqueBrain.m)) saves the results at each iteration to a MAT file in the [Results](https://github.com/rehmanali1994/FrequencyDifferencing/tree/main/Results) folder. The results stored in these MAT files can later be visualized using the [viewSavedResults.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/viewSavedResults.m)) script. 

1) _kWave_SheppLogan.mat_ (created by [AssembleUSCTChannelData.m](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Simulations/AssembleUSCTChannelData.m))

![](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Results/Figure1_kWave.png)

2) _VSX256_YezitronixPhantom22.mat_ and _VSX256_YezitronixPhantom44.mat_

![](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Results/Figure2_PhantomVSX.png)

4) _VSX512_AlcoholBrain_6_14_24_Axial.mat_ and _VSX512_AlcoholBrain_6_14_24_Sagittal.mat_ 

![](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Results/Figure3_AlcoholBrain.png)

4) _VSX256_MacaqueBrain_HF_1_24_24_Slice02.mat_ and _VSX256_MacaqueBrain_HF_1_24_24_Slice03.mat_ 

![](https://github.com/rehmanali1994/FrequencyDifferencing/blob/main/Results/Figure4_MacaqueBrain.png)
