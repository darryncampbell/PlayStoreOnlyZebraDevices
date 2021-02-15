*Please be aware that this application / sample is provided as-is for demonstration purposes without any guarantee of support*
=========================================================

# Play Store Only Zebra Devices
Sample application showing how to target only Zebra devices, so the Play Store will only consider your application compatible with Zebra devices

-------------------------------------------------------

## Targeting Zebra devices with the Play Store device catalogue

When distributing your application through Google’s Play Store, which devices it runs on is a key consideration, especially in Enterprise.  The way the Play Store manages an application's supported devices is through the [Device catalogue](https://play.google.com/console/about/devicecatalog/) although that catalogue (at the time of writing) contains nearly 18,000 entries.  In the consumer sphere ensuring your application runs on the widest range of devices is non-trivial but essential to ensure success, so developers rely on Play Store app stats, crashlytics, A/B testing, device farms and innumerable other techniques.

Enterprise developers are different and will typically target fewer than 20 device models (often just 1) so have different priorities and considerations.

![Device Catalogue](https://raw.githubusercontent.com/darryncampbell/PlayStoreOnlyZebraDevices/main/screenshots/01.png)

### The Managed Play Store

For those not familiar: Google’s Play Store has a dedicated set of capabilities targeted towards Enterprise use cases known as the ‘[Managed Google Play](https://support.google.com/googleplay/work/answer/6138458?hl=en)’.  It is through the managed Play Store that Enterprises control which applications are installed on devices using an EMM, regardless of whether the device is fully owned (typical with Zebra devices) or uses the Android Work Profile on personal devices.  Through managed Google Play, organizations can also publish [private applications](https://support.google.com/googleplay/work/answer/6145139) not otherwise available through the Play Store.

Managed Google Play and the standard Play Store share much of the same infrastructure, including the [Device catalogue](https://play.google.com/console/about/devicecatalog/) meaning you can use the device catalogue to selectively include or exclude supported devices regardless of which Play Store variant you use.

### Manually exclude devices your application does not support

Every application will not run on every device – e.g. some applications require GPS but not all devices have this capability, some applications require a camera but not all devices have a camera etc.  Your application will specify a minimum target SDK but not all devices will support that Android version.  This is the difference between “Supported” and “Excluded”.  By default, your application will be available to all devices whose hardware and OS match your apps criteria which will likely be thousands of device models.

If you want your application to only be available on a subset of these devices, for example you have a ruggedized enterprise deployment that includes both Zebra and another manufacturer’s devices, you will need to **manually** exclude the devices through the Play Store UI.  The Play Store is not set up to bulk-exclude devices and so you are required to painfully exclude them on a page-by-page basis (meaning you can only exclude around 500 at a time).  There are some [Stack Overflow](https://stackoverflow.com/questions/9510649/how-to-restrict-android-app-to-specific-device-make/18638583#18638583) posts describing easier ways to exclude multiple devices but these seem to depend on the page HTML & CSS and given the Play Store UX changes very frequently I cannot say that these techniques will work long-term, though they are worth a try.

![Bulk exclude devices](https://raw.githubusercontent.com/darryncampbell/PlayStoreOnlyZebraDevices/main/screenshots/02.png)

### Writing an app that is only ‘supported’ on Zebra devices

If you only plan on only targeting Zebra devices with your application, you can take advantage of the existing Play Store mechanisms for filtering supported devices based on hardware and software requirements.

If you look at any device in the catalogue you will see the System features and Shared libraries that it supports, for example there are 70 shared libraries supported by the MC3300x:

![MC3300x shared libraries](https://raw.githubusercontent.com/darryncampbell/PlayStoreOnlyZebraDevices/main/screenshots/03.png)

Looking at this list, many of them are standard libraries but others are unique to Zebra devices, i.e. anything that starts with `com.symbol.*`.  Exactly what each of these libraries do is undocumented, subject to change and in some cases not obvious but the functionality of a handful of these libraries is self-evident.

By taking a Zebra-specific shared library and ensuring it is required by our application we can ensure that only Zebra devices are shown as "supported" for our app in the device catalogue.  For example, the underlying functionality for the EMDK API is provided by the `com.symbol.emdk` service.  If your application specifies that this library is required in its manifest (even if it does not use the EMDK) it will still only be "supported" by devices that contain this library:

```xml
<uses-library
  android:name="com.symbol.emdk"
  android:required="true"/>
```

A sample of how to do this is available from https://github.com/darryncampbell/PlayStoreOnlyZebraDevices.  The application itself is also available on the [Play Store](https://play.google.com/store/apps/details?id=com.darryncampbell.playstore_onlyzebradevices).  

Notice how the application will only show as installable on Zebra devices:

![TC52x Play Store](https://raw.githubusercontent.com/darryncampbell/PlayStoreOnlyZebraDevices/main/screenshots/04.jpg)

The application as it appears in the Play Store on a Zebra TC52x

![Pixel 4 Play Store](https://raw.githubusercontent.com/darryncampbell/PlayStoreOnlyZebraDevices/main/screenshots/05.png)

The application as it appears in the Play Store on a Google Pixel 4

### Zebra devices in the catalogue: Why do some devices have multiple models?

If you look at the device catalogue and filter on Zebra devices you will see that some devices have multiple models associated with them, for example the TC20 can be expanded into two models but the TC20KB is shown as a separate device.  The TC51 has two available models, the TC51 and TC51HC:

![Multiple models per device](https://raw.githubusercontent.com/darryncampbell/PlayStoreOnlyZebraDevices/main/screenshots/06.png)

I mentioned earlier how each model will have an associated set of System features and Shared libraries and this explains why some devices have multiple models.  Zebra offers multiple SKUs for all our devices and where the hardware features differ between these SKUs, separate models are created in the Play Store when we submit to Google CTS testing.  

To take the TC51 / TC51HC example shown in the screenshot, the TC51HC has an additional system feature, `android.hardware.camera.front`, because the healthcare variant of the TC51 (TC51HC) features a front facing camera that is lacking from the non-healthcare TC51 device.  It also explains why the TC52 only appears as a single device, because both the TC52 and TC52HC both feature a front facing camera.

The camera will not always be the cause for multiple models, for example the difference in ET51 tablet models is because we are part way through the AR core certification for our tablet line ([only the ‘s’ variants currently support ARCore](https://developers.google.com/ar/discover/supported-devices)) but in all cases the reason multiple models are shown will be hardware related.

Where multiple models are shown it is possible to include or exclude individual models, either manually from the device catalogue or by specifying `<uses-feature/>` in your application manifest, giving you full control over where your application will run.

### Conclusion

In conclusion, it is possible to fully control where your application gets deployed by using the Play Store device catalogue which is particularly useful for our ISV partners who need to control where there enterprise apps will be deployed.
