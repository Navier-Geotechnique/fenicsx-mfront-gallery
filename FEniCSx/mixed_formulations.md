# Mixed formulations for the Poisson equation

## Why could we need mixed formulations?

Previously, we have solved the classical Poisson equation, using a _primal formulation_.
If this approach produces good results in most cases, it has a major drawback.
Indeed, when using for instance 1st order Lagrange elements for the pressure $p$,
the gradient $\nabla p$ is piecewise constant per element.
In particular, the normal flux is not conserved at the interface between two elements.
In the case of a mass flux, this means that the local mass balance is not necessarily satisfied.
Mass balanced is only ensured in an averaged sense.
When conductivity is low, this can lead to spurious oscillations in the solution, as we shall demonstrate hereafter.

Introducing a so-called _mixed formulation_ (with pressure and flux both unknown) can solve this issue by using vector elements for the
flux which preserve the normal component of the flux between elements, ensuring local mass balance.
