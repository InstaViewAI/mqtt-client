#### MQTT Publisher Binaries

This folder contains binaries for various operating systems that can be used to simulate wake-up messages for a target device.

#### Binary Usage:

***Download*** : Download the appropriate binary for your operating system from this folder.
***Execute***: Run the binary from your command line interface. You may need to provide additional ***arguments*** depending on the specific binary and your desired configuration.
#### Example:

./[Binary Name] [options] 

#### Binary Options:
**clientid**: client id used by the device   
**username**: device id  
***password***: mcuaccess token from sdk
***thingname***: thingname from sdk
***deviceID***: deviceid for the device

#### Result
The successful execution will publish a message on the topic intended for the wakeup

Ensure you have the necessary permissions to execute the binary.
Disclaimer:
These binaries are provided as-is and without warranty. Use them at your own risk.
