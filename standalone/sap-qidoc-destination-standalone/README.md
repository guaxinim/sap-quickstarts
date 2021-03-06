Standalone SAP Queued IDoc Destination Endpoint Quick Start   
===========================================================   
**Demonstrates the sap-qidoc-destination component running in a standalone camel runtime.**  
![SAP Tool Suite](../../sap_tool_suite.png "SAP Tool Suite")

* * * 
Author: William Collins - Fuse Team  
Level: Beginner  
Technologies: SAP, Camel, Spring  
Summary: This quickstart demonstrates how to configure and use the sap-qidoc-destination component. This component sends IDoc documents to SAP using the *Queued RFC* (qRFC) protocol.  
Target Product: Fuse  
Source: <http://github.com/punkhorn/sap-quickstarts/>  

* * *

What is it?  
-----------  

This quick start shows how to integrate Apache Camel with SAP using the JBoss Fuse SAP Queued IDoc Destination Camel component. This component and its endpoints should be used in cases where a camel route is required to send Intermediate documents (IDocs) to an SAP system in order.  

This quick start uses XML files containing serialized IDoc documents to create Customer records in the Flight Data Application within SAP. These files are consumed by the quickstart's route and their contents are then converted to string message bodies. These messages are then routed to an `sap-qidoc-destination` endpoint which converts and sends them to SAP as `FLCUSTOMER_CREATEFROMDATA01` type IDoc documents to create Customer records.  These iDocs are serializes within SAP on the inbound queue `QUICKSTARTQUEUE`. 

In studying this quick start you will learn:

* How to define a Camel route containing the JBoss Fuse SAP Queued IDoc Destination Camel component using the Spring XML syntax.
* How to use the JBoss Fuse SAP Queued IDoc Destination Camel component to send IDocs to SAP. 
* How to configure connections used by the component.

For more information see:

* <https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_Fuse/6.2/html/Apache_Camel_Component_Reference/SAP.html> for more information about the JBoss Fuse SAP Camel components 
* <https://access.redhat.com/site/documentation/JBoss_Fuse/> for more information about using JBoss Fuse

System requirements
-------------------

Before building and running this quick start you will need:

* Maven 3.0.4 or higher
* JDK 1.7 or 1.8
* JBoss Fuse 6.2
* SAP JCo3 and IDoc3 libraries (sapjco3.jar, sapidoc3.jar and JCo native library for your OS platform)
* SAP instance with [Flight Data Application](http://help.sap.com/saphelp_erp60_sp/helpdata/en/db/7c623cf568896be10000000a11405a/content.htm) setup.

Configuring the ALE Subsystem
-----------------------------

To send IDocs from the quick start's route to your SAP system, you must first configure the Application Linking Enabling (ALE) subsystem in your SAP system:

1. Using the SAP GUI, run transaction `SALE`, the ALE Implementation Guide.
2. Ensure that Logical Systems for the quick start and your SAP client have been defined:  
    a. Run the `Define Logical System` step (Basic Systems > Logical Systems > Define Logical System).  
    b. Click `New Entries` and create and save the following logical systems:    
    
        | Log.System | System     |   
        | ---------- | ---------- |     
        | QUICKSTART | QUICKSTART |      
        | QUICKCLNT  | QUICKCLNT  |

	c. Return to the `SALE`  transaction main screen (Goto > Back).
3. Ensure the `QUICKCLNT` logical system has been assigned to you SAP client:  
    a. Run the `Assign Logical System to Client` step (Basic Settings > Logical Systems > Assign Logical System to Client).   
    b. Select the `QUICKCLNT` for your client's `Logical system` and save your changes.     
    c. Return to the `SALE`  transaction main screen (Goto > Back).  
4.  Ensure the destination `QUICKSTART` has been defined:   
    a. Run the `Create RFC Connections` step (Communication > Create RFC Connections).    
    b. Create a new destination (Edit > Create):  
		1. **RFC Destination** : `QUICKSTART`.    
        2. **Connection Type** : `T`.    
        3. **Technical Settings** :    
            i. **Activation Type** : `Registered Server Program`.    
            ii.**Program ID** : `QUICKSTART`.   
        4. **Unicode**:   
        	i. **Communication Type with Target System** : `Unicode`   
	c. Return to the `SALE` transaction main screen (Goto > Back).   
5. Ensure that a Model View for the message flow from the quick start to your SAP system has been defined:  
   a. Run the `Maintain Distribution Model and Distribute Views` step (Modelling and Implementing Business Processes > Maintain Distribution Model and Distribute Views).   
   b. Ensure a `QUICKSTART` model view has been created with the technical name `QUICKSTART` (Edit > Model view > Create).   
   c. Ensure the `QUICKSTART` model view has a BAPI call configured (Edit > Add BAPI):   
      1. **Sender/client** : `QUICKSTART`.  
      2. **Reciever/server** : `QUICKCLNT`.  
      3. **Obj.name/interface** : `FlightCustomer`.  
      4. **Method** : `CreateFromData`.   
   d. Ensure `Partner Profiles` have been generated for the quick start and your SAP client (Environment > Generate Partner Profiles).   
 
Configuring the Quickstart for your environment
-----------------------------------------------

To configure the quick start for your environment: 

1. Deploy the JCo3 library jar and native library (for your platform) and IDoc3 library jar to the `lib` folder of the project.
* Edit the project's Spring file (`src/main/resources/META-INF/spring/camel-context.xml`) and modify the `quickstartDestinationData` bean to match the connection configuration for your SAP instance. 
* Edit the project's IDoc files (`src/data/idoc?.xml`) and enter the SID of your SAP in the location indicated.

Build and Run the Quickstart
----------------------------

To build and run the quick start:

1. Change your working directory to the `sap-qidoc-destination-standalone` directory.
* Run `mvn clean install` to build the quick start.
* Run `mvn camel:run` to start the Camel runtime.
* In the console observe the contents of the IDoc processed by the route.
* Execute the queued IDocs waiting in the inbound queue `QUICKSTARTQUEUE`. Using the SAP GUI, run transaction `SMQ2`, the Inbound Queue qRFC Monitor:  
    a. Select the `QUICKSTARTQUEUE` queue.  
    b. Display the queue contents (Edit > Display Selection).  
    c. Select the entry for your Client connection and activate the queue (Edit > Activate).  
* Using the SAP GUI, run transaction `SE16`, Data Browser, and display the contents of the table `SCUSTOM`.
* Search the table (Edit > Find..) for the newly created Customer records: `Fred Flintstone`, `Wilma Flintstone`, `Barney Rubble`, and `Betty Rubble`. 

Stopping the Quickstart
-----------------------

To stop the camel run-time:

1. Enter Ctrl-c in the console.

