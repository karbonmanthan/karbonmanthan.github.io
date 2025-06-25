---
title: "On Chloris Geospatial's capability to assist with methdology VM0047"
description: "Chloris Geospatial's capability to assist with methdology VM0047"
tags: Remote_Sensing 
---

## On Chloris Geospatial's capability to assist with methodology VM0047

[Chloris Geospatial](https://www.chloris.earth/about-us) claims to have developed a Remote Sensing solution that can assist various carbon market actors involved in developing forest projects using VERRA's methodology VM0047. What precisely is their solution? And what potential does it hold for different stakeholders?

---

### Context - Nature based carbon projects are growing fast. 

Voluntary environmental carbon markets are witnessing an unprecedented rise in projects related to Nature based solutions across several standards. Forests are proven, time-tested carbon capture technology. And since their performance is this regard is scientifically demonstrable, there has been an increased incentive - environmental and financial - to develop projects under carbon mechanism Standards - like [Gold Standard](https://www.goldstandard.org/), [VERRA](https://verra.org/), [Global Carbon Council](https://globalcarboncouncil.com/), [Ecosystem Restoration Standard](https://www.ers.org/), [Climate Action Reserve](https://climateactionreserve.org/) etc. VERRA’s VM0047 methodology enables such demonstration by providing quantification approaches that tangentially rely on Remote Sensing. 

---

    
### Remote Sensing component in Methodology

Each of the above mentioned self-proclaimed Standard has publishes their own methodology that sets out a common quantification framework that has to be adopted in order to conservatively determine how much carbon is being sequestered by a specific forest - planted by the project developers under their scheme. These methodologies update slowly with time - getting better and more precise in each iteration.

A noticeable trend in this iteration is the increase role Remote Sensing is starting to play as the standards are slowly coming around to accepting models and modelling approaches. Models are almost unavoidable when using Remote Sensing approaches. 

**However, no one is focusing on standardizing 'data quality' and uncertainties across the different methodologies from different standards. Right now every methodology is handling these issues differently and there remains no common approach for what constitutes good enough 'approximations'**.

VERRA’s [VM0047 Methodology](https://verra.org/methodologies/vm0047-afforestation-reforestation-and-revegetation-v1-1/) has been faster in adopting the advances in remote sensing – which plays a relatively larger role than the Gold Standard's [AFFORESTATION/REFORESTATION GHG EMISSIONS REDUCTION & SEQUESTRATION METHODOLOGY v.2.1](https://globalgoals.goldstandard.org/403-luf-ar-methodology-ghgs-emission-reduction-and-sequestration-methodology/). VM0047 adopts Remote Sensing for additionality demonstration and developing dynamic baseline. Baseline is termed "dynamic" since VM0047 updates baseline over time using satellite date. By contract Gold Standard's methodology which relies on "static" baseline fixed at the start that does not undergo any update in practice - a more outdated approach in comparison. (Side note - This discussion pertains only to what VM0047 describes as - "Area based approach").

#### VM0047's technical approach breakdown

* VM0047 requires the Project Developer to define "control plots" - areas that represent the business-as-usual Baseline conditions - in contrast to the "project plots" - that instead represent the project scenario of afforestation/reforestation/revegetation. These "control plots" are similar to project plots but are outside the project area where "Stocking Index" is monitored using Remote Sensing. The assumption is that control plots that are left alone by the project developer will represent the situation of what would have happened at the project area had the project developer left it alone - i.e. business-as-usual scenario.
  
* Appendix 1 of VM0047 outlines for project developers how the potential control points from chosen from a donor pool. See [Project 5085](https://registry.verra.org/app/projectDetail/VCS/5085) for an example of how this approach was adopted in Burkina Faso.

<img src="https://github.com/karbonmanthan/karbonmanthan.github.io/blob/assets/control_point.png" alt="Figure from the public Validation Report linked above.">


* "Stocking index" – is a remote sensing based score that estimates forest biomass above ground (a "proxy for above ground biomass"). Examples include - NDVI, NDFI or EVI. Regression is performed between observed above ground biomass to stocking index to derive the slope of the time series. Slope of stocking index from "project plots" is compared with “control points” to check that the project is really adding more carbon.  The project is required to show that there is significant statistical difference between the stocking index of control points and project plots.
  

* $$
  \text{"Performance Benchmark"} = \frac{\text{average change or slope in Stocking Index in control plots}}{\text{average change or slope in Stocking Index in the project area}}
  $$
  
  where if the average change or slope in Stocking Index is negative or statistically insignificant (at P<0.05 level), then Performance Benchmark is set to zero.

* Negative slope of stocking indices over "control plots" means a performance benchmark closer to zero and the project is additional.

---

### Innovation from Chloris

Solution proposed by Chloris aligns with Appendix 1 of VM0047 and might help VVBs trace the full process — from data selection to benchmark calculation.

Usually indices like NDFI, NDVI, EVI are used. These traditional indices (derived from subtracting different spectral bands of satellite images of Landsat/Sentinel) measure vegetation “greenness” but are noisy and seasonal. Chloris seems to have developed their own custom index – by fusing data from multiple satellite sensors, incorporating it with LiDAR estimates to be used in Machine Learning model's training datasets that spits out their index - let's call it Chloris index. This index, Chloris claims to be 
* way more stable than standard indices mentioned (NDFI etc.) 
* less affected by seasonal changes and noise
* better at showing real carbon gains and perhaps higher additionality demonstration.

This is based on the following result Chloris has published on its website.

<img src="https://github.com/karbonmanthan/karbonmanthan.github.io/blob/main/assets/Chloris.jpg"
 alt="Comparision of stocking indices from [Chloris](https://www.chloris.earth/resources/gettingvm0047right)">

---

### Potential for VVBs

A Validation and Verification Body (VVB) is a third party with a mandate to verify that the project demonstrate its impacts, its additionality and baseline at each validation and verification stage. And since for VM0047, testing additionality and baseline involves assessing Performance Benchmark dynamically, solution from Chloris in this regard – stands to potentially help the VVBs to be able to test the veracity of impacts being claimed by the project developers - **if** the Chloris’s solution is clear and precise in demonstrating how exactly that remote sensing quantification mentioned in Appendix 1 of the methodology is applied for a project. A VVB needs to be able to track how a project is arriving at their "Performance Benchmark" - step by step - from start (donor pool) to end (benchmark) for a project area

### Potential for Project Developers

Appendix 1 remains a complicated component of the methodology that the project developer needs to contend with if they are looking to develop projects under this methodology. Chloris claims in its blogs to provide Project developers help in even deriving donor pool definition. This could mean that a potential project developer can completely subcontract its Appendix 1 related quantification to players like Chloris that can provide such an expertise.
        
         
### Potential for Digital Monitoring Reporting and Verification.
    
The entire market is busy brainstorming strategies for DMRV in the AFLOU sector. Chloris clearly forms part of a pool of potential technology provider. As the approaches developed by Chloris are clearly headed towards exploiting the possibility of near-real time measurements of even project scenarios (not just baselines). The campaigns runs by Chloris and the technical updates in their products reflect a clear focus on diversifying biomes and geographical locations to which their solutions can be applied – a major technological challenge of DMRV.


Will Chloris is able to fulfill its role as a stable and reliable tech. provider? Only time will tell but they seem to have a promising bunch of people.

---
