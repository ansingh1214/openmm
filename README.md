
## OpenMM (Nonconservative CustomNonbonded Forces)

Introduction
------------
This is a modified version of OpenMM that allows implementation of **Nonconservative Custom Nonbonded Forces** for **CUDA** and **OpenCL**. In other words, a force expression can be passed into CustomNonbondedForce (instead of an energy expression). The following setters and getters have been added:
```C++
boolean CustomNonbondedForce.getIsNC() // returns if the force is nonconservative
void CustomNonbondedForce.setNC()      // set the Force to nonconservative
```
To use this, have a look at the following example:

```python
from openmm import *
import numpy as np

N      = 2
system = System()
a    = np.array([5,0.0,0.0])
b    = np.array([0.0,5,0.0])
c    = np.array([0.0,0.0,5])
system.setDefaultPeriodicBoxVectors(a,b,c)
for i in range(N):
    system.addParticle(1)

energy_expression = 'r'
force             = CustomNonbondedForce(energy_expression)
for i in range(N):
    force.addParticle([])
force.setNonbondedMethod(CustomNonbondedForce.CutoffPeriodic)
force.setCutoffDistance(2) 
force.setNC()
force.setForceGroup(1)
print(force.getIsNC()) # returns True
system.addForce(force)

platform   = openmm.Platform.getPlatformByName("OpenCL")
integrator = VerletIntegrator(0.01)
context    = Context(system,integrator,platform)
pos        = np.array([[1,0,0],
                       [0,0,0]])
context.setPositions(pos)
integrator.step(1)
state  = context.getState(getForces=True)
print(state.getForces()._value) 

```
The output of the force is
```
[Vec3(x=1.0, y=1.0, z=1.0), Vec3(x=1.0, y=1.0, z=1.0)]
[Vec3(x=1.0, y=1.0, z=1.0), Vec3(x=1.0, y=1.0, z=1.0)]
```
which is simply equal to the distance between the atoms.

[OpenMM](http://openmm.org) is a toolkit for molecular simulation. It can be used either as a stand-alone application for running simulations, or as a library you call from your own code. It
provides a combination of extreme flexibility (through custom forces and integrators), openness, and high performance (especially on recent GPUs) that make it truly unique among simulation codes.  

