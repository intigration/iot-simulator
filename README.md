


----------
IoT-data-simulator is the tool which allows you to simulate IoT devices data with great flexibility. With this tool you won't need to code another new simulator for each IoT project.


Simulator **features** that you will like:
1. Replay existing datasets with modified data (such as updated timestamps, generated ids etc);
2. Automatic derivation of dataset structure which allows you to customize your dataset without the need to describe its structure from scratch;
3. Generate datasets of any complexity. Generated data can be described via constructor or JavaScript function. Multiple rules are available in constructor such as "Random integer", "UUID" and others. JS function  gives you maximum flexibility to generate data - it supports popular JS libraries *lodash* and *momentjs*.
4. Send data to the platforms you use with minimum configuration (see *Supported target systems* section);
5. Customize frequency with which data will be sent - based on dataset timestamp properties or just constant time interval. The tool also supports relative timestamp properties which depend on other timestamp or date properties. It means that data can be replayed with the same interval between timestamps as in initial dataset.
6. Easy installation - all you need is to download 2 docker files and run 2 commands.

If you would like read more about why we created this tool, please read **Motivation** section.

## Introduction video

## Installation

**Prerequisites**
*docker* (v. 17.05+) and *docker-compose* should be installed

**Startup**
Run the following commands in the folder with *docker-compose.yml* and *.env* files from *release* folder:

    docker-compose pull
    docker-compose up

After a while UI will be available by the following url:
http://localhost:8090 or http://docker-machine-ip:8090
depending on the OS or docker version.

## Terms

To use IoT-data-simulator you should understand the following concepts:

 - **session** – main composite application entity. Data is being generated and sent to external IoT platforms only when session is running. Session consists of *data definition (optional)*, *timer*, *devices (optional)*, *processing rules* and *target system*;
 
 - **data definition** - describes data structure that is sent to a *target system*. Consists of *dataset*, or *schema*, or *dataset* **and** *schema*.

	 - *dataset* (required if no schema provided) - .csv or .json file that is uploaded by user to the internal tool object storage (minio) via UI or to the external Amazon S3 service. 
If *dataset* is provided, its content is replayed by the tool during session run.

	 - *schema* (required if dataset is not provided) - describes dataset structure or data that will be generated on runtime. Schema can be derived from dataset or created from scratch via UI constructor. If schema is not provided, data will be generated via custom JS function provided by user.

 - **target system** - describes external system where simulated data is sent to;
 
 - **device** – encapsulates specific set of values and target system properties. 
	 - Devices can be used to override specific values in generated payload. It can be useful when you have common payload structure and would like to ovveride some specific properties such as *id*, *geo position* for different IoT devices.
	 - Devices can be used to override *target system* properties. This use case is helpful for instance when each device should send payload to its own topic.
 
##  Usage

**Replay existing dataset as is**

The simplest use case which can help you to start working with the tool - replay existing dataset as is. 

We need to create *session* which will send dataset records without any modification to a target system. For this use case we don't need data *schema* - just JS function that will return current dataset record (*datasetEntry*). Go to the *create session* screen, then:

   1. Upload new *json* or *csv* dataset on data definition creation flow and skip schema step;
   2. Select timer options (how often data will be generated by the tool)
   3. Skip "Select devices" step. 
   4. Apply data processing rules.  As you can see processing function is very simple:

    function process(state, datasetEntry, deviceName) { 
	    return datasetEntry;
    }

   5. Select or create *dummy* target system which is not actually sending data anywhere but allows you to see payload in session console;
   6. Enter session name and complete creation flow;
   7. Run session and explore payload in the session console;

Session will stop when all records from dataset will be read.

----------

**Replay dataset with updated date/timestamp properties**

Replaying dataset without modifying any parameters is not really useful use case, so let's replay dataset with updated date/timestamp properties. Go to *create session* screen, then:

   1. Upload dataset which has at least one date/timestamp property on *create definition* screen. Make sure that date/timestamp property type has been derived correctly and apply *schema*. Dataset may look like the following:
   

    {"timestamp": 1517303155600, "id": 1 }
    {"timestamp": 1517303155800, "id": 2 }
    ...

   2. Select timer options. 
   3. Skip *Select devices* step
   4. Apply data processing rules. 
*Current time* rule is selected by default for date/timestamp property. It means that timestamp property will be updated with current time, but the difference between timestamps of two adjacent dataset records will stay the same. Also, if there are more than one timestamp properties in one record, the first one will have *Current time* rule by default, others - *Relative time* rule, which means that timestamps will be updated relatively to *Current time* timestamp.

   6. Select or create *dummy* target system
   7. Enter new session name and complete creation flow
   8. Run session and explore session console

---------

**Generate data**

If you would like to generate data on fly. This can be achieved in two possible ways:

a) With JS function. Go to *Create session* screen, then:

   1. Skip *Select definition* step;
   2. Select timer options;
   3. Skip "Select devices" step. 
   4. On data processing step you can write any JS function that returns any value. By default the following function is used:

    function process(state, datasetEntry, deviceName) { 
	    return {
	        timestamp: moment().valueOf()
	    }
    }


   5. Select or create *dummy* target system;
   6. Enter session name and complete creation flow;
   7. Run session and explore payload in the session console;

b) With schema
   
   1. Skip dataset selection step on definition creation flow;
   2. Create simple schema
   3. Select timer options (how often data will be generated by the tool)
   4. Skip "Select devices" step. 
   5. Apply data processing rules via rules constructor;
   6. Create or select *dummy* target system;
   7. Enter session name and complete creation flow
   8. Run session and explore payload in the session console;


----------

## Intended usage and useful scenarios

**Derive schema from dataset and use it without dataset  (data generation mode)**

This scenario is very useful when dataset has complex structure and you doesn't want to create schema for it from scratch.
This can be achieved with the following steps:
1. Create definition with dataset and apply derived schema;
2. Update created definition and unselect dataset - derived schema won't be removed;

----------


**Store state between data processing iterations. Implement counter**

Let's say we would like to create simple counter with the following payload sent to a target system:

    {
       "count": 1 // number which increases on each iteration
    }

In this case we could use *data generatation* mode (see **Usage** section above). On the step #2 create schema with one integer property as shown on the screenshot below:
![counter-example](https://user-images.githubusercontent.com/4072962/38543711-fe726e92-3cad-11e8-9e64-fe633a39735c.png)

On step #5 select "Custom function" rule for the "count" property. Open editor and apply the following JS function:

    function custom(ruleState, sessionState, deviceName) { 
        if(typeof ruleState.counter === 'undefined') { 
            ruleState.counter = 0;
        }
        ruleState.counter++
        return ruleState.counter;
    }


----------


**Send data to different Kafka topics**:

To send data to different Kafka topics we should create several devices which will override target system properties with their "topic data".
Lets **generate** data for our example. 
Go to *Create session* screen, then:

   1. Create data definition without dataset and with simple schema;
   2. Select timer options
   3. Go to "Create device" screen, enter device name and press "Proceed".
   4. You should see *target system* screen that is specific for this device. Go to *Create target system* screen;
   5. Enter *target system* name and select "Kafka" type. 
   6. Populate only *topic* field with "Kafka" topic corresponding to this device. When creating device specific target system, only populated fields will override session target system properties.
   7. Repeat steps 3 - 6 for all Kafka topics
   8. Select all devices on *Select devices* step of session creation flow and press "Proceed";
   9. Select device injection rule. Please, read more about device injection rules in project Gitbook;
   10. Apply processing rules.
   11. Create or select Kafka target system (*topic* should be filled, but anyway it will be overwritten by devices specific target systems *topic* parameter;
   12. Enter session name and complete session creation flow;
   13. Run created session and observe session console;

----------

**Generate dataset and save it to local minio object storage**

To generate data and save it as file on local minio object storage create *Local storage* target system while creating session. Populate *dataset* field with desired file name.


----------

**Inject device property to processing rules**

 Go to *Create session* screen, then:
  
 1. Select data definition with schema
 2. Select timer options
 3. Create one or more devices with populated properties fields
 4. Select created devices
 5. Select device injection rule
 6. On *processing rules* step select *Device property* rule for specific property that you would like to overwrite with device property;
 7. Complete session creation flow


## Supported target systems

IoT-data-simulator can send payload to the following *target systems* (with available security types in parentheses):

 - AMQP (credentials and certificates)
 - Kafka (certificates)
 - Local object storage (*minio* object storage)
 - MQTT (access keys, access token, credentials, certificates)
 - REST (credentials, certificates)
 - Websocket (credentials, certificates)

## FAQ

**Q: I've updated session but it still uses old parameters**
A: Session parameters are updated only when session is stopped. If session was paused, it will still use the previous parameters;

**Q: What IoT platforms do you support?**
A: In our company we used IoT data simulator tool for working with Thingsboard, AWS and Predix platforms. But it doesn't mean that it cannot be used with others.

## Motivation

An IoT data simulator is a required tool in any IoT project. While using real world sensors and devices is required for final integration testing to make sure the system works end to end without any surprises, it is very impractical to use the real devices during development and initial testing phases where tests tend to be quicker, shorter, failing quicker and more often.

Data simulator solves several problems at once:
1.	It is not necessary to run the physical equipment just to perform a unit or early integration test.
2.	You can use synthetic generated data if you do not yet have access to the physical devices, or you can replay captured data to get it as real as possible.
3.	By imploding the data with some randomization, you can simulate a large number of devices that you may not be able to get hold of during the test and development.
4.	You can replay data at various frequencies to run the test quicker (e.g. when the real world data is pushed every 5 minutes and it takes several days to trigger an alert).
5.	You can simulate data from different stages of the system, so that the testing of the later stages does not have to wait until all the previous stages are completed to transform raw data into a suitable input. 
For example, you can simulate raw device data to develop and test initial processing, simulate “cleaned up” data stream to develop and test analytic components, and simulate analytic component results to develop and test visualization and UX.


## Contributing

#
