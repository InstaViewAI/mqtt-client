#### MQTT Subscriber Binaries

This folder contains binaries for various operating systems that can be used to view message being send on the wakeup topic

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

#### Result
The successful execution will list messages being send on the wakeup topic
Ensure you have the necessary permissions to execute the binary.
Disclaimer:
These binaries are provided as-is and without warranty. Use them at your own risk.
