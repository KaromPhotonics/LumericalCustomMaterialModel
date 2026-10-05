# Femtosecond Pulsed Laser Model (for Ansys Lumerical FDTD)

A custom material model for Ansys Lumerical FDTD designed to accurately model femtosecond pulsed laser free carrier densities.

This model includes the following physics:
- Single-photon absorption (SPA)
- Two-photon absorption (TPA)
- Free carrier absorption (FCA)
- Free carrier refraction (FCR)
- Kerr effect

By default, all coefficients are for Si at 1260 nm, but different materials / material stacks can be simulated by adjusting the coefficients and using multiple instances of the custom material.

## Purpose

Femtosecond pulsed lasers are essential in scientific research, as they allow researchers to probe nonlinear effects of materials or study ultra-fast molecular and atomic events in real-time. One specific use of pulsed lasers is in the field of radiation effects, which aims to study how energetic particles negatively interact with semiconductor/photonic devices. One type of radiation effects is a single-event effect (SEE), which involves a heavy ion passing through a device, depositing charge in the form of electron-hole pairs (EHPs). Depending on the strike location and magnitude of charge deposition, SEEs can result in loss of data and bit flips.

Historically, SEEs have been studied at particle accelerators capable of accelerating atoms to the energies seen in the space environment. This comes with a steep operating cost and limited beam time. One alternative to particle accelerators for SEE testing is using femtosecond pulsed lasers to inject similar densities of EHPs in a similar time scale to that of heavy ion strikes. As femtosecond pulsed lasers (one example being Ti-Sapphire) are more common and widespread compared to particle accelerators, using these lasers allows for increased testing capabilities. Femtosecond pulsed lasers offer more benefits than just convenience: by varying the pulse energy and focal point, different EHP densities can be deposited in various sensitive regions, allowing for full device parameterization with a single laser, compared to needing multiple ion cocktails at a traditional particle accelerator.

This custom material and simulation framework is designed to produce a 3D carrier density consistent with femtosecond pulsed lasers. This 3D carrier density can then be exported into standard electrical simulators (Sentaurus TCAD is the primary electrical simulator used in this framework) to determine a device's response to a specific carrier density, focal point, or radial distribution.

## How to Implement

There are two ways to obtain the custom material: downloading the .dll file directly or building it using a program such as Visual Studio. Once you obtain the .dll file, it needs to be moved into Lumerical's material database (Lumerical\bin\plugins\materials). This will require administrator permission. Once there, you can create a new material in Lumerical and select "Custom Material Model Example". Si parameters for a 1260 nm pulsed laser are defined by default.

To simulate the femtosecond pulsed laser, a Gaussian source should be defined with 

## Framework (Basics)

At every timestep n, for every mesh cell, Lumerical solves the equation:

$$U^{n}E^{n}+\frac{P^{n}}{e_{0}} = V^{n}$$

where E is the electric field, U and V are values provided by Lumerical, and P is the polarization that this custom model adds. Every physical model is wrapped into P.

## Full Derivation

This section will go through the equations governing each physical model, how they combine into an expression for P, how the equation for electric field is time discretized, and how Newton's method is used to analytically determine a final solution.

### Time Discretization

In order to analytically solve for $$E^{n+1}$$, it is necessary to transition from the frequency domain into the time domain by replacing $$-iw$$ with temporal derivatives ($$dt$$). From there, the derivatives are time discretized to align with the FDTD algorithm, which dictates that the displacement field $$D^{n+1}$$ is calculated from the curl of the magnetic field $$H^{n+\frac{1}{2}}$$.

### Two-Photon Absorption (TPA)

TPA depends on the electric field magnitude, frequency of the light, and the TPA-coefficient $$\beta$$, which is a function of frequency.

$$P_{TPA} = \frac{n_{0}^{2}c^{2}e_{0}^{2}\beta}{2iw}E^{3}$$

### Free-Carrier Generation

The free carrier density in each cell is updated each timestep according to the following equation:

$$G_{TPA} = \frac{\beta c^{2} n_{0}^{2} e_{0}^{2} E^{4}}{3hw}$$

### Free-Carrier Absorption (FCA)

Some of the light is absorbed by the free carriers generated during the laser pulse. FCA is modeled according to the following equation:

$$P_{FCA} = -\frac{n_{0} e_{0} c \sigma N}{iw} E$$

### Free-Carrier Refraction (FCR)

The existence of free carriers perturbs the refractive index in the laser pulse, thus shaping the shape of the pulse in real time. FCR is modeled according to the following equation:

$$P_{FCR} = 2 n_{0} e_{0} \triangle n(N) E$$

where $$\triangle n(N)$$ is the change in refractive index as a function of the free carrier density N. There exist two models which convert carrier density to changes in refractive index: Soref-Bennet and Drude.

- The Soref-Bennett model (fcr_model = 1) is only valid for Si. It is an empirical model that is fit to experimental data. It is a function of wavelength and has coefficients that are functions of wavelength. A list of coefficients can be found at [#]. For a wavelength of 1300 nm (closest wavelength to 1260 nm with coefficients), it is modeled according to the following equation:

$$\triangle n = -2.98E-22 \triangle N_{e}^{1.016} - 1.25E-18 \triangle N_{h}^{0.835}$$

- The Drude model (fcr_model = 2) is valid for all materials and is a function of wavelength and carrier effective mass. It is modeled according to the following equation:

$$\triangle n = -\frac{e^{2}\lambda^{2}}{8 \pi^{2} c^{2} e_{0} n_{0}} (\frac{N_{e}}{m_{e}^{@} m_{0}} + \frac{N_{h}}{m_{h}^{@} m_{0}})$$

### Kerr Effect

The Kerr effect is a change in refractive index in response to the local electric field strength. For femtosecond laser pulses, the Kerr effect becomes important and needs to be considered. The Kerr effect is modeled according to the following equation:

$$P_{Kerr} = \frac{4}{3} e_{0} n_{0}^{2} n_{2} E^{3}$$

where $$n_{2}$$ is the Kerr coefficient.
