# MQ7 carbon monoxide detector library for Arduino

## Features
This library implements an MQ7 class with the following features:

- device calibration
- read carbon monoxide parts per million (ppm)


## Quick-start guide
After downloading the library, load the `MQ7_Example.ino` example sketch onto your device, with 

- analog out connected to A2
- GND connected to GND
- Vin connected to 5V

Upload the sketch, and then monitor serial. The example sketch will calibrate the device over 15 seconds, and then print the carbon monoxide ppm once every second to the serial console.

## Detailed guide
In the MQ7 Data sheet is provided the typical behaviour of the device. The carbon monoxide ppm is a function of the surface resistance divided by a reference resistance, following the general exponential equation 

<img src="https://latex.codecogs.com/gif.latex?PPM\left(\frac{R_s}{R_0}\right) = A_0 \left(\frac{R_s}{R_0}\right) ^ {A_1}" />

**NB:** the plot in the data sheet has reversed axes.

The reference resistance is defined by the constraint

<img src="https://latex.codecogs.com/gif.latex?PPM(1) = 100" />

Using this point, and `PPM(0.4) ~ 400`, one can calculate the coefficients of fit with

<img src="https://latex.codecogs.com/gif.latex?A_1 = \frac{\text{ln}(PPM_2) - \text{ln}(PPM_1) }{\text{ln}(R_s/R_{0,2}) - \text{ln}(R_s/R_{0,1})" />

<img src="https://latex.codecogs.com/gif.latex?A_0 = \text{exp}\{ -A_1 \times \left( \text{ln}(PPM_2) - A_1 \text{ln}(R_s/R_{0,2}) \right) \}" />

Using just two points from the plot, I calculated

<img src="https://latex.codecogs.com/gif.latex?A_1 = -1.513" />

<img src="https://latex.codecogs.com/gif.latex?A_0 = 100.0" />

Which are defined in `MQ7.h`. The library then calibrates `R0` by continuously recalculating the value every seconds for `CALIBRATION_SECONDS` (TODO: until delta based condition met). As such, **this calibration function must be called in `setup()`** to avoid faulty data.

The `R0` is calculated by assuming clean air (approx 10 ppm of carbon monoxide) -- it is therefore best to perform this calibration outside. You can read in the value calculated for `R0` using the `getR0` function.

Alternatively, as the data sheet recommends, the device should be calibrated after ambient on-time of 48 hours -- thus, I would recommend for accurate use, to leave the device on for some hours in clean air, and then call `calibrate()` and `getR0` to read in an accurate calibration value. In `MQ7.h`, uncomment the line
```cpp
#define _R0 0.025	
```
and put in your `R0` value. Doing this will override the `calibration()` method to do nothing, so there would be no need to call it under these conditions.

## References

- [MQ7 Data Sheet](https://www.parallax.com/sites/default/files/downloads/605-00007-MQ-7-Datasheet.pdf)
