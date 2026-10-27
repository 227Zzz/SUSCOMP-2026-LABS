# Estimating Embodied Emissions
Note: The emission factor of 0.235 kg CO2e/kWh for Slovenia is based on the data provided in the lab instructions (IJS Center for Energy Efficiency).

## Asset 1: iPhone 16 (128GB)

### Manufacturing Footprint
* 44.8 kg CO₂e, 80% of the iPhone 16's total carbon footprint
* Source: https://www.apple.com/environment/pdf/products/iphone/iPhone_16_and_iPhone_16_Plus_PER_Sept2024.pdf
* Reason for choosing: This is the official Product Environmental Report published by Apple, providing reliable carbon footprint data for the device.

### Use Phase CO2 Emissions
* Yearly energy consumption: 7.014 kWh
* Emission factor: 0.235 kg CO2e/kWh

* Yearly CO2 emissions:
  $$7.014 \times 0.235 = 1.64829 kg CO2e$$

### Comparison of Use and Manufacturing Emissions

| Time span | Energy consumption | Use phase emissions | Manufacturing emissions (total) | Manufacturing emissions per year |
| :--- | ---: | ---: | ---: | ---: |
| 1 year | 7.014 kWh | 1.648 kg CO2e | 44.80 kg CO2e | 44.80 kg CO2e |
| 3 years | 21.042 kWh | 4.945 kg CO2e | 44.80 kg CO2e | 14.93 kg CO2e |
| 5 years | 35.070 kWh | 8.241 kg CO2e | 44.80 kg CO2e | 8.96 kg CO2e |
| 10 years | 70.140 kWh | 16.483 kg CO2e | 44.80 kg CO2e | 4.48 kg CO2e |

---

## Asset 2: Macbook Air (M1, 256GB)

### Manufacturing Footprint
* 122.36 kg CO₂e, 76% of the Macbook Air's total carbon footprint
* Source: https://www.apple.com/environment/pdf/products/notebooks/13-inch_MacBookAir_PER_Nov2020.pdf
* Reason for choosing: This is the official Product Environmental Report published by Apple, providing reliable carbon footprint data for the device.

### Use Phase CO2 Emissions
* Yearly energy consumption: 31.4196 kWh
* Emission factor: 0.235 kg CO2e/kWh

* Yearly CO2 emissions:
  $$31.4196 \times 0.235 = 7.383606 kg CO2e$$

### Comparison of Use and Manufacturing Emissions

| Time span | Energy consumption | Use phase emissions | Manufacturing emissions (total) | Manufacturing emissions per year |
| :--- | ---: | ---: | ---: | ---: |
| 1 year | 31.420 kWh | 7.384 kg CO2e | 122.36 kg CO2e | 122.36 kg CO2e |
| 3 years | 94.259 kWh | 22.151 kg CO2e | 122.36 kg CO2e | 40.79 kg CO2e |
| 5 years | 157.098 kWh | 36.918 kg CO2e | 122.36 kg CO2e | 24.47 kg CO2e |
| 10 years | 314.196 kWh | 73.836 kg CO2e | 122.36 kg CO2e | 12.24 kg CO2e |

---

## Asset 3: Wi-Fi Router (Reference: Swisscom Internet-Box 4)

### Manufacturing Footprint
* 50.3 kg CO₂e
* Source: https://www.swisscom.ch/content/dam/assets/b2c/products/internet/help/pdf/pcf_factsheet_ib4.pdf
* Reason for choosing: I could not find data for my actual router, I selected a reference model with similar power consumption. The data is also provided in an official product carbon footprint factsheet published by the provider.

### Use Phase CO2 Emissions
* Yearly energy consumption (from Week 2): 75.92 kWh
* Emission factor: 0.235 kg CO2e/kWh
  
* Yearly CO2 emissions:
  $$75.92 \times 0.235 = 17.8412 \ kg \ CO2e$$

### Comparison of Use and Manufacturing Emissions

| Time span | Energy consumption | Use phase emissions | Manufacturing emissions (total) | Manufacturing emissions per year |
| :--- | ---: | ---: | ---: | ---: |
| 1 year | 75.920 kWh | 17.841 kg CO2e | 50.30 kg CO2e | 50.30 kg CO2e |
| 3 years | 227.760 kWh | 53.524 kg CO2e | 50.30 kg CO2e | 16.77 kg CO2e |
| 5 years | 379.600 kWh | 89.206 kg CO2e | 50.30 kg CO2e | 10.06 kg CO2e |
| 10 years | 759.200 kWh | 178.412 kg CO2e | 50.30 kg CO2e | 5.03 kg CO2e |

**P.S.** I could not find manufacturing carbon footprint data for my actual Wi-Fi router. Therefore, I used the Swisscom Internet-Box 4 as a reference model because its reported power consumption (9.6 W during normal operation and 8.3 W in standby mode) is close to the estimated values used in Week 2 (10 W active and 8 W idle). The energy consumption calculations remain based on my Week 2 estimates.
* Power consumption source: https://www.swisscom.ch/en/residential/help/apps-and-services/energy-consumption.html

---

## Reflections

I found that manufacturing emissions are much higher than use-phase emissions for the iPhone 16 and MacBook Air, even after 10 years of use. However, the Wi-Fi router shows a different pattern, with its cumulative use-phase emissions exceeding manufacturing emissions after about three years. This suggests that the relative importance of manufacturing and electricity consumption depends on the type of device and how long it is used.