# Optimising a tuned mass damper integrated in the rotary boring tool holder
This branch consist of replicating the process and calculation of optimising the parameters of the mass damper tuned inside the rotary boring tool. The **Documented Research Paper** ipynb file has the explained code of the process.

Here is an overview of the file:
-    Calculation of Mode shape for solid bar and cavity intergrated solid bar in Modal space
-    Building simplified matrices of Mass, Damping, K stiffness to use them in extended Hamiltons principle. For both types of bars
-    Calculate Natural frequencies of the bars with inclusion of effects **(Coriolis, Gyroscopic, Centrifugal)** due to speed
-    With the help of matrices and hamiltons principle, frequency response function (FRF) is calculated in modal space for every frequency
-    An function is made to calculate Frf at any given speed which makes it easier to calculation stability limit
-    Averaged directional coefficients are calculated to account for varying angles made during cuts
-    Stability limit is calculated and also plotted with lower envelope using Budak-Altintas algorithm
-    Stability limit is also calculated from sweeping the frequency range calculated frf at a fixed speed
-    Optimisation of kd(stiffness of absorber's O-ring) and cd(damping coefficient of absorbers's oil) using Nedler Mead algorithm with average square root stability      limit function
-    Calculated kd and cd from Den Hartogs method to compare the results
-    Absorber Mass value sweep was done to find optimised cd and kd which is then compared with stability limit
-    Plotted & Replicated all the plots in the reference paper


An additional effort was made, with the refernce code, I have integrated multiple modes effecting the stability limit. Which is shown in **MultiModal** ipynb file.
