# Advanced setup of tuned mass absorber in rotary boring tool holder- Project 
This branch has the initial stages of the project statement which is adapted directly from the research paper code.

Project Statement:

**Optimization of Boring Bar Dynamics**
*    Rotary Boring Bar
*    Optimize the boing bar geometry and damper mass.

**Functions**:
+    Maximize limiting depth of cut or Minimize peak magnitude of receptance   
+    Minimize weight   
+    Maximize static stiffness

**Parameters**:
*    Position of cavity,
*    Length and diameter of cavity,
*    Volume of damper mass (less than 70% of the cavity volume),
*    Stiffness of spring (limiting to stiffness that can be provided by rubber spring),
*    Internal diameter of hollow section of body..

**Assumptions**:  
+    Hollow cross-section of body,
+    Damping oil with damping ratio 0.3,
+    Damper mass made of tungsten carbide,
+    Body made from material with Y =~280 GPa, rh0rho steel 1.2
+    Body comprises of 5 sections. Section 1 is a hollow cylinder of higher outer diameter than other sections. Section 2 has a decreasing outer diameter and           constant inner dia. Section3 is a hollow cylinder of constant cross-section. Section 4 is also hollow, but with a higher inner diameter to house the damper        mass. The thickness of this section must be equal to or higher than 3mm.
+    Section 5 is a solid cylindrical section. Outer diameters of sections 3,4 & 5 are same.
+    Rotational dynamics of the tool and the damper must be considered
