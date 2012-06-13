.. file:potential class

.. _potential class:



.. file:potential class - description

.. _potential class - description:



============================
Potential class
============================

This class defines an atomistic potential to be used by :class:`~pysic.calculator.Pysic`.
An interaction between two or more particles can be defined and the 
targets of the interaction can be specified by chemical symbol, tag, or index. 
The available types of potentials are always inquired from the
Fortran core to ensure that any changes made to the core are
automatically reflected in the Python interface.

There are a number of utility functions in :mod:`~pysic` for inquiring
the keywords and other data needed for creating the potentials. For example:

- Inquire the names of available potentials: :meth:`~pysic.list_valid_potentials`
- Inquire the names of parameters for a potential: :meth:`~pysic.names_of_parameters`
- Ask for a short description of a potential: :meth:`~pysic.description_of_potential`

.. file:potential cutoffs

.. _potential cutoffs:




Cutoffs
-------

Many potentials decay towards zero in infinity, but in a numeric simulation
they are cut at a finite range as specified by a cutoff radius. However, if the potential
is not exactly zero at this range, a discontinuity will be introduced. 
It is possible to avoid this by including a smoothening factor in the potential
to force a decay to zero in a finite interval:

.. math::

   \tilde{V}(r) = f(r) V(r),

where the smoothening factor is (for example)

.. math::

   f(r) = \begin{cases} 1, & r < r_\mathrm{soft} \\ \frac{1}{2}\left(1+\cos \pi\frac{r-r_\mathrm{soft}}{r_\mathrm{hard}-r_\mathrm{soft}}\right), & r_\mathrm{soft} < r < r_\mathrm{hard} \\ 0, & r > r_\mathrm{hard} \end{cases}.

Pysic allows one to specify both a hard and a soft cutoff for all potentials to include such
a smooth cutoff. If no soft cutoff if given or it is zero (or equal to the hard cutoff),
no smoothening is applied.


.. only:: html
 
 .. plot::

   import matplotlib.pyplot as plt
   from math import cos, pi
   start = 0.5
   end = 2.5
   steps = 100
   dx = (end-start)/steps
   off = 2.0
   plt.plot(0)
   plt.xlim(0.9,2.2)
   plt.ylim(-0.3,0.3)

   for m in range(2):
     margin = 0.3-m*0.3
     x = []; y = [];
     for i in range(steps+1):
       xval = start + i*dx
       if xval < off - margin:
           smooth = 1.0 
       elif xval > off:
           smooth = 0.0
       else:
           if margin == 0.0:
               smooth = 1.0
           else:
               smooth = 0.5*(1.0 + cos(pi*(xval - off + margin)/margin) ) 
       x.append( xval )
       y.append( smooth*((1.0/xval)**12 - (1.0/xval)**6) )

       plt.plot(x,y)

   plt.xlabel('$r$')
   plt.ylabel('$V$')
   plt.title(r'A potential with and without smoothening')
   plt.show()



.. file:list of potentials

.. _list of potentials:





List of currently available potentials
--------------------------------------

Below is a list of potentials currently implemented.

- :ref:`constant potential`
- :ref:`constant force potential`
- :ref:`power decay potential`
- :ref:`harmonic potential`
- :ref:`Lennard-Jones potential`
- :ref:`Buckingham potential`
- :ref:`charge exponential potential`
- :ref:`tabulated potential`
- :ref:`bond bending potential`
- :ref:`dihedral angle potential`


.. file:constant potential

.. _constant potential:




Constant potential
__________________

1-body potential defined as

.. math ::
   
   V(\mathbf{r}) = V,

i.e., a constant potential.

A constant potential is of course irrelevant in force calculation since the
gradient is zero. However, one can add a :class:`~pysic.BondOrderParameters` bond order
factor with the potential to create essentially a bond order potential. 

The constant potential can also be used for assigning an energy offset which depends on the
number of atoms of required types.

Keywords::

    >>> names_of_parameters('constant')
    ['V']


Fortran routines:

- :meth:`create_potential_characterizer_constant_potential`
- :meth:`evaluate_energy_constant_potential`


.. file:constant force potential

.. _constant force potential:




Constant force potential
________________________

1-body potential defined as

.. math ::
   V(\mathbf{r}) = - \mathbf{F} \cdot \mathbf{r},

i.e., a constant force :math:`\mathbf{F}`.

Keywords::

    >>> names_of_parameters('force')
    ['Fx', 'Fy', 'Fz']

Fortran routines:

- :meth:`create_potential_characterizer_constant_force`
- :meth:`evaluate_energy_constant_force`
- :meth:`evaluate_force_constant_force`



.. file:power decay potential

.. _power decay potential:




Power decay potential
_____________________

2-body interaction defined as

.. math::

    V(r) = \varepsilon \left( \frac{a}{r} \right)^n

where :math:`\varepsilon` is an energy scale constant, :math:`a` is a lenght scale constant or lattice parameter, and :math:`n` is an exponent.

There are many potentials defined in Pysic which already incorporate power law terms, and so this potential is often not needed. Still, one can build, for instance, the :ref:`Lennard-Jones potential` potential by stacking two power decay potentials. Note that :math:`n` should be large enough fo the potential to be sensible. Especially if one is creating a Coulomb potential with :math:`n = 1`, one should not define the potential through :class:`~pysic.Potential` objects, which are directly summed, but with the :class:`~pysic.CoulombSummation` class instead.

Keywords::

    >>> names_of_parameters('power')
    ['epsilon', 'a', 'n']

.. only:: html

 .. plot::
   
   import matplotlib.pyplot as plt
   from math import exp
   x = []; y = []
   start = 0.9
   end = 5.0
   steps = 100
   dx = (end-start)/steps
   for i in range(steps+1):
       xval = start + i*dx
       x.append( xval )
       y.append( 1.0 / (xval*xval) )
   plt.plot(x,y)
   plt.title(r'Power decay potential: $\varepsilon = 1.0$, $a = 1.0$, $n = 2.0$')
   plt.xlim(0.9,5.0)
   plt.ylim(0.0,1.0)
   plt.xlabel('$r$')
   plt.ylabel('$V$')
   plt.show()


Fortran routines:

- :meth:`create_potential_characterizer_power`
- :meth:`evaluate_energy_power`
- :meth:`evaluate_force_power`


.. file:harmonic potential

.. _harmonic potential:




Harmonic potential
__________________

2-body interaction defined as

.. math::

    V(r) = \frac{1}{2} k (r-R_0)^{2} - \frac{1}{2} k (r_\mathrm{cut}-R_0)^2,

where :math:`k` is a spring constant, :math:`R_0` is the equilibrium distance, and :math:`r_\mathrm{cut}` is the potential cutoff. The latter term is a constant whose purpose is to remove the discontinuity at cutoff.

Keywords::

    >>> names_of_parameters('spring')
    ['k', 'R_0']

.. only:: html

 .. plot::

   import matplotlib.pyplot as plt
   from math import sqrt
   x = []; y = []
   start = -0.0
   end = 3.5
   steps = 100
   dx = (end-start)/steps
   for i in range(steps+1):
       xval = start + i*dx
       x.append( xval )
       if xval < 3.0:
           y.append( 0.5 * (sqrt(xval**2)-1.0)**2 - 2.0 )
       else:
           y.append( 0.0 )
   plt.plot(x,y)
   plt.title(r'Harmonic potential: $k = 1.0$, $R_0 = 1.0$, $r_\mathrm{cut} = 3.0$')
   plt.xlim(-0.0,3.5)
   plt.ylim(-2.5,0.5)
   plt.xlabel('$r$')
   plt.ylabel('$V$')
   plt.show()


Fortran routines:

- :meth:`create_potential_characterizer_spring`
- :meth:`evaluate_energy_spring`
- :meth:`evaluate_force_spring`


.. file:Lennard-Jones potential

.. _Lennard-Jones potential:




Lennard-Jones potential
_______________________

2-body interaction defined as

.. math::

   V(r) = \varepsilon \left[ \left( \frac{\sigma}{r} \right)^{12} - \left( \frac{\sigma}{r} \right)^{6} \right],

where :math:`\varepsilon` is an energy constant defining the depth of the potential well and :math:`\sigma` is the distance where the potential changes from positive to negative in the repulsive region.

Keywords::

    >>> names_of_parameters('LJ')
    ['epsilon', 'sigma']    

.. only:: html
 
 .. plot::
   
   import matplotlib.pyplot as plt
   x = []; y = []
   start = 0.5
   end = 2.0
   steps = 100
   dx = (end-start)/steps
   for i in range(steps+1):
       xval = start + i*dx
       x.append( xval )
       y.append( (1.0/xval)**12 - (1.0/xval)**6 )
   plt.plot(x,y)
   plt.plot(0)
   plt.xlim(0.9,2.0)
   plt.ylim(-0.3,0.3)
   plt.xlabel('$r$')
   plt.ylabel('$V$')
   plt.title(r'Lennard-Jones potential: $\varepsilon = 1.0$, $\sigma = 1.0$')
   plt.show()


Fortran routines:

- :meth:`create_potential_characterizer_LJ`
- :meth:`evaluate_energy_LJ`
- :meth:`evaluate_force_LJ`

.. file:Buckingham potential

.. _Buckingham potential:




Buckingham potential
____________________

2-body interaction defined as

.. math::

    V(r) = A e^{-\frac{r}{\sigma}} - C \left( \frac{\sigma}{r} \right)^6

where :math:`\sigma` is a length scale constant, and :math:`A` and :math:`C` are the energy scale constants for the exponential and van der Waals parts, respectively.

Keywords::

    >>> names_of_parameters('Buckingham')
    ['A', 'C', 'sigma']

.. only:: html

 .. plot::
   
   import matplotlib.pyplot as plt
   from math import exp
   x = []; y = []
   start = 0.9
   end = 3.0
   steps = 100
   dx = (end-start)/steps
   for i in range(steps+1):
       xval = start + i*dx
       x.append( xval )
       y.append( exp(-xval) - (1.0/(xval))**6 )
   plt.plot(x,y)
   plt.title(r'Buckingham potential: $A = 1.0$, $C = 1.0$, $\sigma = 1.0$')
   plt.xlim(0.9,3.0)
   plt.ylim(-0.2,0.2)
   plt.xlabel('$r$')
   plt.ylabel('$V$')
   plt.show()


Fortran routines:

- :meth:`create_potential_characterizer_buckingham`
- :meth:`evaluate_energy_buckingham`
- :meth:`evaluate_force_buckingham`


.. file:charge exponential potential

.. _charge exponential potential:




Charge dependent exponential potential
______________________________________

2-body interaction defined as

.. math::

  V(r,q) & = \varepsilon_{ij} \exp\left(-\zeta_{ij} r + \frac{\xi_i D_i(q_i) + \xi_j D_j(q_j)}{2} \right) \\
  D_i(q) & = R_{i,\max} + | \beta_i (Q_{i,\max} - q) |^{\eta_i} \\
  \beta_i & = \frac{ (R_{i,\min} - R_{i,\max})^\frac{1}{\eta_i} }{ Q_{i,\max} - Q_{i,\min} }\\
  \eta_i  & = \frac{ \ln \frac{R_{i,\max}}{R_{i,\max} - R_{i,\min}} }{ \ln \frac{Q_{i,\max}}{Q_{i,\max} - Q_{i,\min}} },

where :math:`\varepsilon` is an energy scale constant, :math:`\zeta` is a length decay constant, :math:`\xi_i` are charge decay constants, and :math:`R_{i,\min/\max}` and :math:`Q_{i,\min/\max}` are the changes in valence radii and charge, respectively, of the ions for the minimum and maximum charge. :math:`D_i(q)` is the effective atomic radius for the charge :math:`q`.


Keywords::

    >>> names_of_parameters('exponential')
    ['epsilon', 'zeta', 
     'Rmax1', 'Rmin1', 'Qmax1', 'Qmin1', 
     'Rmax2', 'Rmin2', 'Qmax2', 'Qmin2', 
     'xi1', 'xi2']

.. only:: html

 .. plot::

   import matplotlib.pyplot as plt
   import math

   x = []; y = []
   start = 0.0
   end = 10.0
   steps = 100
   dx = (end-start)/steps

   eps = -1.0
   zeta = 0.5
   xi1 = 2.0
   xi2 = 4.0
   Rmax1 = -0.15
   Rmin1 = 0.15
   Rmax2 = -1.5
   Rmin2 = 0.5
   Qmax1 = 2.0
   Qmin1 = -6.0
   Qmax2 = 6.0
   Qmin2 = -2.0
   eta1 = ( math.log((Rmax1)/(Rmax1-Rmin1)) ) / ( math.log((Qmax1)/(Qmax1-Qmin1)) )
   eta2 = ( math.log((Rmax2)/(Rmax2-Rmin2)) ) / ( math.log((Qmax2)/(Qmax2-Qmin2)) )
   beta1 = (Rmin1-Rmax1)**(1/eta1) / (Qmax1-Qmin1)
   beta2 = (Rmin2-Rmax2)**(1/eta2) / (Qmax2-Qmin2)

   x = [[] for i in range(3*3)]
   y = [[] for i in range(3*3)]
   index = 0

   for dq1 in range(3):
     for dq2 in range(3):
       q1 = -1.0+dq1*1.0
       q2 = -1.0+dq2*1.0
       D1 = Rmax1 + (beta1 * (Qmax1 - q1))**eta1
       D2 = Rmax2 + (beta2 * (Qmax2 - q2))**eta2

       for i in range(steps+1):
           xval = start + i*dx
           x[index].append( xval )
           y[index].append( eps * math.exp(-zeta*xval + 0.5*(D1*xi1 + D2*xi2)) )
       
        
       plt.plot(x[index],y[index],color=str(index*0.1))
       index += 1

   plt.title(r'Charge dependent exponential potential:')
   plt.xlim(-0.0,5.0)
   plt.ylim(-1.0,0.0)
   plt.xlabel('$r$')
   plt.ylabel('$V$')
   plt.show()




Fortran routines:

- :meth:`create_potential_characterizer_charge_exp`
- :meth:`evaluate_energy_charge_exp`
- :meth:`evaluate_force_charge_exp`
- :meth:`evaluate_electronegativity_charge_exp`

.. file:tabulated potential

.. _tabulated potential:




Tabulated potential
_____________________

2-body interaction of the type :math:`V(r)` defined with a tabulated list of values. 

The values of the potential are read from text files of the format::

  V(0)     V'(0)*R
  V(dr)    V'(dr)*R
  V(2*dr)  V'(2*dr)*R
  ...      ...
  V(R)     V'(R)*R

where ``V`` and ``V'`` are the numberic values of the potential and its derivative (:math:`V(r)` and :math:`V'(r)`), respectively, for the given values of atomic separation :math:`r`. The grid is assumed to be uniform and starting from 0, so that the values of :math:`r` are :math:`0, \mathrm{d}r, 2\mathrm{d}r, \ldots, R`, where :math:`R` is the range of the tabulation and :math:`\mathrm{d}r = R/(n-1)` where :math:`n` is the number of value pairs (lines) in the table. Note that the first and last derivatives are forced to be zero regardless of what is given in the table (:math:`V'(0) = V'(R) = 0`).

For each interval :math:`[r_i,r_{i+1}]` the value of the potential is calculated using third degree spline (cspline) interpolation

.. math::

  r \in & [r_i,r_{i+1}]: \\
  t(r) = & \frac{r-r_i}{r_{i+1}-r_i} \\
  V(r) = & V(r_i) h_{00}(t) + V'(r_i) (r_{i+1}-r_i) h_{10}(t) + \\
         & V(r_{i+1}) h_{01}(t) + V'(r_{i+1}) (r_{i+1}-r_i) h_{11}(t) \\
  h_{00}(t) = & 2t^3-3t^2+1 \\
  h_{10}(t) = & t^3-2t^2+t \\
  h_{01}(t) = & -2t^3+3t^2 \\
  h_{11}(t) = & t^3-t^2 

The range of the tabulation :math:`R` is given as a parameter when creating the potential. The tabulated values should be given in a text file with the name ``table_xxxx.txt``, where ``xxxx`` is an identification integer with leading zeros (e.g., ``table_0001.txt``). The id is also given as a parameter when defining the potential. 

The tabulation range :math:`R` is not the same as a cutoff. It merely defines the value of :math:`r` for the last given value in the table. If the cutoff is longer than that, then :math:`V(r) = V(R)` for all :math:`r > R`. If the cutoff is shorter than the range, then as for all potentials, :math:`V(r) = 0` for all :math:`r > r_{\mathrm{cut}}`. A smooth cutoff (see :ref:`potential cutoffs`) can also be applied on top of the tabulation. The reason for this formalism is that once you have tabulated your potential, you may change the cutoff and smoothening marginal without having to retabulate the potential. 

You can also scale the potential in :math:`r`-axis just by changing the range :math:`R`, which is the reason why the table needs to include the derivatives multiplied by :math:`R`: Say you tabulate a potential for a given :math:`R`, obtaining values :math:`V(r), V'(r)R`. If you scale the :math:`r`-axis of the potential by a factor :math:`\rho`, :math:`R^* = \rho R`, you obtain a new potential :math:`U(r)`. This new potential is a scaling of the original potential :math:`U(r) = V(r/\rho), U'(r) = V'(r/\rho)/\rho`. The tabulated :math:`r` values are scaled so that the :math:`i`\ th value becomes :math:`r^*_i = \rho r_i`, and so the expected tabulated values are for the potential :math:`U(r_i^*) = V(r_i^*/\rho) = V(r_i)` and for the derivative :math:`U'(r^*_i)R^* = V'(r^*_i/\rho)R^*/\rho = V'(r_i)R`. That is, the tabulated values are invariant under the scaling of :math:`R`.

Finally, another parameter is available for scaling the energy scale :math:`V(r) \to \varepsilon V(r)`.

The file containing the tabulated values is directly read in by the Fortran core, and it is not allowed to contain anything besides two equally long columns of real numbers separated by white spaces.

Keywords::

    >>> names_of_parameters('tabulated')
    ['id', 'range', 'scale']

.. only:: html

The example plot below shows the resulting potentials for table::

  1.0  0.0
  0.5  0.0
  0.0  0.0

as well as similar tables with the second number on the second row replaced by -1.0 or -2.0 with range :math:`R=2.0` (i.e., the midpoint gets derivative values of 0.0, -0.5, or -1.0).


 .. plot::
   
   import matplotlib.pyplot as plt
   from math import exp
   x = []; y = []; y2 = []; y3 = []
   xp = [0,1,2]
   yp = [1,0.5,0]
   start = 0.0
   end = 2.0
   steps = 100
   dx = (end-start)/steps
   for i in range(steps+1):
       xval = start + i*dx
       x.append( xval )
       if xval < 1.0:
           t = xval
           y.append( 1.0 * (2*t**3-3*t**2+1) + 0.5 * (-2*t**3+3*t**2))
           y2.append( 1.0 * (2*t**3-3*t**2+1) - 0.5 * (t**3-t**2) + 0.5 * (-2*t**3+3*t**2) )
           y3.append( 1.0 * (2*t**3-3*t**2+1) - 1.0 * (t**3-t**2) + 0.5 * (-2*t**3+3*t**2) )
       else:
           t = xval-1.0
           y.append( 0.5 * (2*t**3-3*t**2+1) )
           y2.append( 0.5 * (2*t**3-3*t**2+1) - 0.5 * (t**3-2*t**2+t) )
           y3.append( 0.5 * (2*t**3-3*t**2+1) - 1.0 * (t**3-2*t**2+t) )
   plt.plot(x,y)
   plt.plot(x,y2)
   plt.plot(x,y3)
   plt.plot(xp,yp,'o')
   plt.title(r'Tabulated potential $R=2.0$')
   plt.xlim(0.0,2.0)
   plt.ylim(-0.1,1.1)
   plt.xlabel('$r$')
   plt.ylabel('$V$')
   plt.show()


Fortran routines:

- :meth:`create_potential_characterizer_table`
- :meth:`evaluate_energy_table`
- :meth:`evaluate_force_table`


.. file:bond bending potential

.. _bond bending potential:




Bond bending potential
______________________

3-body interaction defined as

.. math::

   V(\theta) = \frac{k}{2} ( \cos \theta - \cos \theta_0 )^2,

where :math:`k` is an angular spring constant, :math:`\theta` is an angle defined by three points in space (atomic positions) and :math:`\theta_0` is the equilibrium angle. The potential therefore describes an angular spring force related to bending of bonds.

Keywords::

    >>>	names_of_parameters('bond_bend')
    ['k', 'theta_0']

Three bodies form a triangle and so there are three possible angles the potential could bend. To remove this ambiguousness, the angle is defined so that as the potential is given a list of targets, the middle target is considered to be at the tip of the angle. 

Example::

    >>> pot = Potential('bond_bend')
    >>> pot.set_symbols(['H', 'O', 'H'])

This creates a potential for H-O-H angles, but not for H-H-O angles.

Also remember that the bond bending potential does not include any actual bonding potential between particles - it only generates an angular force component. It must be coupled with other potentials to build a full bonding potential.

.. only:: html

 .. plot::
   
   import matplotlib.pyplot as plt
   from math import cos, pi
   theta = []; r1 = []; r2 = []
   start = 0.0
   end = 2.0*pi
   steps = 100
   dtheta = (end-start)/steps
   for i in range(steps+1):
       thval = start + i*dtheta
       theta.append( thval )
       r1.append( 0.5 * (cos(thval)-cos(0.0))**2 )
       r2.append( 0.5 * (cos(thval)-cos(pi/2))**2 )

   plt.polar(theta,r1)
   plt.polar(theta,r2)   
   plt.title(r'Bond bending potential: $k = 1.0$, [$\theta_0 = 0$; $\theta_0 = \pi/2$]')
   plt.show()


Fortran routines:

- :meth:`create_potential_characterizer_bond_bending`
- :meth:`evaluate_energy_bond_bending`
- :meth:`evaluate_force_bond_bending`


.. file:dihedral angle potential

.. _dihedral angle potential:




Dihedral angle potential
________________________

4-body interaction defined as

.. math::

    V(r) = \frac{k}{2} (\cos \theta - \cos \theta_0)^2 

where :math:`k` is a spring constant and :math:`\theta` is the dihedral angle, with :math:`\theta_0` denoting the equilibrium angle.
(So, this is the cosine harmonic variant of the dihedral angle potential.)

For an atom chain 1-2-3-4 the dihedral angle is the angle between the bonds 1-2 and 3-4 when projected on the plane perpendicular to the bond 2-3. In other words, it is the torsion angle of the bond chain. If we write :math:`\mathbf{r}_{ij} = \mathbf{R}_j - \mathbf{R}_i`, where :math:`\mathbf{R}_i` is the coordinate vector of the atom :math:`i`, the angle is given by

.. math::

  \cos \theta =& \frac{\mathbf{p} \cdot \mathbf{p}'}{|\mathbf{p}||\mathbf{p}'|}\\
  \mathbf{p} =& -\mathbf{r}_{12} + \frac{\mathbf{r}_{12} \cdot \mathbf{r}_{23}}{|\mathbf{r}_{23}|^2}\mathbf{r}_{23}\\
  \mathbf{p}' =& \mathbf{r}_{34} - \frac{\mathbf{r}_{34} \cdot \mathbf{r}_{23}}{|\mathbf{r}_{23}|^2}\mathbf{r}_{23}

Keywords::

    >>> names_of_parameters('dihedral')
    ['k', 'theta_0']

Fortran routines:

- :meth:`create_potential_characterizer_dihedral`
- :meth:`evaluate_energy_dihedral`
- :meth:`evaluate_force_dihedral`


.. file:potential class - autogenerated

.. _potential class - autogenerated:




List of methods
---------------

    
Below is a list of methods in :class:`~pysic.Potential`, grouped according to
the type of functionality.


Interaction handling
____________________

- :meth:`~pysic.Potential.get_cutoff`
- :meth:`~pysic.Potential.get_cutoff_margin`
- :meth:`~pysic.Potential.get_parameter_names`
- :meth:`~pysic.Potential.get_parameter_value`
- :meth:`~pysic.Potential.get_parameter_values`
- :meth:`~pysic.Potential.get_potential_type`
- :meth:`~pysic.Potential.get_soft_cutoff`
- :meth:`~pysic.Potential.set_cutoff`
- :meth:`~pysic.Potential.set_cutoff_margin`
- :meth:`~pysic.Potential.set_parameter_value`
- :meth:`~pysic.Potential.set_parameter_values`
- :meth:`~pysic.Potential.set_soft_cutoff`

Coordinator handling
_____________________

- :meth:`~pysic.Potential.get_coordinator`
- :meth:`~pysic.Potential.set_coordinator`

Target handling
__________________

- :meth:`~pysic.Potential.accepts_target_list`
- :meth:`~pysic.Potential.add_indices`
- :meth:`~pysic.Potential.add_symbols`
- :meth:`~pysic.Potential.add_tags`
- :meth:`~pysic.Potential.get_different_indices`
- :meth:`~pysic.Potential.get_different_symbols`
- :meth:`~pysic.Potential.get_different_tags`
- :meth:`~pysic.Potential.get_indices`
- :meth:`~pysic.Potential.get_number_of_targets`
- :meth:`~pysic.Potential.get_symbols`
- :meth:`~pysic.Potential.get_tags`
- :meth:`~pysic.Potential.set_indices`
- :meth:`~pysic.Potential.set_symbols`
- :meth:`~pysic.Potential.set_tags`


Description
___________

- :meth:`~pysic.Potential.describe`



Full documentation of the Potential class
-----------------------------------------

.. currentmodule:: pysic
.. autoclass:: Potential
   :members:
   :undoc-members:
