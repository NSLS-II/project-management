Work relating to HKL

I think the first step here is to decide on a common 'hkl_object' and an approach to defining them. The current 'hkl_objects' provided by `hklpy` do not yet satisfy all of the requirements, but do form a good basis.

### Requirements for all diffractometer `hkl_object`'s based on requirements from CSX, CHX, J-PLS and ISR

- [ ] Be able to find the current location in reciprocal-space (hkl) and real-space values.
    - this can be done by the 2 commands:
    ```
    hkl_object.position  # reciprocal space position in hkl co-ordinates.
    hkl_object.real_postiion  # real-space position.
    ```
- [ ] Be able to calculate the sample orientation based on a known real_space and reciprocal_space location.
    - The main 'use case' for this is, a reflection (and hence the values of h,k,l) has been found at a defined 'real space' location which does not match the calculated value. This is most likely related to sample 'misalignment' in that one or more of the defined lattice angles is out by some 'offset' value.
    - It has been requested that we provide a method which determines what this mis-alignment is.
    - Note: this is not a current feature of hkl as far as I can tell.
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
    - I am so far not sure on how to approach this, essentially it would mean making the reduction form say a 6 circle diffractometer to a 4 circle diffractomer (by fixing the value of certain axes) changable.
- [ ] Be able to move the diffractometer to a given (hkl) position.
    - This can be done via the following:
    ```
    hkl_object.move(h,k,l)  # move to these 'h,k,l' co-ordinates
    ```
- [ ] Be able to, at a minimum, perform `scan`'s and `grid_scan`'s  in reciprocal space.
    - Note: In addition to scanning in 'reciprocal space' a request for 'energy' scanning with 'fixed Q' has also been made (by ISR).
         - This would involve fixed h,k,l and therefore non- fixed real-space locations (as the h,k,l calculation relies on photon energy).
        - This would also require us to 'couple' the beamlines 'photon energy' to the`hkl_objects` photon wavelength found at `hkl_object.calc.wavelength`.
