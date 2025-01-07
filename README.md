# FMRIPREP-AWS
How to create up-to-date FMRIPREP pipeline on AWS

# Prologue
Thanks to Mingwen Dong and Mengxue Kang for their 2019 AWS Open Blog post on running FMRIPREP on AWS: [fMRI data preprocessing on AWS using fMRIprep](https://aws.amazon.com/blogs/opensource/fmri-data-preprocessing-aws-fmriprep/).

This GitHub page extends their work by providing a guide on setting up an FMRIPREP pipeline using the latest version on AWS.

# Guide



# Epilogue
1. This is essential a guide on how to build an Amazon Machine Image with FMRIPREP preloaded. The steps outlined on this GitHub page can also be generalized to other Docker packages, such as XCP-D (for resting-state denoising), QSIPREP/QSIRECON (for structural preprocessing and reconstruction), and fmripost-aroma (for ICA-AROMA denoising if using FMRIPREP version 23.1.0 or later). Also, it is possible to create a single Amazon Machine Image with multiple packages loaded.
2. I am neither a computer scientist nor an AWS expert. However, I once found myself struggling to figure out how to run analyze fMRI data for my research without access to high-performance computing. My hope in writing this guide is that it might help someone who is facing a similar challenge.
