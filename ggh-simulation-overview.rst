GGH Simulation Overview
========================

All GGH simulations include a list of *objects*, a description of their *interactions* and *dynamics* and appropriate *initial conditions*.

Objects in a GGH simulation are either generalized cells or *fields* in two dimensions (2D) or three dimensions (3D). Generalized cells are spatially-extended objects (Figure 1), which reside on a single *cell lattice* and may correspond to biological cells, sub-compartments of biological cells, or to portions of non-cellular materials, e.g. ECM, fluids, solids, etc. [8][48][60]-[72]. We denote a lattice site or pixel by a vector of integers, :math:`i` , the *cell index* of the generalized cell occupying pixel :math:`i` by :math:`\sigma(i)`  and the type of the generalized cell :math:`\sigma(i)` by :math:`\tau(\sigma(i))`. Each generalized cell has a unique cell index and contains many pixels. Many generalized cells may share the same cell type. Generalized cells permit coarsening or refinement of simulations, by increasing or decreasing the number of lattice sites per cell, grouping multiple cells into clusters or subdividing cells into variable numbers of *subcells* (subcellular compartments). Compartmental simulation permits detailed representation of phenomena like cell shape and polarity, force transduction, intracellular membranes and organelles and cell-shape changes. For details on the use of subcells, which we do not discuss in this chapter see [27][31][73][74]. Each generalized cell has an associated list of attributes, e.g., cell type, surface area and volume, as well as more complex attributes describing a cell’s state, biochemical interaction networks, etc.. Fields are continuously-variable concentrations, each of which resides on its own lattice. Fields can represent chemical diffusants, non-diffusing ECM, etc.. Multiple fields can be combined to represent materials with textures, e.g., fibers.

*Interaction descriptions* and *dynamics* define how GGH objects behave both biologically and physically. Generalized-cell behaviors and interactions are embodied primarily in the effective energy, which determines a generalized cell’s shape, motility, adhesion and response to extracellular signals. The effective energy mixes true energies, such as cell-cell adhesion with terms that mimic energies, e.g., the response of a cell to a chemotactic gradient of a field [75]. Adding constraints to the effective energy allows description of many other cell properties, including osmotic pressure, membrane area, etc. [76]-[83].

The cell lattice evolves through attempts by generalized cells to move their boundaries in a caricature of cytoskeletally-driven cell motility. These movements, called index-copy attempts, change the effective energy, and we accept or reject each attempt with a probability that depends on the resulting change of the effective energy, :math:`\Delta H`, according to an acceptance function. Nonequilibrium statistical physics then shows that the cell lattice evolves to locally minimize the total effective energy. The classical GGH implements a modified version of a classical stochastic Monte-Carlo pattern-evolution dynamics, called Metropolis dynamics with Boltzmann acceptance [84][85]. A Monte Carlo Step (MCS) consists of one index-copy attempt for each pixel in the cell lattice.

Auxiliary equations describe cells’ absorption and secretion of chemical diffusants and extracellular materials (i.e., their interactions with fields), state changes within cells, mitosis, and cell death. These auxiliary equations can be complex, e.g., detailed RK descriptions of complex regulatory pathways. Usually, state changes affect generalized-cell behaviors by changing parameters in the terms in the effective energy (e.g., cell target volume or type or the surface density of particular cell-adhesion molecules).

Fields also evolve due to secretion, absorption, diffusion, reaction and decay according to partial differential equations (PDEs). While complex coupled-PDE models are possible, most simulations require only secretion, absorption, diffusion and decay, with all reactions described by ODEs running inside individual generalized cells. The movement of cells and variations in local diffusion constants (or diffusion tensors in anisotropic ECM) mean that diffusion occurs in an environment with moving boundary conditions and often with advection. These constraints rule out most sophisticated PDE solvers and have led to a general use of simple forward-Euler methods, which can tolerate them.
The initial condition specifies the initial configurations of the cell lattice, fields, a list of cells and their internal states related to auxiliary equations and any other information required to completely describe the simulation.

Effective Energy
------------------
The core of GGH simulations is the effective energy, which describes cell behaviors and interactions.

One of the most important effective-energy terms describes cell adhesion. If cells did not stick to each other and to extracellular materials, complex life would not exist [86]. Adhesion provides a mechanism for building complex structures, as well as for holding them together once they have formed. The many families of adhesion molecules (CAMs, cadherins, etc.) allow embryos to control the relative adhesivities of their various cell types to each other and to the noncellular ECM surrounding them, and thus to define complex architectures in terms of the cell configurations which minimize the adhesion energy.

To represent variations in energy due to adhesion between cells of different types, we define a *boundary energy* that depends on :math:`J(\tau(\sigma),\tau(\sigma'))` , the *boundary energy* per unit area between two cells :math:`(\sigma, \sigma')` of given types (:math:`\tau(\sigma), \tau(\sigma')`) at a link (the interface between two neighboring pixels):

.. math:: H_{boundary} = \sum_{i,j} J(\tau(\sigma(i)), \tau(\sigma(j)))) (1-\delta(\sigma(i), \sigma(j)))
  :label: h_boundary

where the sum is over all neighboring pairs of lattice sites   and   (note that the neighbor range may be greater than one), and the boundary-energy coefficients are symmetric,

.. math:: J(\tau(\sigma),\tau(\sigma')) = J(\tau(\sigma'), \tau(\sigma))
  :label: j

In addition to boundary energy, most simulations include multiple constraints on cell behavior. The use of constraints to describe behaviors comes from the physics of classical mechanics. In the GGH context we write constraint energies in a general elastic form:

.. math:: H_{constraint} = \lambda(value - target\_value)^2
   :label: h_constraint

The constraint energy is zero if :math:`value = target\_value` (the constraint is satisfied) and grows as value diverges from :math:`target\_value`. The constraint is *elastic* because the exponent of 2 effectively creates an ideal spring pushing on the cells and driving them to satisfy the constraint. :math:`\lambda` is the *spring constant* (a positive real number), which determines the *constraint strength*. Smaller values of :math:`\lambda` allow the pattern to deviate more from the *equilibrium condition* (i.e., the condition satisfying the constraint). Because the constraint energy decreases smoothly to a minimum when the constraint is satisfied, the energy-minimizing dynamics used in the GGH automatically drives any configuration towards one that satisfies the constraint. However, because of the stochastic simulation method, the cell lattice need not satisfy the constraint exactly at any given time, resulting in random fluctuations. In addition, multiple constraints may conflict, leading to configurations which only partially satisfy some constraints.

Because biological cells have a given volume at any time, most GGH simulations employ a *volume constraint*, which restricts volume variations of generalized cells from their target volumes:

.. math:: H_{vol} = \sum_{\sigma} \lambda_{vol}(\sigma) (v(\sigma)-V_t(\sigma))^2
   :label: h_vol

where for cell :math:`\sigma, \lambda_{vol}(\sigma)` denotes the *inverse compressibility* of the cell :math:`v(\sigma)` is the number of pixels in the cell (its *volume*), and :math:`V_t(\sigma)` is the cell’s *target volume*. This constraint defines :math:`P \equiv -2\lambda(v(\sigma) - V_t(\sigma))` as the *pressure* inside the cell. A cell with :math:`v < V_t`  has a positive internal pressure, while a cell with :math:`v > V_t` has a negative internal pressure.
Since many cells have nearly fixed amounts of cell membrane, we often use a *surface- area constraint* of form:

.. math:: H_{surf} = \sum_{\sigma} \lambda_{surf}(\sigma) (s(\sigma)-S_t(\sigma))^2
   :label: h_surf

where :math:`s(\sigma)` is the surface area of cell :math:`\sigma, S_t` is its target surface area and :math:`\lambda_{surf}(\sigma)` is its *inverse membrane compressibility* [1]_.
Adding the boundary energy and volume constraint terms together (equations (1) and (4)), we obtain the basic *GGH effective energy*:

.. math:: H_{GGH} = \sum_{i,j} J(\tau(\sigma(i)), \tau(\sigma(i)))(1-\delta(\sigma(i),\sigma(j))) + \sum_{\sigma} \lambda_{vol}(\sigma) (v(\sigma)-V_t(\sigma))^2
   :label: h_ggh

.. [1] Because of lattice discretization and the option of defining long range neighborhoods, the surface area of a cell scales in a non-Euclidian, lattice-dependent  manner with cell volume, i.e., :math:`s(v) \neq (4\pi)^{1/3}(3v)^{2/3}`   see [61] on bubble growth
