<!--yml
category: 未分类
date: 2024-07-01 18:17:59
-->

# Android 2.x Sensor Simulator : ezyang’s blog

> 来源：[http://blog.ezyang.com/2011/02/android-2-x-sensor-simulator/](http://blog.ezyang.com/2011/02/android-2-x-sensor-simulator/)

## Android 2.x Sensor Simulator

OpenIntents has a nifty application called [SensorSimulator](http://code.google.com/p/openintents/wiki/SensorSimulator) which allows you feed an Android application accelerometer, orientation and temperature sensor data. Unfortunately, it doesn't work well on the newer Android 2.x series of devices. In particular:

*   The mocked API presented to the user is different from the true API. This is due in part to the copies of the Sensor, SensorEvent and SensorEventHandler that the original code had in order to work around the fact that Android doesn't have public constructors for these classes,
*   Though the documentation claims “Whenever you are not connected to the simulator, you will get real device sensor data”, this is not actually the case: all of the code that interfaces with the real sensor system is commented out. So not only is are the APIs incompatible, you have to edit your code from one way to another when you want to vary testing. (The code also does a terrible job of handling the edge condition where you are not actually testing the application.)

Being rather displeased with this state of affairs, I decided to fix things up. With the power of Java reflection (cough cough) I switched the representation over to the true Android objects (effectively eliminating all overhead when the simulator is not connected.) Fortunately, Sensor and SensorEvent are small, data-oriented classes, so I don't think I stepped on the internal representation too much, though the code will probably break horribly with future versions of the Android SDK. Perhaps I should suggest to upstream that they should make their constructors public.

You can grab the code here: [SensorSimulator on Github](https://github.com/ezyang/SensorSimulator). Let me know if you find bugs; I've only tested on Froyo (Android 2.2).