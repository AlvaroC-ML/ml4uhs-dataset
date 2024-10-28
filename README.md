# UHS Dataset

This repositry stores a dataset intended for training machine learning models on the task of predicting underground hydrogen storage. It is the dataset used in the paper [OUR PAPER].

## Brief description of data

The dataset includes 60,000 training pairs. In particular it is divided among 1000 simulations each consisting of 10 years of simulation data. The ten years of simulation is split into 2 month fragments, resulting in 60 time steps per simulation. Each simulation is uniquely defined by a porosity and permeability map. Each time step consists of a pressure (bar) and hydrogen saturation (/) map. Every map has 128x128 resolution.

## Accessing data

The data is stored in LMDB format. The keys to the data have the following format. The following script gives an example of how to access it.
```
import pickle
import lmdb
import torch
import matplotlib.pyplot as plt

to_p = lambda obj: pickle.dumps(obj) # Pickles
from_p = lambda obj: pickle.loads(obj) # Depickles

sim_number = "0" # Simulations go from 0 to 999
time = "60" # Time steps go from 0 to 60

env = lmdb.open("dataset") # Assuming the dataset is stored in the working directory
print(env.stat()["entries"]) # Prints 62000
with env.begin() as txn:
    sim_constants = from_p(txn.get(to_p(sim_number)))
    sim_vars = from_p(txn.get(to_p(sim_number+"-"+time)))
    
porosity = sim_constants[0]
permeability = sim_constants[1]
pressure = sim_vars[0]
hgas = sim_vars[1]

plt.imshow(hgas, vmin=0, vmax=1)
plt.colorbar()
plt.axis('off')
plt.show()
```

**Summary**
* All keys are strings.
* To access simulation constants (porosity and permeability), use the simulation number as the key (e.g. "0").
* To access a simulation variable (pressure and hydrogen saturation) at a given time, use the simulation number followed by that time with a hyphen in the middle (e.g. "0-60").
* Simulation numbers range timefrom 0 to 999.
* Simulation times go from 0 to 60. Time 0 symbolizes the step before any injection has taken place. Thus, the fields are empty. This time step is the same for every simulation, so effectively there are only  61,000 functional entries in the dataset.
