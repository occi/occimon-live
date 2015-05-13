# Summary #
This repository contains all is needed to run a simple demo of the OCCI Monitoring extension. The demo does not contain the front-end of the API, the one across which the user inputs the description of the monitoriing framework, and assumes that the description has already been loaded. Instead, the demo contains the engine that, using the description, creates the monitoring framework.

The monitored system is composed of three virtual machines

* **server** is the the web server that delivers the framework description as HTTP response content,
* **sensor** manages the monitoring stream, 
* **pc1** is the target of the monitoring activity. 

The whole system is generated by Vagrant with a single "up" command.

The monitoring consists of the measurement of the CPU load on the resource: the results are filtered using a Exponentially Weighted Moving Average, and delivered to the guest machine as UDP datagrams on port 8888.

#HOWTO#

You need to have VirtualBox and Vagrant installed on the guest machine to run the demo. The first time you need also a fast Internet connection to download approx. 500Mb for a disk image (Vagrant box)

* Clone this repository are step into the created directory
```
#!console
$ git clone https://bitbucket.org/augusto_ciuffoletti/occimon-demo.git
$ cd occimon-demo
```

* Run

```
#!console
$ vagrant up
```
The system is generated and booted. The first time you run this command a disk image will be downloaded, so expect a significant delay.

* Launch the minimalistic http server on **server**:

```
#!console
vagrant@server:~$ sudo service httpServer start 
Server pronto...
```

* Launch the metric container on **pc1**. Use the command "probe.sh" with the id of the monitored resource urn:uuid:c2222

```
#!console
me@mydesktop:~/demodir$ vagrant ssh pc1
...
vagrant@pc1:~$ ./probe.sh urn:uuid:c2222
Launch collector endpoint http://192.168.5.2:6789/urn:uuid:c2222
Metric container is ready (192.168.5.3:12312)
```
The metric container is now up and running, and waits for input from the sensor

* Open another terminal to launch the sensor container on **sensor**. Use the command "sensor.sh" with the id of the sensor resource(s): urn:uuid:s1111

```
#!console
me@mydesktop:~/demodir$ vagrant ssh sensor
...
vagrant@sensor:~$ ./sensor.sh urn:uuid:s1111
Launching sensor urn:uuid:s1111
Sensor receiving from TCP socket 192.168.5.6:52812
Launching remote collectors: [urn:uuid:2345]
Sensor launching collector from 192.168.5.3:12312
Logging true to my/log/file
ewma input: 1.0
sendudp: sending 1.0
Logging true to my/log/file
ewma input: 1.0
sendudp: sending 1.0
...
```
The sensor description is loaded from the web server. The sensor thread opens an input socket and fetches from the server the description of the collector. Using this information it instructs the metric container on **pc1** to start its activity. Data is received, processed with the EWMA filter, and delivered as a stream of datagrams to the guest machine.

* Now you can switch on the guest machine and observe the stream of UDP packets:
```
#!console
me@mydesktop:~/Desktop$ nc -ul 8888
Data: 2.3689306
Data: 2.2884052
Data: 2.2126167
Data: 2.1412864
...
```
#Under the hood...#

The configuration files are in the www directory: you can play with them. In order to see the results you need to stop the whole and restart the server, and the components.

To restart the server 
```
#!console
sudo service httpserver restart
```

#References#
* [research paper](http://eprints.adm.unipi.it/1913/1/paper.pdf) that illustrates the concepts behind the extension and how it works

Ciuffoletti, Augusto (2014) A simple and generic interface for a Cloud Monitoring System. In: CLOSER 2014 - 4th International Conference on Cloud Computing and Services Science, April 2-5 2014, Barcelona.

* [proposal](http://redmine.ogf.org/projects/occi-wg/repository/show?rev=monitoring) a draft of the OGF document that defines the extension
* [repository](https://bitbucket.org/augusto_ciuffoletti/occi-monitoring) of the Java code of the metricContainer and of the sensorContainer as an Eclipse project.

### Who do I talk to? ###

* augusto.at.di.unipi.it
* OCCI working group maillist
