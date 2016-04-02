A New VRF Heat Recovery System Model
================

 **Rongpeng Zhang, Tianzhen Hong.**
 **Lawrence Berkeley National Laboratory**

 - Original Date: Jan. 16, 2016
 - Revision Date: Mar. 20, 2016
 

## Justification for New Feature ##

The proposed new feature introduces an innovative model to simulate the energy performance of Variable Refrigerant Flow (VRF) systems with heat recovery (HR) configurations, which is capable of achieving heat recovery from cooling zones to heating zones and providing cooling and heating operation simultaneously. This feature is a continuation of the new VRF heat pump (HP) system model implemented in EnergyPlus V8.4.

The new VRF HR model is developed for the 3-pipe VRF-HR systems, which is the dominant system configuration in the current VRF-HR market. Compared with the VRF-HP system, VRF-HR is more complicated in terms of system configuration and operational controls. To enable simultaneous cooling and heating, complex refrigerant management loop is implemented and one more heat exchanger is added to the outdoor unit (OU). The two outdoor heat exchangers can work at different combination of evaporator/condenser mode to handle diverse and changing indoor heating/cooling load requirements. This leads to varying refrigerant flow directions and different control logics for various operational modes, and therefore specific algorithm is needed for separate major operational modes.

The proposed new VRF-HR model inherits many key features of the newly implemented VRF-HP model. Compared with the existing system curve based VRF-HR model, the new model is more physics-based by simulating the refrigerant loop performance. It is able to consider the dynamics of more operational parameters, which is essential for the representation of the complex control logics (e.g., the adjustment of superheating degrees during low load operations). Furthermore, the proposed model implements component-level curves rather than the system-level curves, and thus requires much fewer curves as model inputs. These features considerably extend the modeling capabilities of the new VRF-HR model, including:

-	Allowing more accurate estimations of HR loss, a critical parameter in the VRF-HR operations.
-	Allowing variable evaporating and condensing temperatures in the indoor and outdoor units under a variety of operational conditions.
-	Allowing implementation of various control logics for different VRF-HR operational modes.
-	Allowing further modifications of operational parameters (e.g., evaporating temperature and superheating degrees) during low load conditions.
-	Allowing variable fan speed based on the temperature and zone load in the indoor units (IU).
-	Allowing an enhanced physics-based model to calculate piping loss in the refrigerant piping network (varies with refrigerant flow rate, operational conditions, pipe length, and pipe and insulation materials) instead of a constant correction factor. 
-	Allowing the potential simulation of demand response of VRF systems by directly slowing down the speed of compressors in the outdoor units with inverter technology.



## E-mail and  Conference Call Conclusions ##

•	Brent, Jan. 20. 2016

-	Comments:
This looks interesting.  I have no problems with the overall approach.  
It seems to me after studying Figure 3 that there might be a reason to organize it as having 12 modes, each of your 6 modes with and without the low load modification.
-	Reply:
You are right that every operational mode may have two sub-modes: with and without the low load modification. That will affect the operational conditions such as evaporating temperature levels, but will not change the system configurations, i.e., the piping connections and the refrigerant flow directions. So, it may not be quite necessary to increase the mode number from 6 to 12 just for the low load modifications. 
We defined the 6 operational mode mainly depending on the different system configurations. We tried to keep the mode number low to make the algorithm clear and concise, but are these six modes are so different that we cannot combine them. We have to develop specific algorithm for each of them.

•	Richard, Jan. 20. 2016

-	Comments:
1) I don't believe all manufacturers use a split condenser configuration. How will you model single coil condensers?
2) How will you know when to provide heating and cooling with (mode 3) and W/O HR mode (mode 2)?
3) Is there enough information about the piping system to accurately model piping losses?
-	Reply:
(1) It seems most current VRF-HR systems implement two OU heat exchangers to support enhanced control strategy. Not sure how widely used the single heat exchange systems are. I will double check with our manufacturer partner. Probably we will only handle the dominant system configurations in the market for now.
(2) The operational mode is determined by the algorithm, not the user, based on the load requirements and operational conditions. We have developed a specific HR loss calculation method making use of the refrigerant loop information. This plays a key role in the mode determinations. More details will be included in DD.
(3) The piping loss calculations in VRF-HR are similar to those in the VRF-HP model. The information it requires (including main pipe length, equivalent length, diameter, and insulation) can usually be obtained from the system engineering manual.

•	Jason Glazer, Jan. 27. 2016

-	Comments:
It would be great if either a user input or at least an output would show the ARHI Simultaneous Cooling and Heating Efficiency (SCHE) rating condition as described in AHRI 1230 and any other figures of merit that are unique for heat recovery VRF systems. Outputs of the rated IEER would be good too. 
-	Reply:
Good suggestions. We will ourput this key parameter for HR operations.

•	Bereket Nigusse, Jan. 27. 2016

-	Comments:
The VRF model description and the schematic diagram provided are for 3-pipe VRF-HR system.  Some VRF manufacturers use a 2-pipe VRF-HR system design. Have you considered the 2-pipe VRF-HR system model? 
-	Reply:
The new VRF HR model is for 3-pipe systems, which is the dominant system configuration in the VRF-HR market. Most VRF manufacturers adopt 3-pipe configuration (e.g., Daikin, Samsung, Carrier, LG, Toshiba, etc; except for Mitsubishi). The 2-pipe and 3-pipe systems are very different in terms of refrigerant loop operations, piping connections and control logic, so they cannot be covered by a generic physics based model. Currently we don't have resource to add a 2-pipe VRF HR model (a very expensive process to cover algorithm development, EnergyPlus implementation, obtaining measured data, and model validation). We will document this (current model for 3-pipe system) clearly in the EnergyPlus IORef and EngRef.


## Overview ##

The VRF-HR system has more complicated configurations than the VRF-HP system. Figure 1 shows the schematic chart of a 3-pipe VRF-HR system, a dominant system configuration in the VRF-HR market. As can be seen in the figure, there are two heat exchangers in the outdoor unit. They can work at different evaporator/condenser combinations to generate specific operational modes. Also note that there are a number of Four-Way Directional Valves (FWV) and Branch Selector (BS) Units in the system, which are used to create specific refrigerant piping connections for different operational mode. These features allow the system to provide simultaneous heating/cooling to address diverse space conditioning situations.

![](VRF-HR-Chart-Schematic.PNG)
Figure 1. Schematic chart of a 3-pipe VRF-HR system

Depending on the indoor cooling/heating requirements and the outdoor unit operational states, the operations of the VRF-HR system can be divided into six modes:
-	Mode 1: Cooling load only. No heating load. Both OU heat exchangers perform as condensers.
-	Mode 2: Simultaneous heating and cooling. The sum of cooling loads and compressor heat is much larger than the heating loads. Both OU heat exchangers perform as condensers.
-	Mode 3: Simultaneous heating and cooling. The sum of cooling loads and compressor heat is slightly larger than the heating loads. One OU heat exchanger perform as condenser while the other performs as evaporator.
-	Mode 4: Simultaneous heating and cooling. The sum of cooling loads and compressor heat is slightly smaller than the heating loads. One OU heat exchanger perform as condenser while the other performs as evaporator.
-	Mode 5: Simultaneous heating and cooling. The sum of cooling loads and compressor heat is much smaller than the heating loads. Both OU heat exchangers perform as evaporators.
-	Mode 6: Heating load only. No cooling load. Both OU heat exchangers perform as evaporators.

The heat balance diagram for all the six operational modes are shown Figure 2.

![](VRF-HR-Chart-HeatBalance.PNG)
Figure 2. Heat Balance Diagram for All Six VRF-HR Operational Modes

With the help of FWV and BS units, every operational mode has its own refrigerant piping connections for different refrigerant flow directions, as shown in Table 1. This leads to different refrigerant operations (HP charts shown in Table 2) and piping loss situations. The operational control logics for various modes are also different. Therefore, we need to design particular algorithm for different operational modes.

![](VRF-HR-Chart-Piping.PNG) 
![](VRF-HR-Chart-EnthalpyPressure.PNG)

The implementation of the proposed VRF HR algorithm will go to the method “HVACVariableRefrigerantFlow::CalcVRFCondenser_FluidTCtrl”, which was newly added in V8.4 specifically for the VRF-FluidTCtrl-HP model. Therefore, the implementation of the new VRF-HR feature is expected to generate no or little impacts on other parts of the codes. 

![](VRF-CodingHierarchy.PNG)
Figure 3. Coding Hierarchy of the VRF model in the current EnergyPlus V8.4


## Approach ##

The holistic logics of the new VRF-HR model are illustrated in Figure 4 to show the key simulation steps under different operational modes. More detailed calculation procedures are described in the following sections. Note that the algorithms of the VRF-HR Cooling Only Mode and Heating Only Mode are the same as those in the VRF-HP Cooling Mode and Heating Mode, respectively. Therefore, the following algorithm descriptions only focus on the VRF-HR Simultaneous Heating and Cooling Mode.

![](VRF-HR-AlgorithmOverview-Mode1.png)
(a) Cooling Only Mode 
![](VRF-HR-AlgorithmOverview-Mode6.png)
(b) Heating Only Mode 
![](VRF-HR-AlgorithmOverview-Mode2-5.png)
(c) Simultaneous Heating and Cooling Mode

Figure 4. Schematic chart of the new VRF-HR algorithm


##### Step 1: Obtaining zonal load/condition information

Obtain the following information for each zone from the zone modules within EnergyPlus: 

* zone sensible loads <span>$Q_{in, sensible}$</span>

* zone total loads <span>$Q_{in, total}$</span>

* indoor air temperature <span>$T_{in}$</span>

* indoor air humidity ratio <span>$W_{in}$</span>

If there is only cooling load and no heating load, go to the VRF-HR Cooling Only Mode, the algorithms of which is the same as those for the VRF-HP Cooling Mode. If there is only heating load and no cooling load, go to the VRF-HR Heating Only Mode, the algorithms of which is the same as those for the VRF-HP Heating Mode. Otherwise, go to the VRF-HR Simultaneous Heating and Cooling Mode and perform the following calculations.

##### Step 2: Calculate I/U required evaporating temperature
Evaluate the required coil surface air temperature <span>$T_{fs}$</span> and then the required evaporator refrigerant temperature <span>$T_{e,req}$</span> for each indoor unit with cooling requirements. Likewise, evaluate the required condenser refrigerant temperature <span>${T_{c,req}}$</span> for each indoor unit with heating requirements.
(Refer to Engineering Reference V8.4+: Step 1.2 in the VRF-FluidTCtrl-HP model for more details.)

##### Step 3: Calculate I/U effective evaporating temperature Te
There are two refrigerant temperature control strategies for the indoor unit: (1) *ConstantTemp*, (2) *VariableTemp*.
 - In the *ConstantTemp* strategy, <span>$T_e$</span> and <span>$T_c$</span> are kept at constant values provided by the user.
 - In the *VariableTemp* strategy, <span>$T_e$</span> and <span>$T_c$</span> are determined using the equations below: 
(Refer to Engineering Reference V8.4+: Step 1.3 in the VRF-FluidTCtrl-HP model for more details.)


##### Step 4: I/U condenser side piping loss calculations

This section calculates the I/U condenser side piping loss, which occurs at High and Low Pressure Gas Pipe where the refrigerant flowing from O/U compressor outlets to the I/U condensers. It includes both the refrigerant pressure drop <span>$\Delta{P_{pipe}}$</span> and heat loss <span>$Q_{pipe}$</span>. Note that the change of compressor operational conditions may lead to different control strategies of the system, which in reverse affects the amount of piping loss. So the piping loss analysis and system performance analysis are coupled together. Numerical iterations are designed to address the coupling effect, as described below.

In this step, the compressor discharge saturated temperature <span>$T'_c$</span> (i.e., saturated vapor temperature corresponding to compressor discharge pressure) can be obtained using the calculated refrigerant pressure drop <span>$\Delta{P_{pipe}}$</span>.
(Refer to Engineering Reference V8.4+: Step 2h.1 in the VRF-FluidTCtrl-HP model for more details.)

##### Step 5: I/U evaporator side piping loss calculations

This section calculates the I/U evaporator side piping loss, which occurs at Suction Gas Pipe where the refrigerant flowing from the I/U evaporators to the O/U compressor inlets. Similarly to the I/U condenser side piping loss, it includes both the refrigerant pressure drop <span>$\Delta{P_{pipe}}$</span> and heat loss <span>$Q_{pipe}$</span>. 

In this step, the compressor suction saturated temperature <span>$T'_e$</span> (i.e., saturated vapor temperature corresponding to compressor suction pressure) can be obtained using the calculated refrigerant pressure drop <span>$\Delta{P_{pipe}}$</span>.

Note that one key input of the calculation is the enthalpy of the refrigerant at I/U evaporator inlets. It is assumed to be equal to the average enthalpy of the refrigerant at I/U condenser outlets, which is obtained in the I/U condenser side piping loss calculations.
(Refer to Engineering Reference V8.4+: Step 2c.1 in the VRF-FluidTCtrl-HP model for more details.)

##### Step 6: Determine the operational mode for simultaneous heating and cooling operations

As noted earlier, simultaneous heating and cooling operations include the following modes:
-	Mode 2: Cooling dominant w/o HR loss
-	Mode 3: Cooling dominant w/ HR loss
-	Mode 4: Heating dominant w/ HR loss
-	Mode 5: Heating dominant w/o HR loss

This section determines the operational mode based on the load requirements and operational conditions:

a. Calculate the Loading Index LI_1 satisfying I/U cooling load (Refer to Engineering Reference V8.4+: Step 2c.4 in the VRF-FluidTCtrl-HP model for more details.)

b. Calculate the Loading Index LI_2 satisfying I/U heating load (Refer to Engineering Reference V8.4+: Step 2h.4 in the VRF-FluidTCtrl-HP model for more details.)

c. If LI_1 <= LI_2, the system operates at Mode 5

d. If LI_1 > LI_2 and Te' < To - 5, the system operates at Mode 2

e. If LI_1 > LI_2 and Te' >= To - 5, the system operates at Mode 3 or 4 (these two modes can be handled by one set of algorithms)


##### Step 7-A: O/U operation analysis at Mode 5

If Te' < To - 5, perform the following procedures:

a. Select the compressor speed corresponding to LI_2

b. Calculate the compressor power corresponding to LI_2 and the previously obtained Tc and Te'
(Refer to Engineering Reference V8.4+: Step 2c.6 in the VRF-FluidTCtrl-HP model for more details.)

c. Calculate the evaporative capacity (Cap_tot_evap) provided by the compressor at LI_2 and the previously obtained Tc and Te'
(Refer to Engineering Reference V8.4+: Step 2c.4 in the VRF-FluidTCtrl-HP model for more details.)

d. Calculate the O/U evaporator load (Cap_ou_evap) based on system-level heat balance

e. Obtain the O/U fan flow rate (m_air_evap) corresponding to Cap_ou_evap, and thus the fan power
(Refer to Engineering Reference V8.4+: Step 2c.3 in the VRF-FluidTCtrl-HP model for more details.)

If Te' >= To - 5, perform the following procedures:

a. Select the compressor speed corresponding to LI_1

b. Perform iterations between step b-i to identify the compressor Loading Index and power consumption.

c. Initialized compressor power (Ncomp_ini)

c.1 For the 1st iteration step, calculate Ncomp = f_pow_comp(Tc, Tout – 5, LI_2)

c.2 For the following iteration steps, update Ncomp = (Ncomp_ini + Ncomp_new)/2

d. Calculate the O/U evaporator load (Cap_ou_evap) based on system-level heat balance

e. Obtain the O/U evaporating temperature Te' level using Cap_ou_evap and the rated air flow rate
(Refer to Engineering Reference V8.4+: Step 2c.3 in the VRF-FluidTCtrl-HP model for more details.)

f. Update Te level and I/U evaporator side piping loss, corresponding to Te' update

g. Identify the compressor Loading Index LI_new to provide sufficient evaporative capacity (Cap_tot_evap) at updated Te' level
(Refer to Engineering Reference V8.4+: Step 2c.4 in the VRF-FluidTCtrl-HP model for more details.)

h. Calculate the compressor power (Ncomp_new) corresponding to LI_new and the updated Te'
(Refer to Engineering Reference V8.4+: Step 2c.6 in the VRF-FluidTCtrl-HP model for more details.)

i. Compare Ncomp_new and Ncomp_ini. Start a new round of iteration if the difference is greater than the tolerance.

##### Step 7-B: O/U operation analysis at Mode 2

a. Select the compressor speed corresponding to LI_1

b. Calculate the compressor power corresponding to LI_1 and the previously obtained Tc and Te'
(Refer to Engineering Reference V8.4+: Step 2c.6 in the VRF-FluidTCtrl-HP model for more details.)

c. Calculate the evaporative capacity (Cap_tot_evap) provided by the compressor at LI_1 and the previously obtained Tc and Te'
(Refer to Engineering Reference V8.4+: Step 2c.4 in the VRF-FluidTCtrl-HP model for more details.)

d. Calculate the O/U condenser load (Cap_ou_cond) based on system-level heat balance

e. Obtain the O/U fan flow rate (m_air_cond) corresponding to Cap_ou_cond, and thus the fan power

##### Step 7-C: O/U operation analysis at Mode 3 or 4

a. Select the compressor speed corresponding to LI_1

b. Perform iterations between step b-e to identify the updated Te' level within the range of To-5 and the original Te'.

c. Calculate the evaporative capacity (Cap_tot_evap) provided by the compressor at LI_1 and the previously obtained Tc and assumed Te'
(Refer to Engineering Reference V8.4+: Step 2c.4 in the VRF-FluidTCtrl-HP model for more details.)

d. Calculate the O/U evaporator load (Cap_tot_evap) at assumed Te' level and rated fan flow rate
(Refer to Engineering Reference V8.4+: Step 2h.3 in the VRF-FluidTCtrl-HP model for more details.)

e. Perform iterations to identify the updated Te' level to ensure the heat balance for the b. and c. calculations

f. Update Te level and I/U evaporator side piping loss, corresponding to Te' update

g. Calculate the compressor power corresponding to LI_1 and the updated Te'
(Refer to Engineering Reference V8.4+: Step 2c.6 in the VRF-FluidTCtrl-HP model for more details.)

h. Calculate the O/U condenser load (Cap_ou_cond) based on system-level heat balance

i. Obtain the O/U fan flow rate (m_air_cond) corresponding to Cap_ou_cond, and thus the fan power

##### Step 8: Modify I/U operational parameters for capacity adjustments
The air flow rate and SH/SC value of each indoor unit can be manipulated to adjust the cooling/heating capacity.
(Refer to Engineering Reference V8.4+: Step 3c and 3h in the VRF-FluidTCtrl-HP model for more details.)


## Testing/Validation/Data Sources ##

The model is validated with Daikin laboratory measurement data on a typical 3-pipe VRF Heat Recovery Multi-Split System. The system employs two inverter controlled compressors and one fixed speed compressor, which can work jointly to vary the compressor operation mode for a variety of operational conditions. Five terminal units are installed to create numerous heating/cooling load combinations. Comprehensive measurement data was collected for 14 static condition cases, covering the cooling only, heating only mode, and mostly the simultaneous heating and cooling mode. The sensible and latent load (latent capacity only used in cooling mode) in each indoor unit are calculated using the measurement data, and then given to the simulation as the zone load using the schedule object and other equipment object. The simulated compressor performance is then compared with the measured data. As shown in Figure 5, the simulated compressor speed and power using the new VRF-HR model can present a satisfactory match with the measured data throughout all the operational modes at sub-hour levels.

![](Validation_CompPower.png)
(a) Compressor power (sub-hourly)
![](Validation_CompSpeed.png)
(b) Compressor speed (sub-hourly)
Figure 5. Comparison of the measured and simulated compressor data under various operational modes


## Input Output Reference Documentation ##

To be developed.


## IDD Objects (New) ##

We propose to create a new IDD object named "AirConditioner:VariableRefrigerantFlow:FluidTemperatureControl:HR", which is an extension of the existing "AirConditioner:VariableRefrigerantFlow:FluidTemperatureControl" object designed for the VRF-FluidTCtrl-HP model.


## IDD Objects (Revised) ##

We propose to rename the existing object "AirConditioner:VariableRefrigerantFlow:FluidTemperatureControl" as "AirConditioner:VariableRefrigerantFlow:FluidTemperatureControl:HP", in order to distinguish from the proposed "AirConditioner:VariableRefrigerantFlow:FluidTemperatureControl:HR" object.

## Outputs Description ##
All the outputs designed for the VRF-FluidTCtrl-HP model apply for the proposed VRF-FluidTCtrl-HR model.

Newly added for the HR model include:
- Simultaneous Cooling and Heating Efficiency (SCHE). The ratio of the total capacity of the system (heating and cooling capacity) to the effective power when operating in the heat recovery mode.


## Engineering Reference ##

Currently section "17.6.2 Variable Refrigerant Flow Heat Pump Model (Physics Based Model)" only describes the HP model. It will be enriched to include the HR model.


## Example File and Transition Changes ##

The existing example file "VariableRefrigerantFlow_FluidTCtrl_5Zone.idf" is used to illustrate the simulation of VRF-HP system. It will be updated because of IDD updates. It will be renamed as "VariableRefrigerantFlow_FluidTCtrl_HP.idf" to distinguish from the file for the HR system.

A new example file "VariableRefrigerantFlow_FluidTCtrl_HR.idf" will be provided to illustrate the simulation of VRF-HR system.


## References ##

Tianzhen Hong, Kaiyu Sun, Rongpeng Zhang, Oren Schetrit. Modeling, Field Test and Simulation of Energy Performance of Daikin VRF-S Systems in California Houses. LBNL Report, March 2015.

Handbook of Compact Heat Exchangers. Yu Sesimo, Masao Fujii. Tokyo, Japan, 1992.