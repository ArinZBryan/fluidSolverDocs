Edited 1 File:
- FluidSimulator2D.cs

This commit was made to preserve a strange bug in the code which resulted in densities in the simulation spreading infinitely. This means that instead of a given density in an area reducing as the mass of particle spreads out, it remains the same or greater. This in effect gives the simulation an infinite mass, which is not desired.

Example Image here
(Commit 8db3c39)