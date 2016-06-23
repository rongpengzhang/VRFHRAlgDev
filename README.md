
## Comments and Reply ##

**Comment**

Lawrence Scheier, Jun 6 

The approach you are proposing seems sound and, just as important, easy to input but I have two basic concerns:

1) Effect on Sizing: You state: "the fault model will only be applied at real weather simulations instead of the sizing and warm-up simulations". Would the faults be applied during the "Do HVAC Sizing Simulation for Sizing Periods" simulation? I am not sure how this would effect Brent's coincident sizing calcs. It seems like there would be a potential for the coincident plant size to be larger than the sum-of-the-peaks plant size since the faults would not be applied during the normal plant sizing. This might be a good thing or a bad thing. Not sure.

2) Effect on plant/airside convergence: it seems like these operational faults could very easily cause the plant/airside controllers to totally lose control at some point, causing both plant temperatures, airside and space temperatures to oscillate wildly (and to unrealistic extremes). This happens often enough under normal circumstances (or bad user input) without adding operational faults to the mix. I can foresee a lot of situations where only the first few day's results are valid and the rest of the year becomes nonsensical because the mathematics go haywire. This is why I am not enamored with operational faults, in general. I think our models too sensitive in many cases to be purposefully throwing a wrench into the works. But perhaps testing will show me to be wrong on this count. 

**Reply:**

1) Effect on Sizing
The proposed models aim to simulate and evaluate the faults occurred at the "operation" phase. We assume that these faults don't exist during the "design" phase, and therefore, the fault model should have no effect on any sizing related simulations, including the traditional sizing calculations and the coincident sizing calculations. 

2) Effect on plant/airside convergence
For most cases, the chiller faults should only affect the plant loop performance and have no impact on the air loop performance or zone temperatures. Although the chiller supply temperature and chiller PLR may differ from the fault-free cases, we think the plant controls (e.g., bypass control) should be able to ensure the correct heat transfer to the air side. We think the faults will lead to a situation that is similar to that in an over-sized or under-sized system. Since the over-sized and under-sized situations can be well handled by the current plant/airside controllers, we think the faulty-sensor cases should also be well handled and hopefully don't influence the plant/airside convergence. We will keep an eye on the convergence during the test. In addition, we will try to avoid any unrealistic extremes, e.g., set a limit for the sensor offset.
