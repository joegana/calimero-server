Calimero KNXnet/IP Server
=========================

This feature branch provides the Calimero KNXnet/IP server for Java SE Embedded 8. The minimum required runtime environment is 
the profile [compact1](http://www.oracle.com/technetwork/java/embedded/resources/tech/compact-profiles-overview-2157132.html).

Download
--------

~~~ sh
# Either using git
$ git clone https://github.com/calimero-project/calimero-server.git

# Or using hub
$ hub clone calimero-project/calimero-server
~~~

The Calimero KNXnet/IP server requires the Calimero-core library and slf4j.

Supported Features
------------------

* Run your own KNXnet/IP server in software
* Turn a KNX interface into a KNXnet/IP server, e.g., KNX USB or KNX RF USB interfaces, EMI1/2 serial couplers 
* Intercept a KNXnet/IP connection (e.g., for monitoring/debugging purposes)
* Emulate a KNX network

### Client-side KNXnet/IP Support
* Discovery and Self-description
* Tunneling
* Routing
* Busmonitor (for KNX subnet interfaces that support busmonitor mode)
* Local device management

### KNX Subnet-side (Communication with the KNX Bus)
* KNXnet/IP
* KNX IP
* KNX RF USB
* KNX USB
* KNX FT1.2 Protocol (serial connections)

### Configuration
* XML configuration for startup
* KNX Interface Object Server during runtime, e.g., Device object, KNXnet/IP parameter object, cEMI server object, Group object table object

How-to & Examples
-----------------

The server provides a `Launcher`, together with a server configuration template `server-config.xml` (in the folder `resources`) to start the KNXnet/IP server. The launcher expects a URI or file name pointing to a server configuration.
Alternatively, one can also run the KNXnet/IP server and gateway in your Java code; see the implementation in `Launcher.java` as a guide.

First, make sure Java is installed correctly, and all required `jar` packages are available (`calimero-core`, `slf4j`). With the following command, the server should print a message that it expects a configuration file and quit:

#### Maven

~~~ sh
$ mvn exec:java -Dexec.mainClass=tuwien.auto.calimero.server.Launcher
~~~

#### Java

~~~ sh
# Either, assuming all jar dependencies are located in the current working directory
$ java -cp "./*" tuwien.auto.calimero.server.Launcher

# Or, a minimal working example with explicit references to jars (adjust as required)
$ java -cp "calimero-core-2.3-SNAPSHOT.jar:calimero-server-2.3-SNAPSHOT.jar:slf4j-api-1.7.7.jar:slf4j-simple-1.7.7.jar" tuwien.auto.calimero.server.Launcher
~~~

### Start Server with Configuration

*Before trying the examples below with a configuration, make sure the configuration is appropriate for your KNX setup!*

Using maven

~~~ sh
$ mvn exec:java -Dexec.mainClass=tuwien.auto.calimero.server.Launcher -Dexec.args=resources/server-config.xml
~~~

or Java (make sure any referenced files in the folder `resources` are found, or copy them into the current working directory.)

~~~ sh
# Either, assuming all `jar` dependencies are located in the current working directory
$ java -cp "./*" tuwien.auto.calimero.server.Launcher server-config.xml
~~~

On the terminal, the running server instance can be stopped by typing "stop".


### Launcher Configuration

Elements and attributes of `server-config.xml`:

* `<knxServer name="knx-server" friendlyName="My KNXnet/IP Server">` (required): the server ID (for logging etc.) and the KNXnet/IP friendly name (for discovery & self-description)
* `<discovery listenNetIf="all" outgoingNetIf="all" activate="true"/>` (optional attributes): the network interfaces to listen to KNXnet/IP discovery requests, as well as the network interfaces to answers to requests, e.g., `"all"`, `"any"`, or `"lo,eth0,eth1"`. The attribute `activate` allows to disable KNXnet/IP discovery & self-description.
* `<serviceContainer>` (1..*): specify a server service container, i.e., the client-side endpoint for a KNX subnet. Attributes: 
	- `activate`: enable/disable the service container, to load/ignore that container during server startup
	- `routing`: serve KNXnet/IP routing connections (set `true`) or disable KNXnet/IP routing (set `false`)
	- `allowNetworkMonitoring`: allow connection requests in KNX busmonitor layer
	- `udpPort`: (optional) UDP port of the control endpoint to listen for incoming connection requests of that service container, defaults to KNXnet/IP standard port "3671"
	-  `listenNetIf`: (optional): network interface to listen for connection requests, e.g., `"any"` or `"eth1"`, defaults to host default network interface
	- `reuseCtrlEP`: reuse the KNXnet/IP control endpoint (UDP/IP) for subsequent tunneling connections. If reuse is enabled, no list of additional KNX individual addresses is required (see below). Reuse is only possible if the individual address is not yet assigned to a connection, and if KNXnet/IP routing is not activated.

* `<knxAddress type="individual">7.1.1</knxAddress>`: the individual address of the service container (has to match the KNX subnet!)
* `<routingMcast>` (optional): the multicast group used by the service container with KNXnet/IP routing, defaults to the IP multicast address 224.0.23.12 
* `<knxSubnet>` settings of the KNX subnet the service container shall communicate with. The `knxSubnet` element text contains identifiers specific to the KNX subnet interface type, i.e., IP address[:port], or USB interface name/ID, constructor arguments, ... Attributes:
	- `type`: interface type to KNX subnet, one of "ip", "knxip", "usb", "ft12", "virtual", "user-supplied"
	- `medium` (optional): KNX transmission medium, one of "tp1" (default), "pl110", "pl132", "knxip", "rf"
	- `listenNetIf` (KNX IP only): network interface for KNX IP communication
	- `domainAddress` (open media only): domain address for power-line or RF transmission medium
	- `class` (user-supplied KNX subnet type only): class name of a user-supplied KNXNetworkLink to use for subnet communication

* `<groupAddressFilter>`: Contains a (possibly empty) list of KNX group addresses, which represents the server group address filter applied to messages for that service container. An empty filter list does not filter any messages. Only messages with their group address in the filter list will be forwarded. If you specify a filter, you probably also want to add the broadcast address `0/0/0`. 
* `<additionalAddresses>`: Contains a (possibly empty) list of KNX individual addresses, which are assigned to KNXnet/IP tunneling connections. An individual address has to match the KNX subnet (area, line), otherwise it will not be used! If no additional addresses are provided, the service container individual address is used, and the maximum of open tunneling connections at a time is limited to 1. 

### Configuration Examples for KNX subnets

* Turn a PL110 USB interface into a KNXnet/IP server, the USB interface name matches 'busch-jaeger'  

	`<knxSubnet type="usb" medium="pl110" domainAddress="6f">busch-jaeger</knxSubnet>`

* Use the KNXnet/IP server to communicate with a KNX IP network

	`<knxSubnet type="knxip" listenNetIf="eth4">224.0.23.12</knxSubnet>`

* Load a user-supplied KNXNetworkLink class to communicate with the KNX subnet (the element text is parsed into constructor arguments of type String, using separators ',' and '|')

	`<knxSubnet type="user-supplied" class="my.knx.SubnetLink">o1,i2|i4</knxSubnet>`

* Provide a KNXnet/IP server for a KNX USB RF connection, using the USB vendor:product ID

	`<knxSubnet type="usb" medium="rf" domainAddress="000000004b01">0409:005a</knxSubnet>`



Logging
-------

Calimero KNXnet/IP server uses the [Simple Logging Facade for Java (slf4j)](http://www.slf4j.org/). Bind any desired logging frameworks of your choice. The default maven dependency is the [Simple Logger](http://www.slf4j.org/api/org/slf4j/impl/SimpleLogger.html). It logs everything to standard output. The simple logger can be configured via the file `simplelogger.properties`, JVM system properties, or `java` command line options, e.g., `-Dorg.slf4j.simpleLogger.defaultLogLevel=warn`.

