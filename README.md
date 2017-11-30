# **JBoss EAP Upgrade Log** #

## Summary ##
    Day 1: Analysis and checking the application 
    Day 2: Understanding application dependencies
    Day 3: resolving runtime error for generated ws of phase.jar and system.jar. Troubleshooting generated login ws with native client (VB)


----------
# Details #

## Day 1 ##
### 1. Single EAR contains all application needed features ###
### 2. major entry portable at /adminConsole/ ###
### 3. service end points discovered: ###
 3.1 Remote EJB:
    mips.minds.service.client.ejb.ClientService
    mips.minds.service.disseminate.ejb.DisseminationService
    mips.minds.service.integrate.ejb.IntegrateService
    mips.minds.service.workflow.ejb.PhaseService
    mips.minds.service.workflow.ejb.PhaseIntegrationService
    mips.minds.service.workflow.ejb.PhaseMgtService
    mips.minds.service.system.ejb.SystemMS

 3.2 RMI Service:

    mips.minds.service.workflow.ejb.PhaseWebService
    mips.minds.service.system.ejb.SystemMSWebService
    mips.minds.service.bulletin.ejb.BulletinServiceWebService
    mips.minds.service.login.ejb.LoginWebService

### 4. EJBs discovered: ###

    BulletinMgtService	( mips.minds.service.bulletin.ejb.BulletinMgtServiceBean )	
    SignalService ( mips.minds.service.bulletin.ejb.SignalServiceBean )	
    BulletinService ( mips.minds.service.bulletin.ejb.BulletinServiceBean )	
    ClientService	( mips.minds.service.client.ejb.ClientServiceBean )	
    DisseminationService ( mips.minds.service.disseminate.ejb.DisseminationServiceBean )	
    MonitorService ( mips.minds.service.disseminate.ejb.MonitorService )	
    IntegrateService ( mips.minds.service.integrate.ejb.IntegrateServiceBean )	
    Login ( mips.minds.service.login.ejb.LoginBean )	
    PhaseService ( mips.minds.service.workflow.ejb.PhaseServiceBean )	
    PhaseIntegrationService ( mips.minds.service.workflow.ejb.PhaseIntegrationServiceBean )	
    PhaseMgtService ( mips.minds.service.workflow.ejb.PhaseMgtServiceBean )	
    SystemMS ( mips.minds.service.system.ejb.SystemMSBean )

### 5. Basic windup report generated ###
### 6. Challenges: ###
 6.1. Failed to connect to database 

 - Connection could be established in telnet, however, using jdbc connection is failed by TNS nor by thin 
 - Same connection setting is valid in other machines like Marco's
 - Tried ssh-tunneling but still failed

Day 2:
======

- Resolved the database connectivity issue
-  Identifying application depending files
- 3 folders are identified that contains custom build files, namely: conf, deploy and lib
- troubleshoot JAX-RPC error after cleaning the workspace up for bulletin, login, may other services still pending for next action



lib files
---------

Removed Jars

    antlr-2.7.5H3.jar: older version jar, removed
    bsh-1.3.0.jar : older version jar, removed
    cglib-2.1_2jboss.jar: older version jar, removed
    commons-collections-3.1.jar: older version jar, removed
    javax.servlet.jar: older version jar, removed
    javax.servlet.jsp.jar: older version jar, removed
    jbossmq.jar		: older version jar, removed
    mail-1.4.jar: older version jar, removed

keeped jar:

    derby-10.1.1.0.jar: unknown depending jar, keep
    fonts.jar: unknown depending jar, keep
    JCup.jar: unknown depending jar, keep
    joram-client.jar: unknown depending jar, keep
    joram-config.jar: unknown depending jar, keep
    joram-shared.jar: unknown depending jar, keep
    jpl-pattern.jar: unknown depending jar, keep
    jpl-util.jar: unknown depending jar, keep
    ojdbc6.jar: unknown depending jar, keep
    ojdbc7.jar: unknown depending jar, keep
    ow_monolog.jar: unknown depending jar, keep
    snmp-support.jar: unknown depending jar, keep
    

deploy files:
-------------

kept files:

    activemq-ra-5.2.rar: needed for activemq
    ds-console.war: needed for flex ds console
    minds2-3.1.5[Build001]_fb_test.ear: the main application
    ngm-jms-ds.xml: the jms datasource definition
    oracle-xa-ds.xml: the oracle datasource definition

removed file:

    jboss-aop.deployer: old version deployer
    jboss-ws4ee.sar: old version webservice lilbrary
    jbossweb-tomcat55.sar: old version web container library
    taskattach.war: under war
    uuid-key-generator.sar: old version generator
    quartz-ra.rar: conflicting quartz libraries

conf files:
-----------

kept files (application required configuration):

    channel.properties
    emailemail
    mailService.properties
    minds2.site.properties
    quartz.properties

Updated Webservices (JAX-RPC):
-----------

Done:

    bulletin.jar
    login.jar

Pending:

    system.jar
    phase.jar
    integrate.jar
    disseminate.jar
    client.jar

Day 3:
======

- troubleshoot JAX-RPC error after cleaning the workspace up for system, phase
- One Java code issue is found at mips.minds.service.workflow.ObjectNotFoundException with missing default constructor
- Newly generated webservices is able to call to the server backend. however, the method signature is changed
- Troubleshooting the way to solve it by changing the wsdl and jaxrpc mapping definition

Updated Java code for mips.minds.service.workflow.ObjectNotFoundException

    package mips.minds.service.workflow;
    
    /**
     * Exception will be thrown by PhaseServiceProvider to indicate business object
     * cannot be found in persistence store.
     */
    public class ObjectNotFoundException extends Exception {
    	public ObjectNotFoundException() {        
    		super("nothing doesn't exist.");
        }	
        public ObjectNotFoundException(String objType, long id) {
            super(objType + " with id[" + id + "] doesn't exist.");
        }
    }

Example of generated Calls:

    <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ejb="http://ejb.login.service.minds.mips/">
       <soapenv:Header/>
       <soapenv:Body>
          <ejb:wsLogout>
             <String_1>3123</String_1>
          </ejb:wsLogout>
       </soapenv:Body>
    </soapenv:Envelope>

Example of original call:

    <soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ejb="http://ejb.login.service.minds.mips/">
       <soapenv:Header/>
       <soapenv:Body>
          <ejb:wsLogout soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
             <in0 xsi:type="xsd:string">0ca7daa3-315f-4dee-8174-29dc9bc402c2</in0>
          </ejb:wsLogout>
       </soapenv:Body>
    </soapenv:Envelope>



