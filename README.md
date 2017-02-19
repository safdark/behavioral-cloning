<!--- Adding links to various images used in this document --->
[Simulator-Straight]: https://github.com/safdark/behavioral-cloning/blob/master/docs/images/illustration1.png "Straight"
[Simulator-LeftTurn]: https://github.com/safdark/behavioral-cloning/blob/master/docs/images/illustration2.png "Left Turn"
[Simulator-SharpLeftTurn]: https://github.com/safdark/behavioral-cloning/blob/master/docs/images/illustration3.png "Sharp Left Turn"
[Simulator-SharpRightTurn]: https://github.com/safdark/behavioral-cloning/blob/master/docs/images/illustration4.png "Sharp Right Turn"

# Behavioral Cloning - Racetrack Driving

![Simulator-SharpRightTurn]

## Overview

This application provides utilities to train a model of a car to steer along a simulated race traack, and to then allow that trained model to drive the same car autonomously on the same simulated track.

## Design:

Note: This was one of my first few projects in Python. Please excuse any non-pythonic code.

I have tried to design the utility in a way that allowed me to:
* Support a plug-and-play sort of mechanism to easily build and test different kinds of model architectures (CNNS) and associated weights.
* Re-load existing trained models/weights if they already exist
* Write back to saved models (overwrite mode) or add new model/weights files otherwise.
* Read training batches from disk (so as to limit memory consumption). Batch size depends on how much is read from disk at a time.
	* Shuffle the training data and pre-process the images to resize, and randomly flip the images. Other pre-processing steps can easily be added into the pipeline. Shuffling is possible because the actual driving log does not contain images, and can therefore be held in memory in its entirety (at least for the needs of this course project).
	* Append/filter/augment data on top of the existing training data. This functionality doesn't presently exist, but can very easily be added in.
	
For playing with different model architectures, I used inheritance:
- basetrainer.py
	- custom1trainer.py
	- custom2trainer.py
	- ...more coming...
	
	BaseTrainer provides basic utilities like reading the model/weights from disk, and writing out to disk, and training.
	Custom1 and Custom2 (at the moment) are the only two model options. I wanted to be able to switch between models
	on the fly, so as to try out their performance without much bookkeeping overhead. Adding more models requires creating a
	new subclass of BaseTrainer and overriding one or two methods to build the model pipeline and choose the compile options.
	This was probably an overkill for this particular project, but should come in handy for later, I'm hoping.

Trainer: Model.py

This tool performs the training. It can be invoked as follows:
> python model.py 
	-n <model-folder>						: Name of the folder (relative to cwd) where the model files are to be read from and/or saved back to.
	-a <architecture> 						: Name of the model class to use, if one isn't found on disk ('custom1' or 'custom2').
	-t <training-folder>[<training-folder>]*: Name of the folder (or spare-separated list of folders) containing the training data.
	[-e <num_epochs>] 						: Number of epochs to train before evaluating the model against the test set.
	[-b <batch_size>] 						: Size of the batch to use when training
	[-o]									: Overwrite flag. True will cause the model/weights on disk to be overwritten. False will create new files.
	[-r]									: Dry run flag. True will not commit the model to disk after training. False will commit to disk.

Example:
	> cd behavioral-cloning/main
	> python model.py -n scratch/custom2 -a custom2 -t ../data/track1_forward_retain ../data/track1_forward_returns -e 3 -b 100


Server/Driver (drive.py):

This is the server that 'drives' the car in the simulator using the model trained from above. It can be invoked as follows:
> python drive.py
	-n <model-folder>						: Name of the folder (relative to cwd) containing the model files. If they don't exist, a new model 
												will be built using a glorot_uniform weight distribution and cannot be expected to perform.
	-a <architecture>						: Name of the model class to use, if one isn't found on disk ('custom1' or 'custom2').

Out-of-box example:
	> cd behaviora-cloning/main
	> python drive.py -n ../models/custom2 -a custom2
	
	The above command should launch a server that will wait for connections from the simulator.

## Limitations

### Training Data:
This is a work in progress. Due to the absence of a the requisite gaming control, I had to generate training data
using the keyboard, which causes several problems:
	- The driving can be jittery at times
	- Recovery from road sides back towards the road are occasionally sudden, excessive, or insufficient.
I did not have access to a gaming control (such as for PS3) to generate the training data, which is a crucial step
for training this model. Keyboard data has the disadvantage of having a majority of 0s, which can cause the model
to settle into a suboptimal local minima during training. This manifests itself as a constant valued steering angle
that gets returned to the simulator. Generating more recovery data (of the car returning to the center after veering
off to the sides) helps avoid, or get out of, these local minima. The importance of good data cannot be overstated!


