---
dg-publish: true
tags:
  - Process
  - MLOps
---
## Overview
In any organisation that makes use of ML there will be a need to implement some degree of MLOps. The degree to which this is done will depend on the project, the company and the available resources.
MLOps maturity can be increased in stages as the project matures and requirements increase. 
The following stages are meant as a guideline for teams who are in need of increased automation, so as to not ask them to implement full automation all at once. 

The stages are as follows:

1. NO MLOps - No metrics, No Version control, All of the Steps are done manually. 
2. DevOps but not MLOps -No metrics, No Version Control, CI/CD is implemented for the app but not for the model. 
3. Automated Training - Basic Metrics, VC, and CI/CD app, CI for the model. Model releases are manual. 
4. Automated  - A/B Metrics, VC, CI/CD for app and model.
5. Full MLOps Automated Operations - Pipe dream AKA Advanced Verbose Metrics, Zero Downtime, Automated Training and Testing, Automatic bug tracking. 

Let's take a closer look at each stage. 

### Level 0: NO MLOPS

#### People: 
The Data/ML teams are siloed away from the rest of the org. 
The software teams implementing the models are also siloed away from the data team. 

#### Model Creation:
Data is gathered manually. 
Compute is not managed.
Experiments are not tracked. 
The end result is a single model file.

#### Model Release: 
Manual process
Scoring/Validation script created after experiments, not part of version control
Process handled by one person

#### Application Integration
Manual release each time
Needs data person to assist with deployment

### Level 1: DevOps no MLOps

#### People: 
Same as Level 0.

#### Modal Creation:
Data is gathered automatically.
Rest is the same as Level 0.

#### Model Release:
Same as Level 0.

#### Application Integration
Basic integration tests exist for the model.
Releases automated.
The application has unit tests.
Still reliant on Data Person for releases. 

### Automated Training

#### People:
Data people convert experiments into repeatable code/scripts.
Data scientists working with Data engineers.
Software engineers are still siloed away from Data team.

#### Model Creation:
Data is gathered automatically.
Compute managed.
Experiment results tracked.
Both training code and resulting models are version controlled.

#### Model Release:
Manual release.
The scoring script is version controlled and tested.
The release is managed by the software team.

#### Application Integration
Basic integration tests exist for the model.
Heavily reliant on data person expertise.
Application code has unit tests.

### Level 3: Automated Model Deployment

#### People:
Data team and software team work together.

#### Model Creation:
Same as Level 3.

#### Model Release:
Automatic release.
Scoring scripts is tested and version controlled.
Model release is handled by CI/CD pipeline. 

#### Application Integration:
Unit and integration tests for each model release.
Less reliant on data team for implementation.
Application code has unit/integration tests. 

### Level 4: Full MLOps Automated Retraining

#### People:
Data team and software team work tightly together.
Data Scientists: Working with the software team to identify markers for data engineers.
Data Engineers: Working with data scientists and software engineers to manage inputs/outputs.
Software Engineers: Working with data engineers to automate model integration into application code. Implementing post-deployment metrics gathering. 

#### Model Creation:
Data is gathered automatically.
Retraining is triggered automatically based on production metrics.
Compute tracked.
Experiment results tracked.
Both training code and resulting models are version controlled.

#### Model Release:
Automatic Release.
The scoring script is version controlled and tested.
Release managed by CI/CD pipeline. 

#### Application Integration
Unit and integration tests for each model release.
Less reliant on data scientists expertise to implement the model.
Application code has unit/integration tests. 




