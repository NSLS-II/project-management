Work relating to HKL

I think the first step here is to decide on a common 'hkl_object' and an approach to defining them. The current 'hkl_objects' provided by `hklpy` do not yet satisfy all of the requirements, but do form a good basis.

### Requirements for all diffractometer `hkl_object`'s based on requirements from CSX, CHX, J-PLS and ISR
(items without a current solution are in **bold**)
- [ ] Be able to find the current location in reciprocal-space (hkl) and real-space values.
    - this can be done by the 2 commands:
    ```
    hkl_object.position  # reciprocal space position in hkl co-ordinates.
    hkl_object.real_postiion  # real-space position.
    ```
- [ ] Be able to define a sample geometry
    - This can be done by using the steps:
    ```
    # import the hkl Lattice object
    from hkl.util import Lattice

    # Define a lattice, lengths are in Angstrom, angles are in degrees
    lattice = Lattice(a=9.069, b=9.069, c=10.390, alpha=90.0, beta=90.0, gamma=120.0)

    # add the sample to the calculation engine
    hkl_object.calc.new_sample('some_sample', lattice=lattice)
    
    # the sample geometry is then defined in
    hkl_object.calc.sample
    ```
- [ ] Be able to calculate the sample orientation based on a known real_space and reciprocal_space location.
    - The main 'use case' for this is, a reflection (and hence the values of h,k,l) has been found at a defined 'real space' location which does not match the calculated value. This is most likely related to sample 'misalignment' in that one or more of the defined lattice angles is out by some 'offset' value.
    - It has been requested that we provide a method which determines what this mis-alignment is.
    - Note: this misalignment correction is provided by the `hkl_object.sample.UB` object (a matrix) and the calculation based on 2 known reflections can be calculated by the code:
    ```
    # Define the reflection calculations
    r1=hkl_object.calc.sample.add_reflection(h1,k1,l1,
        position=hkl_object.calc.Position(axis1=val,axis2=val,axis3=val,axis4=val,axis5=val,axis6=val))
    r2=hkl_object.calc.sample.add_reflection(h2,k2,l2,
        position=hkl_object.calc.Position(axis1=val,axis2=val,axis3=val,axis4=val,axis5=val,axis6=val))
        
    # Define the photon energy or wavelength
    hkl_object.calc.energy=photon_energy 
    # _or_
    hkl_object.calc.wavelength=wavelength
    
    # Check the reflections are stored
    hkl_object.calc.sample.reflections  # print a list of current hkl reflections that have been stored
    
    # Perform calculation of UB matrix
    hkl_object.calc.sample.compute_UB(r1, r2)
    
    # Check the UB matrix
    hkl_object.calc.sample.UB
    ```
    
    - Note 1: I suspect we will want to add a method `hkl_object.calc.add_current_reflection(h,k,l)` which uses the current real axis location and then calls hkl_object.calc.sample.add_reflection()`

- [ ] **To be able to use the 'Beamline energy' in the calculation instead of the stored 'energy' (but also have the option of not using it)**.
    - I am not yet sure of exactly how to accomplish this, just throwing out ideas I can think of the following ways:
        1. an attribute that 'turns on or off' the link between `hkl_object.calc.sample.energy` and the beamlines energy object.
        2. add an `energy` kwarg to `hkl_object.calc.forward`, `hkl_object.calc.reverse` and `hkl_object.sample.compute_UB`. The default of such a kwarg should be `None` and in the case that `energy=None` the Beamlines 'energy' object value is used.
        3. add duplicates of `hkl_object.calc.forward`, `hkl_object.calc.reverse` and `hkl_object.sample.compute_UB` that use the beamlines energy ophyd object value instead of hkl_object.calc.sample.energy.
- [ ] Be able to calculate real-space values given the reciprocal-space (hkl) values and vice-versa.
    - This can be done using the built-in hkl methods:
    ```
    hkl_object.calc.wavelength = plancks_constant*speed_of_light/ photon_energy
    hkl_object.calc.forward((h,k,l))  # real-space positions based on inputted h,k,l co-ordinates
    hkl_object.calc.reverse(('mu',  'omega',  'chi',  'phi', 'gamma', 'delta'))  # hkl values from angles
    ```
    - Note 1: We may want to write a custom version of the reverse calculation that is a `partial`of this for the 5 axis diffractometer. along the lines of the following:
    ```
    def hkl_object(hkl.diffract.E6C):

        hkl_object.calc.reverse(value):  # where value = ('mu',  'omega',  'chi',  'phi', 'gamma')
            value.append(0)  # adds a zero for the 'delta' angle
            return super().calc.reverse(value)
    ```
    - Note 2: A similiar version of the above should probably be implemented for the forward calculation that limits the return value to only the known angles.

- [ ] Be able to choose _geometry modes_ for each of the diffractometers (which defines which rotation stages to include in calculations and motions).
    - This is probably what is done with the `hkl_object.calc.engine.mode` which must be one of `hkl_object.calc.engine.modes`
    - We should however check with ISR that the required 'modes' are defined here for their systems, and decide how to proceed if they are not.
- [ ] Be able to move the diffractometer to a given (hkl) position.
    - This can be done via the following:
    ```
    hkl_object.move(h,k,l)  # move to these 'h,k,l' co-ordinates
    ```
- [ ] **Be able to, at a minimum, perform `scan`'s and `grid_scan`'s  in reciprocal space.**
    - Note: In addition to scanning in 'reciprocal space' a request for 'energy' scanning with 'fixed Q' has also been made (by ISR).
         - This would involve fixed h,k,l and therefore non- fixed real-space locations (as the h,k,l calculation relies on photon energy).
        - This would also require us to 'couple' the beamlines 'photon energy' to the`hkl_objects` photon wavelength found at `hkl_object.calc.wavelength`.
        
- [ ] **Although the current implementation does allow for most of the requested use cases I think their are a few changes that will greatly improve the usability of this system(scanning in `hkl` co-ordinates is still not clear to me)**.
    1. produce a clear 'guide to using hkl' that shows step by step how to perform each of the items above.
    2. look at creating linking 'shortcuts' to some of the parameters so they are not 'hidden by obscurity', and to simplify the UI a little. For example:
    ```
    hkl_object.sample = hkl_object.calc.sample
    hkl_object.calc.mode = hkl_object.calc.engine.mode
    hkl_object.calc.modes = hkl_object.calc.engine.modes
    ```
