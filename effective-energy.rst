Effective Energy
------------------
The core of GGH simulations is the effective energy, which describes cell behaviors and interactions.

One of the most important effective-energy terms describes cell adhesion. If cells did not stick to each other and to extracellular materials, complex life would not exist [86]. Adhesion provides a mechanism for building complex structures, as well as for holding them together once they have formed. The many families of adhesion molecules (CAMs, cadherins, etc.) allow embryos to control the relative adhesivities of their various cell types to each other and to the noncellular ECM surrounding them, and thus to define complex architectures in terms of the cell configurations which minimize the adhesion energy.

To represent variations in energy due to adhesion between cells of different types, we define a *boundary energy* that depends on :math:`J(\tau(\sigma),\tau(\sigma'))` , the *boundary energy* per unit area between two cells :math:`(\sigma, \sigma')` of given types (:math:`\tau(\sigma), \tau(\sigma')`) at a link (the interface between two neighboring pixels):

.. math:: H_{boundary} = \sum_{i,j} J(\tau(\sigma(i)), \tau(\sigma(j)))) (1-\delta(\sigma(i), \sigma(j)))
  :label: (1)

where the sum is over all neighboring pairs of lattice sites   and   (note that the neighbor range may be greater than one), and the boundary-energy coefficients are symmetric,

.. math:: J(\tau(\sigma),\tau(\sigma')) = J(\tau(\sigma'), \tau(\sigma))
  :label: (2)

In addition to boundary energy, most simulations include multiple constraints on cell behavior. The use of constraints to describe behaviors comes from the physics of classical mechanics. In the GGH context we write constraint energies in a general elastic form:

.. math:: H_{constraint} = \lambda(value - target\_value)^2
   :label: (3)

The constraint energy is zero if :math:`value = target\_value` (the constraint is satisfied) and grows as value diverges from :math:`target\_value`. The constraint is *elastic* because the exponent of 2 effectively creates an ideal spring pushing on the cells and driving them to satisfy the constraint. :math:`\lambda` is the *spring constant* (a positive real number), which determines the *constraint strength*. Smaller values of :math:`\lambda` allow the pattern to deviate more from the *equilibrium condition* (i.e., the condition satisfying the constraint). Because the constraint energy decreases smoothly to a minimum when the constraint is satisfied, the energy-minimizing dynamics used in the GGH automatically drives any configuration towards one that satisfies the constraint. However, because of the stochastic simulation method, the cell lattice need not satisfy the constraint exactly at any given time, resulting in random fluctuations. In addition, multiple constraints may conflict, leading to configurations which only partially satisfy some constraints.

Because biological cells have a given volume at any time, most GGH simulations employ a *volume constraint*, which restricts volume variations of generalized cells from their target volumes:

.. math:: H_{vol} = \sum_{\sigma} \lambda_{vol}(\sigma) (v(\sigma)-V_t(\sigma))^2
   :label: (4)

where for cell :math:`\sigma, \lambda_{vol}(\sigma)` denotes the *inverse compressibility* of the cell :math:`v(\sigma)` is the number of pixels in the cell (its *volume*), and :math:`V_t(\sigma)` is the cell’s *target volume*. This constraint defines :math:`P \equiv -2\lambda(v(\sigma) - V_t(\sigma))` as the *pressure* inside the cell. A cell with :math:`v < V_t`  has a positive internal pressure, while a cell with :math:`v > V_t` has a negative internal pressure.
Since many cells have nearly fixed amounts of cell membrane, we often use a *surface- area constraint* of form:

.. math:: H_{surf} = \sum_{\sigma} \lambda_{surf}(\sigma) (s(\sigma)-S_t(\sigma))^2
   :label: (5)

where :math:`s(\sigma)` is the surface area of cell :math:`\sigma, S_t` is its target surface area and :math:`\lambda_{surf}(\sigma)` is its *inverse membrane compressibility* [1]_.
Adding the boundary energy and volume constraint terms together (equations (1) and (4)), we obtain the basic *GGH effective energy*:

.. math:: H_{GGH} = \sum_{i,j} J(\tau(\sigma(i)), \tau(\sigma(i)))(1-\delta(\sigma(i),\sigma(j))) + \sum_{\sigma} \lambda_{vol}(\sigma) (v(\sigma)-V_t(\sigma))^2
   :label: (6)

.. [1] Because of lattice discretization and the option of defining long range neighborhoods, the surface area of a cell scales in a non-Euclidian, lattice-dependent  manner with cell volume, i.e., :math:`s(v) \neq (4\pi)^{1/3}(3v)^{2/3}`   see [61] on bubble growth
