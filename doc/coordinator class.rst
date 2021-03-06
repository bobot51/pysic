.. file:coordinator class

.. _coordinator class:



.. file:coordinator class - description

.. _coordinator class - description:



.. module:: pysic.interactions.bondorder

============================
Coordinator class
============================

Coordinator is short for 'Coordination Calculator'.

This class provides a utility for calculating and storing bond order factors
needed for bond order or Tersoff-like potentials. Here, bond order refers
roughly to the number of neighbors of an atom, however, the bond order factors may
depend also on the atomic distances, angles and other local geometric factors.

To use a :class:`~pysic.interactions.bondorder.Coordinator`, one must pass it first to a :class:`~pysic.interactions.local.Potential` object,
which is further given to a :class:`~pysic.calculator.Pysic` calculator. Then, one can use the
calculator to calculate forces or just the bond order factors. When a :class:`~pysic.interactions.bondorder.Coordinator`
is added to a :class:`~pysic.interactions.local.Potential`, the potential is multiplied by the bond order factors
as defined by the Coordinator.

.. file:bond order potentials

.. _bond order potentials:




Bond order potentials
---------------------

A bond order factor can be added to any :class:`~pysic.interactions.local.Potential` object. 
This means that the potential is multiplied by the bond order factor. The factors are always
defined by atom, but for a two and many body potentials the average is applied.
To put in other words, if we have, say, a three-body potential

.. math::

   V = \sum_{(i,j,k)} v_{ijk},

where the sum goes over all atom triplets (i,j,k), adding a bond order factor 'b' will modify this to

.. math::

   V = \sum_{(i,j,k)} \frac{1}{3}(b_i + b_j + b_k) v_{ijk}.

The corresponding modified force (acting on atom alpha) would be

.. math::

   F_{\alpha} = -\nabla_{\alpha} V = - \sum_{(i,j,k)} \frac{1}{3}(\nabla_{\alpha} b_i + \nabla_{\alpha} b_j + \nabla_{\alpha} b_k) v_{ijk} + \sum_{(i,j,k)} \frac{1}{3}(b_i + b_j + b_k) f_{\alpha,ijk} .

where

.. math::

   f_{\alpha,ijk} = -\nabla_{\alpha} v_{ijk}

is the gradient of the unmodified potential.

Note that since the bond factor of an atom usually depends on its whole neighborhood, moving a neighbor of an atom may change the bond factors.
In other words, the gradients

.. math::

   \nabla_{\alpha} b_i + \nabla_{\alpha} b_j + \nabla_{\alpha} b_k

can be non-zero for values of alpha other than i, j, k. Thus adding a bond factor to a potential effectively increases the number of bodies in the interaction.

.. file:bond order parameter wrapping

.. _bond order parameter wrapping:




Parameter wrapping
------------------

Bond order factors are defined by atomic elements (chemical symbols).
Unlike potentials, however, they may incorporate different parameters and cutoffs for different
elements and in addition, they may contain parameters separately for single elements,
pairs of elements, element triplets etc. Due to this, a bond order factor can contain
plenty of parameters. 

To ease the handling of all the parameters, a wrapper class
:class:`~pysic.interactions.bondorder.BondOrderParameters` is defined. 
A single instance of this class defines the type of bond order factor and
contains the cutoffs and parameters for one set of elements. 
The :class:`~pysic.interactions.bondorder.Coordinator` object then collects these parameters in one bundle.

The bond order types and all associated parameters are explained in the documentation
of :class:`~pysic.interactions.bondorder.BondOrderParameters`.

.. file:mixing bond order types

.. _mixing bond order types:





Bond order mixing
-----------------

In general, bond order factors are of the form

.. math::

   b_i = s_i( \sum_{(i,j,\ldots)} c_{ij\ldots} )

where :math:`c_{ij\ldots}` are local environment contributors and :math:`s_i` is
a per-atom scaling function. For example, if one would define a factor

.. math::

   b_i = 1 + \sum_{(i,j)} f(r_{ij}),

then

.. math::

   c_{ij} = f(r_{ij}) \\
   s_i(x) = 1 + x.

When bond order factors are evaluated, the sums :math:`\sum_{(i,j,\ldots)} b_{ij\ldots}`
are always calculated first and only then the scaling :math:`s_i` is applied atom-by-atom.

When a :class:`~pysic.interactions.bondorder.Coordinator` contains several :class:`~pysic.interactions.bondorder.BondOrderParameters`::

     >>> crd = pysic.Coordinator( [ bond1, bond2, bond3 ] )

they are all added together in the bond order sums :math:`\sum_{(i,j,\ldots)} b_{ij\ldots}`. 
Mixing different types of bond order factors is possible but not recommenended as the results
may be unexpected.

The scaling is always carried out at most only once per atom. This is done as follows. The list
of bond order parameters is searched for a parameter set which requires scaling and which
contains 1-body parameters for the correct element. (That is, the first atomic symbol of
the list of targets of the parameter must equal the element of the atom for which the scaling
is done.) Once such parameters are found, they are used for scaling and the rest of the parameters
are ignored. In practice this means that the first applicable set of parameters in the list of 
:class:`~pysic.interactions.bondorder.BondOrderParameters` in the :class:`~pysic.interactions.bondorder.Coordinator` is used.

Because of this behaviour, the default scaling can be overridden as shown in the following example.

Let's say we wish to create a potential to bias the coordination number of Cu-O bonds
of Cu atoms, :math:`n`, according to

.. math::

   V(n) = \varepsilon \frac{\Delta N}{1 + \exp(\gamma \Delta N)}\\
   \Delta N = C (n - N).

In general, this type of a potential tries to push :math:`n` towards :math:`N`, a given parameter.

We can define this in pysic by overriding the scaling of the coordination bond order factor.::

   >>> bond_sum = pysic.BondOrderParameters( 'neighbors', cutoff = 4.0, 
   ...                                       cutoff_margin = 1.0,
   ...                                       symbols = [['Cu','O']] )
   >>> bond_scale = pysic.BondOrderParameters( 'c_scale', symbols = ['Cu'],
   ... 		    			       parameters=[epsilon,
   ...					       N,
   ...					       C,
   ...					       gamma] )
   >>> crd = pysic.Coordinator( [bond_scale, bond_sum] )
   >>> pot = pysic.Potential( 'constant', symbols = ['Cu'], 
   ...                        parameters = [1.0], coordinator = crd )

In the final step, the :class:`~pysic.interactions.bondorder.Coordinator` is attached to a 
:class:`~pysic.interactions.local.Potential` with a constant value of 1.0. 
Since the result is a product between the bond order factor and
the potential, the resulting potential is just the bond order factor.

.. file:coordinator class - autogenerated

.. _coordinator class - autogenerated:




List of methods
---------------

Below is a list of methods in :class:`~pysic.interactions.bondorder.Coordinator`, grouped according to
the type of functionality.

Parameter handling
__________________

- :meth:`~pysic.interactions.bondorder.Coordinator.add_bond_order_parameters`
- :meth:`~pysic.interactions.bondorder.Coordinator.set_bond_order_parameters`
- :meth:`~pysic.interactions.bondorder.Coordinator.get_bond_order_parameters`


Coordination and bond order
___________________________

- :meth:`~pysic.interactions.bondorder.Coordinator.calculate_bond_order_factors`
- :meth:`~pysic.interactions.bondorder.Coordinator.get_bond_order_factors`
- :meth:`~pysic.interactions.bondorder.Coordinator.get_bond_order_gradients`
- :meth:`~pysic.interactions.bondorder.Coordinator.get_bond_order_gradients_of_factor`

Miscellaneous
_____________

- :meth:`~pysic.interactions.bondorder.Coordinator.get_group_index` (meant for internal use)
- :meth:`~pysic.interactions.bondorder.Coordinator.set_group_index` (meant for internal use)


Full documentation of the Coordinator class
-------------------------------------------

.. currentmodule:: pysic.interactions.bondorder
.. autoclass:: Coordinator
   :members:
   :undoc-members:

