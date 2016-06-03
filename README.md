Build an IoT Service in 1 Hour
Windows 10 IoT Core Hands-on Lab
========================================
This lab will help you get tiny devices connected to Microsoft Azure, and to implement great IoT solutions taking advantage of Microsoft Azure IoT Hubs and the Universal Windows Platform.

> This lab is stand-alone, but is used at Microsoft to accompany a presentation about Azure, Windows 10 IoT Core, and our IoT services. If you wish to follow this on your own, you are encouraged to do so. If not, consider attending a Microsoft-led IoT lab in your area.



In this lab you will use a [Raspberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) device with [Windows 10 Iot Core](http://ms-iot.github.io/content/en-US/Downloads.htm) and a [FEZ HAT](https://www.ghielectronics.com/catalog/product/500) sensor hat. Using a Windows 10 Universal Application, the sensors get the raw data and format it into a JSON string. That string is then shuttled off to the [Azure IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub/), where it gathers the data and you can communicate commands directly back to the device.

> **Note:** Although AMQP is the recommended approach, at the time this lab was written that protocol was not supported by the [Azure IoT Core SDK](https://github.com/Azure/azure-iot-sdks) for UWP (Universal Windows Platform) applications. However, it is expected to be implemented in a short time, since it's currently under development.  

IoT Workshop Architecture

![iot-workshop](Images/iotworkshop.PNG?raw=true)


This lab includes the following tasks:

1. [Setup](#Task1)
	1. [Software](#Task11)
	2. [Devices](#Task12)
	3. [Azure Account](#Task13)
	4. [Device Registration](#Task14)
2. [Creating a Universal App](#Task2)
	1. [Read FEZ HAT sensors](#Task21)
	2. [Send telemetry data to Azure IoT Hub](#Task22)
3. [Keep building up your project](#Task6)
	1. [Adding extra sensors](#Task55)
	2. [Sending commands to your devices](#Task44)
5. [Summary](#Summary)

<a name="Task1" />
## Pre-requisites 
The following sections are intended to setup your environment to be able to create and run your solutions with Windows 10 IoT Core.

<a name="Task11" />
### Setting up your Software
Your machine setup includes the following items already downloaded:

- Windows 10 (build 10240) or better

- Visual Studio 2015 with Update 1 [Community Edition](http://www.visualstudio.com/downloads/download-visual-studio-vs) is sufficient. Make sure to install Visual Studio Update 1 for the latest 10586 build of Windows IoT Core: https://www.visualstudio.com/en-us/news/vs2015-update1-vs.aspx

	> **NOTE:** If you choose to install a different edition of VS 2015, make sure to do a **Custom** install and select the checkbox **Universal Windows App Development Tools** -> **Tools and Windows SDK**.

- Windows IoT Core Project Templates. You can download them from [here](https://visualstudiogallery.msdn.microsoft.com/55b357e1-a533-43ad-82a5-a88ac4b01dec). Alternatively, the templates can be found by searching for Windows IoT Core Project Templates in the [Visual Studio Gallery](https://visualstudiogallery.msdn.microsoft.com/) or directly from Visual Studio in the Extension and Updates dialog (Tools > Extensions and Updates > Online).

- Make sure you’ve **enabled developer mode** in Windows 10 by following [these instructions](https://msdn.microsoft.com/library/windows/apps/xaml/dn706236.aspx).


### Download Azure Device Explorer
- To register your devices in the Azure IoT Hub Service and to monitor the communication between them you need to install the [Azure Device Explorer](https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/doc/how_to_use_device_explorer.md). Follow this link to download the SetupDeviceExplorer.msi file: https://github.com/Azure/azure-iot-sdks/releases.

![Azure IoT Hub Device Explorer](Images/iot_hub_explorer.PNG?raw=true)

### Download the IoT Core Dashboard
- The Windows 10 IoT Core Dashboard allows you to easily discover and access your tiny devices running Windows IoT Core 10.0.10586 version or later. You can download this dashboard from here: https://ms-iot.github.io/content/en-US/GetStarted.htm


<a name="Task12" />
## During the Lab

### Setting up your Devices

For this project, you have the following items

- [Raspberry Pi 2 Model B](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) with power supply and adaptor
- [GHI FEZ HAT](https://www.ghielectronics.com/catalog/product/500)
- Your PC running Windows 10 or later
- An Ethernet port on the PC, or an auto-crossover USB->Ethernet adapter like the [Belkin F4U047](http://www.amazon.com/Belkin-USB-Ethernet-Adapter-F4U047bt/dp/B00E9655LU/ref=sr_1_2).
- Standard Ethernet cable
- A 16GB Samsung SD Card with Windows 10 IoT Core already mounted.

To setup your devices perform the following steps:

1. Plug the **GHI FEZ HAT** into the **Raspberry Pi 2**.

	![fezhat-connected-to-raspberri-pi-2](Images/fezhat-connected-to-raspberri-pi-2.png?raw=true)

	_The FEZ hat connected to the Raspberry Pi 2 device_

2. Get your Windows 10 IoT Core SD Card and insert into the micro SD card on the Raspberry Pi device.  

3. You already have Windows IoT core image on the SD card for this lab

4. Connect the Raspberry Pi to a power supply and use the Ethernet cable to connect your device and your development PC. You can do it by plugging in one end of the spare Ethernet cable to the extra Ethernet port on your PC, and the other end of the cable to the Ethernet port on your IoT Core device. (Do this using an on-board port or an auto-crossover USB->Ethernet interface.)

	![windows-10-iot-core-fez-hat-hardware-setup](Images/windows-10-iot-core-fez-hat-hardware-setup.png)

5. Wait for the OS to boot.

6. Enable your device to internet share with the Raspberry Pi by following the instructions in the file **'Setup your device to Internet Share.pdf'**

### Find your Device

7. Run the **Windows 10 IoT Core Dashboard** on your development PC and note your Raspberry Pi IP address on the detected device [each device in this lab has a unqiue name, situated on the blue box].

	- Click the windows "**Start**" button
	- Type "**IoT**" to pull it up in the search results
	- You may want to right click on the program name and select "**Pin to Start**" to pin it to your start screen for easy access
	- Press **Enter** to run it

	![windows-iot-core-dashboard](Images/iot_core_dashboard.PNG?raw=true)

	> **NOTE:** If your device does not show up:
	> Go to Settings -> check 'Listen to eBootPinger boardcasts'
	> OR
	> Follow the "GetIPAddressFromHostName.docx" document for instructions on gaining your IP from your unique device name on the bright label>

### Trust Your Device

8. Launch an administrator PowerShell console on your local PC. The easiest way to do this is to type _powershell_ in the **Search the web and Windows** textbox near the Windows Start Menu. Windows will find **PowerShell** on your machine. Right-click the **Windows PowerShell** entry and select **Run as administrator**. The PS console will show.

	![Running Powershell as Administrator](Images/running-powershell-as-administrator.png?raw=true)

9. You may need to start the **WinRM** service on your desktop to enable remote connections. From the PS console type the following command:

	`net start WinRM`

10. From the PS console, type the following command, substituting '<_IP Address_>' with the IP value copied in prev:

	`Set-Item WSMan:\localhost\Client\TrustedHosts -Value <machine-name or IP Address>`

11.  Type **Y** and press **Enter** to confirm the change.

#### OPTIONAL: Web Portal

12. Check your RPI is connected to the internet, find IP address from the Windows IoT Core dashboard or via the 'GetIPAddressFromHostName.docx'

	![open-web-portal](Images/openwebportal.PNG)

13. Use login name: Administrator and password: p@ssw0rd
	
	![login-details](Images/login.PNG)

14. This will open an IoT Core dashboard

	![IoT-core-dashboard](Images/iotcorewebportal.PNG)


<a name="Task13" />
### Setting up your Azure Account
<Add information about Azure Passes>

#### Creating an IoT Hub

1. Enter the Azure portal, by browsing to http://portal.azure.com
2. Create a new IoT Hub. To do this, click **New** in the jumpbar, then click **Internet of Things**, then click **Azure IoT Hub**.

	![Creating a new IoT Hub](Images/creating-a-new-iot-hub.png?raw=true "Createing a new IoT Hub")

	_Creating a new IoT Hub_

3. Configure the **IoT hub** with the desired information:
 - Enter a **Name** for the hub e.g. _iot-sample_,
 - Select a **Pricing and scale tier** (_F1 Free_ tier is enough),
 - Create a new resource group, or select and existing one. For more information, see [Using resource groups to manage your Azure resources](https://azure.microsoft.com/en-us/documentation/articles/resource-group-portal/).
 - Select the **Region** such as _North Europe_ where the service will be located.

	![new iot hub settings](Images/new-iot-hub-settings.png?raw=true "New IoT Hub settings")

	_New IoT Hub Settings_

4. It can take a few minutes for the IoT hub to be created. Once it is ready, open the blade of the new IoT hub, take note of the URI and select the key icon at the top to access to the shared access policy settings:

	![IoT hub shared access policies](Images/iot-hub-shared-access-policies.png?raw=true)

5. Select the Shared access policy called **iothubowner**, and take note of the **Primary key** and **connection string** in the right blade.  You should copy these into a text file for future use. 

	![Get IoT Hub owner connection string](Images/get-iot-hub-owner-connection-string.png?raw=true)


<a name="Task14" />
### Registering your device
You must register your device in order to be able to send and receive information from the Azure IoT Hub. This is done by registering a [Device Identity](https://azure.microsoft.com/en-us/documentation/articles/iot-hub-devguide/#device-identity-registry) in the IoT Hub.

1. Open the Device Explorer app (C:\Program Files (x86)\Microsoft\DeviceExplorer\DeviceExplorer.exe) and fill the **IoT Hub Connection String** field with the connection string of the IoT Hub you created in previous steps and click on **Update**.

	![Configure Device Explorer](Images/configure-device-explorer.png?raw=true)

2. Go to the **Management** tab and click on the **Create** button. The Create Device popup will be displayed. Fill the **Device ID** field with a new Id for your device (_myFirstDevice_ for example) and click on Create:

	![Creating a Device Identity](Images/creating-a-device-identity.png?raw=true)

3. Once the device identity is created, it will be displayed in the grid. Right click on the identity you just created, select **Copy connection string for selected device** and take note of the value copied to your clipboard, since it will be required to connect your device with the IoT Hub.

	![Copying Device connection information](Images/copying-device-connection-information.png?raw=true)

	> **Note:** The device identities registration can be automated using the Azure IoT Hubs SDK. An example of how to do that can be found [here](https://azure.microsoft.com/en-us/documentation/articles/iot-hub-csharp-csharp-getstarted/#create-a-device-identity).


<a name="Task2" />
## Creating a Universal App
Now that the device is configured, you will see how to create an application to read the value of the FEZ HAT sensors, and then send those values to an Azure IoT Hub.

<a name="Task21" />


<a name="Task22" />
### Send telemetry data to the Azure IoT Hub

Now that you know how to read the FEZ HAT sensors data, you will send that information to an Azure IoT Hub. To do that, you will use an existing project located in the **Code\WindowsIoTCorePi2FezHat-IoTHubs\Code\WindowsIoTCorePi2FezHat\Begin** folder.

1. Open the Microsoft Visual Studio solution file located in the **Code\WindowsIoTCorePi2FezHat-IoTHubs\Code\WindowsIoTCorePi2FezHat\Begin** folder.

2. Before running the application, you must set the **Device connection** information. Go to the _MainPage_ method of the _MainPage.xaml.cs_ file and replace **IOT_CONNECTION_STRING** with your device connection string, obtained in previous steps using the Device Explorer app:

	````C#
	ctdHelper = new ConnectTheDotsHelper(iotDeviceConnectionString: "IOT_CONNECTION_STRING",
	    organization: "YOUR_ORGANIZATION_OR_SELF",
	    location: "YOUR_LOCATION",
	    sensorList: sensors);
	````

	![Copying Device connection information](Images/copying-device-connection-information.png?raw=true)

	> **Note:** An **Organization/School** and **Location** may also be provided. Those values will be part of the telemetry data message, and could be used to get a better classification of the data received.

3. Ensure that the target platform for the project is set to "**ARM**":

	![arm-target-platform](Images/arm-target-platform.png?raw=true)


4. To deploy the application to the Raspberry Pi, the device has to be on the same network as the development computer. To run the program, select **Remote device** in the _Debug Target_ dropdown list:

	![Deploy to Remote machine](Images/deploy-to-remote-machine.png?raw=true)

	_Deploying the application to a Remote Machine_

5. If a remote machine has not been selected before, the **Select Remote Connection** screen will be displayed:

	![Remote Connection](Images/remote-connection.png?raw=true)

	_Setting up the Remote Connection_

6. If the device is not auto-detected, the Raspberry Pi IP or name can be entered in the **Address** field. Otherwise, click the desired device. Change the **Authentication Mode** to **Universal (Unencrypted Protocol)** or none if unavaliable:

	![Set Authentication mode to Universal](Images/set-authentication-mode-to-universal.png?raw=true)

	_Setting the Authentication Mode_

7. If you want to change the registered remote device later it can be done in the project **Properties** page. Right-click the project name (_GHIElectronics.UAP.Examples.FEZHAT_) and select **Properties**. In the project Properties' page, select the **Debug** tab and enter the new device name or IP in the **Remote Machine** field.

	![Change Remote connection](Images/change-remote-connection.png?raw=true)

	_Changing the Remote Connection Settings_

	> **Note:** Clicking the **Find** button will display the **Remote Connection** screen.

8. Insert code for a sensor timer

	1. Instead of the **Button_Click** method (commented out in the code *in green*) find the comment "//ADD TIMER_TICK METHOD HERE" and add the code below:

		````C#
		private void Timer_Tick(object sender, object e)
		{
		    ConnectTheDotsSensor sensor = ctdHelper.sensors.Find(item => item.measurename == "Temperature");
		    sensor.value = counter++;
		    ctdHelper.SendSensorData(sensor);
		}
		````

	2. Now uncomment (remove the //) lines of code like below for the timer to be initiated:

		````C#
		//Button_Click(null, null);
		var timer = new DispatcherTimer();
		timer.Interval = TimeSpan.FromMilliseconds(500);
		timer.Tick += Timer_Tick;
		timer.Start();
		````

		Which will make the Timer tick twice a second.


9. Before adding real sensor information you can run this code to see how the device connects to your **Azure IoT Hub** and sends information. Run the application.

	![Debug Console output](Images/debug-console-output.png?raw=true)

	_Debugging in the Output Window_

10. After the app has been successfully deployed, it can start sending messages to the IoT Hub.

	The information being sent can be monitored using the Device Explorer application. Run the application and go to the **Data** tab and select the name of the device you want to monitor (_myFirstDevice_ in your case), then click on **Monitor**

	![Monitoring messages sent](Images/monitoring-messages-sent.png?raw=true)

	> **Note:** If the Device Explorer hub connection is not configured yet, you can follow the instructions explained in the [Registering your device](#Task14) section

11.  Now remove the timer created in that flow before you continue. A new timer will be created in the next steps replacing the previous one. Remove the **Timer_Tick** method you created before and delete the following lines from the **MainPage** constructor

	````C#
	var timer = new DispatcherTimer();
	timer.Interval = TimeSpan.FromMilliseconds(500);
	timer.Tick += Timer_Tick;
	timer.Start();
	````

12. Now that the device is connected to the Azure IoT Hub, add some real sensor information. First, you need to add a reference the FEZ HAT drivers. To do so, instead of manually adding the projects included in the GHI Developer's Guide, you will install the [NuGet package](https://www.nuget.org/packages/GHIElectronics.UWP.Shields.FEZHAT/ "FEZ HAT NuGet Package") that they provide. To do this, open the **Package Manager Console** (Tools/NuGet Package Manager/Package Manager Console) and execute the following command:

	````PowerShell
	PM> Install-Package GHIElectronics.UWP.Shields.FEZHAT
	````

	![Intalling GHI Electronics NuGet package](Images/intalling-ghi-electronics-nuget-package.png?raw=true)

	_Installing the FEZ hat Nuget package_

13. Add a reference to the FEZ HAT library namespace in the _MainPage.xaml.cs_ file. Find all the 'using' statements of code at the top of the file and add the following line of code to the end of them:

	````C#
	using GHIElectronics.UWP.Shields;
	````

14. Declare the variables that will hold the reference to the following objects, find the comment "//DECLARE VARIABLES HERE" and ad the code below:
 - **hat**: Of the type **Shields.FEZHAT**, will contain the fez hat driver object that you will use to communicate with the FEZ hat through the Raspberry Pi hardware.
  - **telemetryTimer**: of the type **DispatchTimer**, that will be used to poll the hat sensors at regular basis. For every _tick_ of the timer the value of the sensors will be get and sent to Azure.

	````C#
	FEZHAT hat;
	DispatcherTimer telemetryTimer;
	````

15. You will add the following method to initialize the objects used to handle the communication with the hat, find the comment "//ENTER SETUP HAT ASYNC METHOD HERE" and place below. 
	The **TelemetryTimer_Tick** method will be defined next, and will be executed every 500 ms according to the value hardcoded in the **Interval** property.

	````C#
	private async Task SetupHatAsync()
	{
	    this.hat = await FEZHAT.CreateAsync();

	    this.telemetryTimer = new DispatcherTimer();

	    this.telemetryTimer.Interval = TimeSpan.FromMilliseconds(500);
	    this.telemetryTimer.Tick += this.TelemetryTimer_Tick;

	    this.telemetryTimer.Start();
	}
	````

16. The following method will be executed every time the timer ticks, and will poll the value of the hat's temperature sensor, send it to the Azure IoT Hub and show the value obtained.
	Place the code just below the comment "//ENTER TELEMENTRYTIMER_TICK METHOD HERE"

	````C#
	private void TelemetryTimer_Tick(object sender, object e)
	{
	    // Temperature Sensor
	    var tSensor = ctdHelper.sensors.Find(item => item.measurename == "Temperature");
	    tSensor.value = this.hat.GetTemperature();
	    this.ctdHelper.SendSensorData(tSensor);

	    this.HelloMessage.Text = "Temperature: " + tSensor.value.ToString("N2");

	    System.Diagnostics.Debug.WriteLine("Temperature: {0} °C", tSensor.value);
	}
	````

	WHAT DOES THIS CODE DO?: The first statement gets the _ConnectTheDots_ sensor from the sensor collection already in place in the project (the temperature sensor was already included in the sample solution). Next, the temperature is polled out from the hat's temperature sensor using the driver object you initialized in the previous step. Then, the value obtained is sent to Azure using the _ConnectTheDots_'s helper object **ctdHelper** which is included in the sample solution.

	The last two lines are used to show the current value of the temperature sensor to the debug console respectively.


17. Before running the application you need to add the call to the **SetupHatAsync** method. Find the **Page_Loaded** method and place those two lines of code inside the {} curly brackets:

	````C#
	private async void Page_Loaded(object sender, RoutedEventArgs e)
	{
		// ADD CALL TO SETUP HAT ASYNC
		// Initialize FEZ HAT shield
		await SetupHatAsync();
	}
	````

	> Note you need to add the the **async** word (a modifier) to the event handler to properly handle an asynchronous call to the FEZ HAT initialization method. Place async as shown above between private and void
	
	> **Asynchronous** methods/keywords help optimize the execution of resources. An asynchronous method returns an answer immediately, allowing the calling program to perform other operations at the same time. Allowing multi-threaded programming. This is different to a synchronous method that waits for the method to complete before continuing to execute the rest of the code.

18. Ensure that the target platform for the project is set to "**ARM**":

	![arm-target-platform](Images/arm-target-platform.png?raw=true)

19. Build the solution to restore the NuGet packages, and make sure it builds:

	![ghifezhat-build-solution](Images/ghifezhat-build-solution.png?raw=true)

	![ghifezhat-build-succeeded](Images/ghifezhat-build-succeeded.png?raw=true)


20. Now you are ready to run the application. Connect the Raspberry Pi with the FEZ HAT and run the application. After the app is deployed you will start to see in the output console the values polled from the sensor. The information sent to Azure is also shown in the console.

	![Console output](Images/console-output.png?raw=true)

	_Output Window_

	You can also check that the messages were successfully received by monitoring them using the Device Explorer

	![Telemetry messages received](Images/telemetry-messages-received.png?raw=true)

<a name="Task6" />
## Keep building up your project

<a name="Task55" />
### Adding extra sensors
Now that your application is sending information from your device to the cloud for one sensor - lets add some more!


1. To incorporate the data from the Light sensor you will need to add a new _ConnectTheDots_ sensor:
	<!-- mark:3 -->
	````C#
	// Hard coding guid for sensors. Not an issue for this particular application which is meant for testing and demos
	List<ConnectTheDotsSensor> sensors = new List<ConnectTheDotsSensor> {
		new ConnectTheDotsSensor("2298a348-e2f9-4438-ab23-82a3930662ab", "Light", "L"),
		new ConnectTheDotsSensor("d93ffbab-7dff-440d-a9f0-5aa091630201", "Temperature", "C"),
	};
	````

2. Next, add the following code to the **TelemetryTimer_Tick** method to poll the data from the temperature sensor and send it to Azure.

	````C#
	// Light Sensor
	ConnectTheDotsSensor lSensor = ctdHelper.sensors.Find(item => item.measurename == "Light");
	lSensor.value = this.hat.GetLightLevel();

	this.ctdHelper.SendSensorData(lSensor);

	System.Diagnostics.Debug.WriteLine("Temperature: {0} °C, Light {1}", tSensor.value.ToString("N2"), lSensor.value.ToString("P2", System.Globalization.CultureInfo.InvariantCulture));
	````

	After running the app you will see the following output in the debug console. In this case two messages are sent to Azure in every timer tick:

	![Debug console after adding Light Sensor](Images/debug-console-after-adding-light-sensor.png?raw=true)

	_Output Window after Adding the Light Sensor_


<a name="Task44" />
### Send commands to your device
Azure IoT Hub is a service that enables reliable and secure bi-directional communications between millions of IoT devices and an application back end. In this section you will see how to send cloud-to-device messages to your device to command it to change the color of one of the FEZ HAT leds, using the Device Explorer app as the back end.

1. Open the Universal app you created before and add the following method to the **ConnectTheDotsHelper.cs** file. Add the code to the bottom of the file where you see the comment "//ADD RECIEVE MESSAGE METHOD HERE":

	````C#
	public async Task<string> ReceiveMessage()
	{
		if (this.HubConnectionInitialized)
		{
			try
			{
				var receivedMessage = await this.deviceClient.ReceiveAsync();

				if (receivedMessage != null)
				{
					var messageData = Encoding.ASCII.GetString(receivedMessage.GetBytes());
					this.deviceClient.CompleteAsync(receivedMessage);
					return messageData;
				}
				else
				{
					return string.Empty;
				}
			}
			catch (Exception e)
			{
				Debug.WriteLine("Exception when receiving message:" + e.Message);
				return string.Empty;
			}
		}
		else
		{
			return string.Empty;
		}
	}
	````

	The _ReceiveAsync_ method returns the received message at the time that it is received by the device. The call to _CompleteAsync()_ notifies IoT Hub that the message has been successfully processed and that it can be safely removed from the device queue. If something happened that prevented the device app from completing the processing of the message, IoT Hub will deliver it again.
	
2. Now you will add the logic to process the messages received. Open the **MainPage.xaml.cs** file and add a new timer to the _MainPage_ class. Add the new variable to the section "//DECLARE VARIABLES HERE":

	````C#
	DispatcherTimer commandsTimer;
	````

3. Add the following method, which will be in charge of processing the commands where it says "//ENTER COMMANDTIMER_TICK METHOD HERE":

	````C#
	private async void CommandsTimer_Tick(object sender, object e)
	{
		string message = await ctdHelper.ReceiveMessage();

		if (message != string.Empty)
		{
			System.Diagnostics.Debug.WriteLine("Command Received: {0}", message);
			switch (message.ToUpperInvariant())
			{
				case "RED":
					hat.D2.Color = new FEZHAT.Color(255, 0, 0);
					break;
				case "GREEN":
					hat.D2.Color = new FEZHAT.Color(0, 255, 0);
					break;
				case "BLUE":
					hat.D2.Color = new FEZHAT.Color(0, 0, 255);
					break;
				case "OFF":
					hat.D2.TurnOff();
					break;
				default:
					System.Diagnostics.Debug.WriteLine("Unrecognized command: {0}", message);
					break;
			}
		}
	}
	````

	It reads the message received, and according to the text of the command, it set the value of the _hat.D2.Color_ attribute to change the color of the FEZ HAT's LED D2. When the "OFF" command is received the _TurnOff()_ method is called, which turns the LED off.
	
4. Lastly, add the following piece of code to the _SetupHatAsync_ method in order to initialize the timer used to poll for messages. 

	````C#
	this.commandsTimer = new DispatcherTimer();
	this.commandsTimer.Interval = TimeSpan.FromSeconds(60);
	this.commandsTimer.Tick += this.CommandsTimer_Tick;
	this.commandsTimer.Start();
	````

	> **Note:** The recommended interval for HTTP/1 message polling is 25 minutes. For debugging and demostration purposes a 1 minute polling interval is fine (you can use an even smaller interval for testing), but bear it in mind for production development. Check this [article](https://azure.microsoft.com/en-us/documentation/articles/iot-hub-devguide/) for guidance. When AMQP becomes available for the IoT Hub SDK using UWP apps a different approach can be taken for message processing, since AMQP supports server push when receiving cloud-to-device messages, and it enables immediate pushes of messages from IoT Hub to the device. The following [article](https://azure.microsoft.com/en-us/documentation/articles/iot-hub-csharp-csharp-c2d/) explains how to handle cloud-to-device messages using AMQP.
	
5. Deploy the app to the device and open the Device Explorer app.

6. Once it's loaded (and configured to point to your IoT hub), go to the **Messages To Device** tab, check the **Monitor Feedback Endpoint** option and write your command in the **Message** field. Click on **Send**

	![Sending cloud-to-device message](Images/sending-cloud-to-device-message.png?raw=true)

7. After a few seconds the message will be processed by the device and the LED will turn on in the color you selected. The feedback will also be reflected in the Device Explorer screen after a few seconds.

	![cloud-to-device message received](Images/cloud-to-device-message-received.png?raw=true)


<a name="Summary" />
## Summary
In this lab, you have learned how to create a Universal app that reads from the sensors of a FEZ hat connected to a Raspberry Pi 2 running Windows 10 IoT Core, and upload those readings to an Azure IoT Hub. You also added more sensors to your application and implemented how to use the IoT Hubs Cloud-To-Device messages feature to send simple commands to your devices.
