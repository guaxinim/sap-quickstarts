Fuse SAP Synchronous RFC Server Endpoint Quick Start
=======================================================================================================================
**Demonstrates the sap-srfc-server component running in a Fuse camel runtime.**   
![SAP Tool Suite](../../sap_tool_suite.png "SAP Tool Suite")

***
Author: William Collins - Fuse Team  
Level: Beginner  
Technologies: SAP, Camel, Spring  
Summary: This quickstart demonstrates how to configure and use the sap-srfc-server component in a Fuse environment to handle a remote function call from SAP. This component handles a remote function call from SAP using the *Synchronous RFC* (sRFC) protocol.       
Target Product: Fuse  
Source: <http://github.com/punkhorn/sap-quickstarts/>  

***

What is it?  
-----------  

This quick start shows how to integrate Apache Camel with SAP using the JBoss Fuse SAP Synchronous Remote Function Call Server Camel component. This component and its endpoints should be used in cases where a Camel route is required to synchronously handle requests from and responses to an SAP system.  

This quick start handles requests from SAP for the `BAPI_FLCUST_GETLIST` BAPI method to query for Customer records in the Flight Data Application. The route of this quick start simply mocks the behavior of this method by returning a fixed response of Customer records. The `sap-srfc-server` endpoint at the beginning of the route consumes requests from SAP and their contents are then converted to string message bodies and logged to the console. The message body of the exchange's message is then replaced with the contents of an XML file that contains a serialized SAP response for the BAPI method. This new message body content is then converted to a string, logged to the console and then sent back by the endpoint to SAP as the response to the call.

**NOTE** The sRFC protocol used by this component delivers requests and responses to and from an SAP system **BEST-EFFORT**. In the case of a communication error when sending a request whether a remote function call is executed in a receiving SAP system is *in doubt*.     

In studying this quick start you will learn:

* How to define a Camel route containing the JBoss Fuse SAP Synchronous Remote Function Call Server Camel component using the Blueprint XML syntax.
* How to use the JBoss Fuse SAP Synchronous Remote Function Call Server Camel component to handle requests from SAP. 
* How to configure connections used by the component.
* How to configure the Fuse runtime environment in order to deploy the JBoss Fuse SAP Synchronous Remote Function Call Server Camel component.

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

Configuring the Quickstart for your environment
-----------------------------------------------

To configure the quick start for your environment: 

1. Deploy the JCo3 library jar and native library (for your platform) and IDoc3 library jar to the `lib` folder of your JBoss Fuse installation.  
2. Edit the custom properties file (`etc/custom.properties`) of your JBoss Fuse installation and add the following packages to the `org.osgi.framework.system.packages.extra` property:  

> org.osgi.framework.system.packages.extra = \  
>...  
>> com.sap.conn.idoc, \  
>> com.sap.conn.idoc.jco, \   
>> com.sap.conn.jco, \   
>> com.sap.conn.jco.ext, \   
>> com.sap.conn.jco.monitor, \  
>> com.sap.conn.jco.rt, \   
>> com.sap.conn.jco.server  

3. Edit the project's Blueprint file (`src/main/resources/OSGI-INF/blueprint/camel-context.xml`) and modify the `quickstartDestinationData` bean and the `quickstartServerData` bean to match the connection configuration for your SAP instance. 
4. Ensure the destination `QUICKSTART` has been defined in your SAP instance:   
	a. Using the SAP GUI, run transaction `SM59` (RFC Destinations).    
    b. Create a new destination (Edit > Create):  
		1. **RFC Destination** : `QUICKSTART`.    
        2. **Connection Type** : `T`.    
        3. **Technical Settings** :    
            i. **Activation Type** : `Registered Server Program`.    
            ii.**Program ID** : `QUICKSTART`.   
        4. **Unicode**:   
        	i. **Communication Type with Target System** : `Unicode`   
5. Ensure the following `ZBAPI_FLCUST_GETLIST` ABAP program is installed and activated in your SAP client:  

			*&---------------------------------------------------------------------*
			*& Report  ZBAPI_FLCUST_GETLIST
			*&
			*&---------------------------------------------------------------------*
			*&
			*&
			*&---------------------------------------------------------------------*
			
			REPORT  ZBAPI_FLCUST_GETLIST.
			
			
			DATA: RFCDEST LIKE RFCDES-RFCDEST VALUE 'NONE'.
			
			
			DATA: RFC_MESS(128).
			
			
			DATA: CUSTOMER_DATA LIKE BAPISCUDAT,
			      IT_CUSTOMER_LIST TYPE STANDARD TABLE OF BAPISCUDAT.
			
			DATA: IT_RETURN TYPE STANDARD TABLE OF BAPIRET2,
			      RETURN TYPE BAPIRET2.
			
			RFCDEST = 'QUICKSTART'.
			
			CALL FUNCTION 'BAPI_FLCUST_GETLIST'
			  DESTINATION RFCDEST
			  TABLES
			    CUSTOMER_LIST = IT_CUSTOMER_LIST
			    RETURN = IT_RETURN.
			
			IF SY-SUBRC NE 0.
			
			WRITE: / 'Call ZBAPI_FLCUST_GETLIST SY-SUBRC = ', SY-SUBRC.
			WRITE: / RFC_MESS.
			
			ELSE.
			
			WRITE: / 'CUSTOMER_LIST:'.
			ULINE.
			
			WRITE: /5 'CUSTOMERID', 16 'FORM', 30 'CUSTNAME', 55 'STREET', 85 'POSTCODE', 95 'CITY', 120 'COUNTR', 127 'PHONE', 157 'EMAIL'.
			
			LOOP AT IT_CUSTOMER_LIST INTO CUSTOMER_DATA.
			
			WRITE: /5 CUSTOMER_DATA-CUSTOMERID, 16 CUSTOMER_DATA-FORM, 30 CUSTOMER_DATA-CUSTNAME, 55 CUSTOMER_DATA-STREET, 85 CUSTOMER_DATA-POSTCODE, 95 CUSTOMER_DATA-CITY, 120 CUSTOMER_DATA-COUNTR, 127 CUSTOMER_DATA-PHONE, 157 CUSTOMER_DATA-EMAIL.
			
			ENDLOOP.
			
			WRITE: / 'RETURN:'.
			ULINE.
			
			WRITE: /5 'TYPE', 20 'ID', 40 'MESSAGE'.
			
			LOOP AT IT_RETURN INTO RETURN.
			
			WRITE: /5 RETURN-TYPE, 20 RETURN-ID, 40 RETURN-MESSAGE.
			
			ENDLOOP.
			ENDIF.

Build and Run the Quickstart
----------------------------

To build and run the quick start:

1. Change your working directory to the `sap-srfc-server-fuse` directory.
* Run `mvn clean install` to build the quick start.
* In your JBoss Fuse installation directory run, `./bin/fuse` to start the JBoss Fuse runtime.
* In the JBoss Fuse console, run `osgi:install -s mvn:org.fusesource/camel-sap` to install the JBoss Fuse SAP Synchronous Remote Function Call Server Camel component. Note the bundle number for the component bundle returned by this command.  
* In the JBoss Fuse console, run `osgi:install -s mvn:org.jboss.quickstarts.fuse/sap-srfc-server-fuse` to install the quick start. Note the bundle number for the quick start returned by this command.  
* In the JBoss Fuse console, run `log:tail` to monitor the JBoss Fuse log.
* Copy the response file (`src/data/response.xml`) in the project to the data directory(`work/sap-srfc-server-fuse/data`) of the quick start route.
* Invoke the camel route from SAP by running the `ZBAPI_FLCUST_GETLIST` program.
* In the console observe the request and response received and returned by the endpoint.  
* Compare this response with the received data displayed by the ABAP program.   

Stopping and Uninstalling the Quickstart
----------------------------------------

To uninstall the quick start and stop the JBoss Fuse run-time perform the following in the JBoss Fuse console:

1. Enter Ctrl-c to stop monitoring the JBoss Fuse log.
* Run `osgi:uninstall <quickstart-bundle-number>` providing the bundle number for the quick start bundle. 
* Run `osgi:uninstall <camel-sap-bundle-number>` providing the bundle number for the component bundle. 
* Run `osgi:shutdown -f` to shutdown the JBoss Fuse runtime.

