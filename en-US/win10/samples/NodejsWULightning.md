---
layout: default
title: NodejsWULightning
permalink: /en-US/win10/samples/NodejsWULightning.htm
lang: en-US
---

# Blinky Node.js (Windows Universal) Sample with Lightning

{% include VerifiedVersion.md %}

In this sample, we will use the [Lightning GPIO provider]({{site.baseurl}}/{{page.lang}}/win10/LightningProviders.htm) to blink an LED attached to a Raspberry Pi 2. 
It also includes steps to reference a custom winmd file in your Node.js project. This sample is based on the [Blinky sample]({{site.baseurl}}/{{page.lang}}/win10/samples/NodejsWUBlinky.htm) 
and shares the same setup steps as well as most of the code. The key difference is setting the Lightning provider as the default controllers provider.


### Set up your PC
* Install Windows 10 [with November update](http://windows.microsoft.com/en-us/windows-10/windows-update-faq).
* Install Visual Studio 2015 Update 1.
* Install the latest Node.js Tools for Windows IoT from [here](http://aka.ms/ntvsiotlatest).


### Set up your hardware
* The setup for this sample is the same as the C# 'Blinky' [sample]({{site.baseurl}}/{{page.lang}}/win10/samples/Blinky.htm).
* Follow the steps on [this page]({{site.baseurl}}/{{page.lang}}/win10/LightningSetup.htm) to set up Lightning on your Raspberry Pi 2.


### Create a new Node.js (Windows Universal) project
Start Visual Studio 2015 and create a new project (File \| New Project...). In the `New Project` dialog, navigate to `Node.js` as shown below (in the left pane in the dialog: Templates \| JavaScript \| Node.js).
Use the `Basic Node.js Web Server (Windows Universal)` template.

When the project has been created, open up server.js and replace the existing code with the code shown below:

<UL>
{% highlight JavaScript %}
var http = require('http');
var uwp = require('uwp');

// Inject 'Windows' and 'Microsoft' namespaces to global
uwp.projectNamespace('Windows');
uwp.projectNamespace('Microsoft');

// Check if Lightning is enabled and set the Lightning provider as the default provider
if (Microsoft.IoT.Lightning.Providers.LightningProvider.isLightningEnabled) {
  Windows.Devices.LowLevelDevicesController.defaultProvider = Microsoft.IoT.Lightning.Providers.LightningProvider.getAggregateProvider();
}

var gpioController = Windows.Devices.Gpio.GpioController.getDefault();

// Open pin 5
pin = gpioController.openPin(5);

// Configure pin for output
pin.setDriveMode(Windows.Devices.Gpio.GpioPinDriveMode.output);

// Write initial 'high' value to pin
pin.write(Windows.Devices.Gpio.GpioPinValue.high);

setInterval(function () {
  // Toggle LED on/off every 1 second
  if (pin.read() == Windows.Devices.Gpio.GpioPinValue.high) {
    pin.write(Windows.Devices.Gpio.GpioPinValue.low);
  } else {
    pin.write(Windows.Devices.Gpio.GpioPinValue.high);
  }
}, 1000);

uwp.close();
{% endhighlight %}
</UL>

Here's what the code above is doing:

* We use the [node-uwp](https://www.npmjs.com/package/uwp) npm package (included in your project by default) to allow the code to use UWP APIs (within Windows and Microsoft namespaces).
* Check if Lightning is enabled and set it as the default provider.
* `GpioController.getDefault()` is called to get the GPIO controller.
* Then we attempt to open the pin by calling `GpioController.openPin()` with the LED pin value.
* Once we have the `pin`, we set it to be off (high) by default using the `GpioController.write()` function.
* Every 1000 milliseconds (1 second), the value of the LED is checked and then set to the opposite of the current value.

### Add Microsoft.IoT.Lightning.Providers library to your project
* Clone the bus provider repository on your PC with `git clone https://github.com/ms-iot/BusProviders`.
* In the Solution Explorer in your project, right click on the solution node, select 'Add->Existing Project...', then add \<Bus Provider Clone Root\>\Microsoft.IoT.Lightning.Providers\Providers\Microsoft.Iot.Lightning.Providers.vcxproj.
* Build the solution.
* In the Solution Explorer, right click on your Node.js project (with server.js), select 'Add->Existing Item...'. then add the following files:
  * \<Project Root\>\ARM\Debug\Microsoft.Iot.Lightning.Providers\Microsoft.IoT.Lightning.Providers.dll
  * \<Project Root\>\ARM\Debug\Microsoft.Iot.Lightning.Providers\Microsoft.IoT.Lightning.Providers.winmd
* Open the Package.appxmanifest file. Add the capabilities below:

<UL>
{% highlight XML %}
    <iot:Capability Name="lowLevelDevices" />
    <DeviceCapability Name="109b86ad-f53d-4b76-aa5f-821e2ddf2141"/>
{% endhighlight %}
</UL>

* Then add the extension below:

<UL>
{% highlight XML %}
    <Extension Category="windows.activatableClass.inProcessServer">
      <InProcessServer>
        <Path>Microsoft.IoT.Lightning.Providers.dll</Path>
        <ActivatableClass ActivatableClassId="Microsoft.IoT.Lightning.Providers.LightningPwmProvider" ThreadingModel="both" />
        <ActivatableClass ActivatableClassId="Microsoft.IoT.Lightning.Providers.LightningGpioProvider" ThreadingModel="both" />
        <ActivatableClass ActivatableClassId="Microsoft.IoT.Lightning.Providers.LightningSpiProvider" ThreadingModel="both" />
        <ActivatableClass ActivatableClassId="Microsoft.IoT.Lightning.Providers.LightningI2cProvider" ThreadingModel="both" />
        <ActivatableClass ActivatableClassId="Microsoft.IoT.Lightning.Providers.LightningProvider" ThreadingModel="both" />
        <ActivatableClass ActivatableClassId="Microsoft.IoT.Lightning.Providers.LightningAdcProvider" ThreadingModel="both" />
      </InProcessServer>
    </Extension>
{% endhighlight %}
</UL>

* Build the solution.


### Deploy the server to your Windows IoT Core device
* Go to the Project menu and select '&lt;Your project name&gt; Properties' (You could also right-click on the project node in solution explorer to access Properties). Enter the IP Address in the Remote Machine text box. If you're building for Minnowboard Max, select `x86` in the dropdown.  If you're building for Raspberry Pi 2 or 3, select `ARM`.

* Now we're ready to deploy to the remote Windows IoT Core device. Simply press F5 (or select Debug \| Start Debugging) to start the app.


### GitHub
* NTVS IoT Extension source code: [https://github.com/ms-iot/ntvsiot](https://github.com/ms-iot/ntvsiot)
* Node.js UWP wrapper source code: [https://github.com/ms-iot/node-uwp-wrapper](https://github.com/ms-iot/node-uwp-wrapper)
