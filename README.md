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

## Framework (Basics)

At every timestep n, for every mesh cell, Lumerical solves the equation:\
$$U^{n}E^{n}+\frac{P^{n}}{e_{0}} = V^{n}$$
where E is the electric field, U and V are values provided by Lumerical, and P is the polarization that this custom model adds. Every physical model is wrapped into P.

## Full Derivation

This section will go through the equations governing each physical model, how they combine into an expression for P, how the equation for electric field is time discretized, and how Newton's method is used to analytically determine a final solution.

### Two-Photon Absorption (TPA)

TPA depends on the electric field magnitude, frequency of the light, and the TPA-coefficient $$\beta$$, which is a function of frequency.

$$P_{TPA} = \frac{n_{0}^{2}c^{2}e_{0}^{2}\beta}{2iw}E^{3}$$
