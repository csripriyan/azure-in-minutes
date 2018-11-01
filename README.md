# Publishing an mqtt packet from RPi3 to Azure IoT Hub : A no-frill step-by-step guide
# 1. OBJECTIVE:
This article is about setting up Azure IoT Hub, sending some data using mosquitto_pub from Raspberry Pi3 and ensuring that the data is received by Azure IoT Hub.

When I searched the Internet on how to get started with Azure IOT, I did not find any easy step-by-step guidance. In this article I have tried to put my findings together so that you would be able to push your first packet into Azure IoT cloud relatively quickly. I do not claim that this is a better method nor the only method available; but it would help you to start off easily.

# 2. BEFORE YOU START:
## You need Azure Account setup:
- For this experiment I used a free azure account
## You need the below hardwares available:
- RPi3 for acting as device.
- Windows machine:
* a. To run the utilities Device Explorer and Storage explorer
* b. To access https://portal.azure.com/
## Note:
* Wherever I needed to give name, I started with zippy.
* The device on IoT Hub is named as tiny1.
* The keys, account name and other identifications (such as zippy, tiny1) are made to look real; but they are not :wink:

# 3. ON AZURE PORTAL:
## Creating Iot Hub:

Log in to your azure portal and create a IOT Hub using the instructions available here.

https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-create-through-portal

HUB Name: zippyhub

Pricing: F1 - Free

HotHub Units: 1 (not configurable; feature disabled)

Device To Cloud partitions: 2 (not configurable; feature disabled)

Subscription : Pay-As-You-Go

Resource Grp: Create New: zippyresgrp

Location: East US

[x] Pin To Dashboard

For this experiment you don’t need all the details available from this web page. You don’t have to create “Shared access policies", “Endpoints” or “Routes” at this point of time.

## Creating Iot Device:

You need to create one IOT device inside IoT Hub, zippyhub.

Select IoT Devices → + Add → Give tiny1 as Device ID. Accept all other default parameters and press “Save“


## Creating storage:
Create a storage using the instructions available here. https://docs.microsoft.com/en-gb/azure/iot-hub/iot-hub-store-data-in-azure-table-storage

### Create storage account:

storage account Name: zippystorage

Deployment model: Resource manager

Acct kind: Storage(general purpose v1)

Performance : Standard

Replication: Zone-redundant storage(ZRS classic)

Secure Transfer Required: Disabled

Subscription: Pay As You Go

Resource Group: Use Existing -> zippyresgrp

Location : East US

Virtual network: (not configurable; feature disabled)

[x] Pin to dashboard

### Add Endpoint:

Name : zippystorageep

Endpoint Type: Azure storage container

Azure storage container: select zippystorage

### Add container:

Name : zippycontainer

Pub access level: Container (anonymous read access for containers and blobs)

Azure storage container becomes, https://zippystorage.blob.core.windows.net/zippycontainer

Accept the default Blob file name format:

{iothub}/{partition}/{YYYY}/{MM}/{DD}/{HH}/{mm}

### Add new routes:

route 1:

Name: zippystorageroute

Data source: device messages

Endpoint: zippystorageep

route 2:

Name: zippyroute

Data source: device messages

Endpoint: events

# 4. ON WINDOWS MACHINE:

a. Install Storage Explorer. This would be needed later to check if the data published by mqtt client is received by the Azure IOT Hub.

b. SharedAccessSignature is needed for this experiment. And one of the ways to get this information is by using Device Explorer. Download Device Explorer binary for windows and install it.

Before you launch the application Device Explorer you need the below two information From https://portal.azure.com:

1. Get IoT Hub connection string from IotHub → Shared Access Policies → iothubowner → Connection string—primary key
HostName=zippyhub.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=T6BeWd4KLd12gWsVK5paODYse8YooDDknXYfuXvUPIc=

2. From the connection string you know, HostName=zippyhub.azure-devices.net

Now launch Device Explorer and furnish these two information:

IoT Hub Connection String:
HostName=zippyhub.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=T6BeWd4KLd12gWsVK5paODYse8YooDDknXYfuXvUPIc=

Protocol Gateway HostName:
zippyhub.azure-devices.net

Then press "Update"

Note: You can configure the validity of this key, with max limit of 365 days.

Press "Generate SAS". The key is generated. It looks something like this,

SharedAccessSignature sr=zippyhub.azure-devices.net&sig=Lx%2fn7j7VrUVFDYX67ZKUaGhpKUfY2bJd8NLuADhFCOez4%3d&se=1780777960&skn=iothubowner

# 5. RPi3 DEVELOPMENT KIT:

Use the above generated SAS to frame the mosquitto_pub command. Shown below is the command being punched in at the command prompt.

    `# mosquitto_pub -h zippyhub.azure-devices.net -p 8883 -t "devices/tiny1/messages/events/" -i tiny1 -u "zippyhub.azure-devices.net/tiny1" -P "SharedAccessSignature sr=zippyhub.azure-devices.net&sig=Lx%2fn7j7VrUVFDYX67ZKUaGhpKUfY2bJd8NLuADhFCOez4%3d&se=1780777960&skn=iothubowner" --capath /etc/ssl/certs/ --tls-version tlsv1 -d -V mqttv311 -q 1 -m "{\"HELLO\": \"AZURE\"}"`

You should see something like this,

```
Client tiny1 sending CONNECT
Client tiny1 received CONNACK
Client tiny1 sending PUBLISH (d0, q1, r0, m1, 'devices/tiny1/messages/events/', ... (18 bytes))\n
Client tiny1 received PUBACK (Mid: 1)
Client tiny1 sending DISCONNECT
```

If there is any ssl related error, probably its a good idea to update the certificate file using the below command.

    `sudo echo -n | openssl s_client -connect zippyhub.azure-devices.net:8883 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/ssl/certs/ca-azure.crt`

# 6. VERIFYING THE DATA ON CLOUD:

**From https://portal.azure.com:**
* Select zippystorage → Access Keys
* Storage Account Name: zippystorage
* Under key1 section, key: ueZw5OlXkoBduKkS0MD12gNYvTHy3UhA0DV04ZpPdNRzs+ot5a+dbR5SDxMzPffiQSz6czEvfdf3IT3O5iUnQW==

**Account name and Account key**
* Invoke Azure Storage explorer.
* Connect to Azure Storage:
* Select Use a storage account name and key. Fill the details,
  * Account Name: zippystorage
  * Account Key: ueZw5OlXkoBduKkS0MD12gNYvTHy3UhA0DV04ZpPdNRzs+ot5a+dbR5SDxMzPffiQSz6czEvfdf3IT3O5iUnQW==
  * Storage Endpoints Domain: Azure

Now its ready to view the data. Select the zippystorage under Storage Accounts.

Blob Containers → zippycontainer → zippyhub → 00 → 2018 → 02 → 21 and so on till you reach the latest blobs.

**Note:** The blobs are organized as {iothub}/{partition}/{YYYY}/{MM}/{DD}/{HH}/{mm} following the configuration defined when the container was created.
I did not find much time exploring the ways to see the content online “live”. So, I just went ahead and downloaded the blob and opened it in editor.
Select the file and press the download button available on the top menu bar. If you are not bothered by the presence of non-printable ascii characters open the blob file in a text editor. You should be able to locate the string, {"HELLO": "AZURE"}.

# 7. APPENDIX:
Before experimenting with mosquitto_pub I ran a sample code from azure-iot-sdk-c code. To run one of the mqtt publish example here are the step to follow,

1. Go to home directory in RPi3. And issue download the code.

    `# git clone -b 2018-01-29 --recursive https://github.com/Azure/azure-iot-sdk-c.git`

2. Open the file /home/pi/azure-iot-sdk-c/iothub_client/samples/iothub_client_sample_mqtt_esp8266/iothub_client_sample_mqtt_esp8266.c. A static global variable, “connectionString”, at the beginning of the program needs to be initialized with proper key.
The connection string is nothing but “the connection string – primary key” of the device that we want to communicate to. In this case, it is tiny1. The modified code looks something like this,

    `static const char* connectionString = "HostName=zippyhub.azure-devices.net;DeviceId=tiny1;SharedAccessKey=R5sCKRFq5Vh/m49ADDvFEaGR9BH8e39ifKgwDxWvcOd=”;`
3. Compile the entire sdk,
    `# mkdir /home/pi/azure-iot-sdk-c/cmake/`
    `# cd /home/pi/azure-iot-sdk-c/cmake/`
    `# cmake ..`
    `# cmake --build .`
4. Whenever you modify this example, recompile,
    `# cd /home/pi/azure-iot-sdk-c/cmake/iothub_client/samples/iothub_client_sample_mqtt_esp8266`
    `# make`
5. Run the example,
    `# cd /home/pi/azure-iot-sdk-c/cmake/iothub_client/samples/iothub_client_sample_mqtt_esp8266`
    `# ./iothub_client_sample_mqtt_esp8266`

Few more examples,

Example 1: Checking if the data is published into Azure cloud.

Open a Linux terminal & issue the below subscribe command, 

    `$ iothub-explorer monitor-events tiny1 --login "HostName=zippyhub.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=T6BeWd4KLd12gWsVK5paODYse8YooDDknXYfuXvUPIc=" `

Open another Linux terminal & issue the below publish command, 

    `$ mosquitto_pub -h zippyhub.azure-devices.net -p 8883 -t "devices/tiny1/messages/events/" -i tiny1 -u "zippyhub.azure-devices.net/tiny1" -P "SharedAccessSignature sr=zippyhub.azure-devices.net&sig=Lx%2fn7j7VrUVFDYX67ZKUaGhpKUfY2bJd8NLuADhFCOez4%3d&se=1780777960&skn=iothubowner" --cafile /etc/ssl/certs/653b494a.0 --tls-version tlsv1 -d -V mqttv311 -q 1 -m "{\"HELLO\": \"AZURE\"}"`
    (Note: in place of --cafile you can use--capath /etc/ssl/certs/ too) 

Example 2: Checking if the data can be subscribed from Azure cloud. 

Open a Linux terminal & issue the below subscribe command, 

    `$ mosquitto_sub -h viviohub.azure-devices.net -p 8883 -t "devices/tiny1/messages/devicebound/#" -i tiny1 -u "viviohub.azure-devices.net/tiny1" -P "SharedAccessSignature sr=viviohub.azure-devices.net&sig=BlfhdbTaNuVNQ7kt3IQnHtwwGG2w37l7VrechK1Wo3A%3D&se=1554304897&skn=iothubowner" --cafile /etc/ssl/certs/653b494a.0 --tls-version tlsv1 -d -V mqttv311 -q 1 `
    
Open another Linux terminal & issue the below publish command, 

    `$ iothub-explorer send tiny1 "{'April':'value03" --login "HostName=zippyhub.aPublishing an mqtt packet from RPi3 to Azure IoT Hub : A no-frill step-by-step guidezure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=T6BeWd4KLd12gWsVK5paODYse8YooDDknXYfuXvUPIc="`
    
    
