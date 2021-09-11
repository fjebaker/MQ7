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

![](https://github.com/fjebaker/MQ7/blob/master/equations/eq1.gif)

**NB:** the plot in the data sheet has reversed axes.

The reference resistance is defined by the constraint

![](https://github.com/fjebaker/MQ7/blob/master/equations/constraint1.gif)


Using this point, and `PPM(0.4) ~ 400`, one can calculate the coefficients of fit with

![](https://github.com/fjebaker/MQ7/blob/master/equations/eq2.gif)

![](https://github.com/fjebaker/MQ7/blob/master/equations/eq3.gif)
 

or

![](https://github.com/fjebaker/MQ7/blob/master/equations/val1.gif)

![](https://github.com/fjebaker/MQ7/blob/master/equations/val2.gif)

which are defined in `MQ7.h`. The library then calibrates `R0` by continuously recalculating the value every second for `CALIBRATION_SECONDS` (TODO: until delta based condition met). As such, **this calibration function must be called in `setup()`** to avoid faulty data.

The `R0` is calculated by assuming clean air (approx 10 ppm of carbon monoxide) -- it is therefore best to perform this calibration outside. You can read in the value calculated for `R0` using the `getR0` function.

Alternatively, as the data sheet recommends, the device should be calibrated after ambient on-time of 48 hours -- thus, it would be recommended for accurate use, to leave the device on for some hours in clean air, and then call `calibrate()` and `getR0` to read in an accurate calibration value (though in practice I do not know if this mitigates errors enough to be worth the effort). 

In `MQ7.h`, uncomment the line
```cpp
#define _R0 0.025	
```
and put in your `R0` value. Doing this will override the `calibrate()` method to do nothing, so there would be no need to call it under these conditions.

## References

- [MQ7 Data Sheet](https://www.parallax.com/sites/default/files/downloads/605-00007-MQ-7-Datasheet.pdf)
