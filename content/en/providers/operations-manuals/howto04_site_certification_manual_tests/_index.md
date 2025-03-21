---
title: HOWTO04 Site Certification Manual tests
weight: 130
type: "docs"
description: "Manual testing of services offered by a site"
---

This page provides instructions on how to test manually the functionality of the
grid and cloud services offered by a site. These checks are executed by the EGI
Operations Team for sites that want to be formally included into EGI Production
infrastructure. The site must successfully pass either the grid or the cloud
certification tests to become part of the EGI Production infrastructure.

## Check the functionality of the grid elements

Be sure that the site's GIIS URL is contained in the Top level BDII/Information
System your NGI will use for your certification.

Note that the examples here use the Italian NGI and sites. Please substitute
with **YOUR OWN** NGI and site credentials when running the test.

### Cream-CE checks

Open in your browser the following URL:
`https://<hostname-of-cream-ce>:8443/ce-cream/services`

A page with link to the CREAM WSDL should be shown.

Try a gsiftp (e.g. using `globus-url-copy` or `uberftp`) towards that CREAM CE.
E.g.:

```shell
$ globus-url-copy \
    gsiftp://<hostname-of-cream-ce>/opt/glite/yaim/etc/versions/glite-yaim-core \
    file:/tmp/yaim-version-test
```

Try the following command:

```shell
$ glite-ce-allowed-submission <hostname-of-cream-ce>:8443
```

It should report: `Job Submission to this CREAM CE is enabled`.

Try a submission to Cream-CE using the `glite-ce-job-submit` command, e.g.:

```shell
$ cat sleep.jdl

[
executable="/bin/sleep";
arguments="1";
]
```

```shell
$ glite-ce-job-submit -a -r <hostname-of-cream-ce>:8443/<queue> sleep.jdl
```

```shell
$ glite-ce-job-submit -a -r ce-cr-02.ts.infn.it:8443/cream-lsf-cert sleep.jdl

https://ce-cr-02.ts.infn.it:8443/CREAM127814374
```

Check the status of that job, which eventually should be `DONE-OK`:

```shell
$ glite-ce-job-status https://ce-cr-02.ts.infn.it:8443/CREAM127814374

2010-07-27 11:55:37,986 WARN - No configuration file suitable for loading. Using built-in configuration

******  JobID=[https://ce-cr-02.ts.infn.it:8443/CREAM127814374]
Status        = [DONE-OK]
ExitCode      = [0]
```

Try a submission to that CE using the `glite-ce-job-submit` command, and then
tries to cancel it (using the `glite-ce-job-cancel` command):

```shell
$ cat sleep2.jdl

[
executable="/bin/sleep";
arguments="1000";
]
```

```shell
$ glite-ce-job-submit -a -r cecream-cyb.ca.infn.it:8443/cream-lsf-poncert sleep2.jdl

https://cecream-cyb.ca.infn.it:8443/CREAM126335182
```

```shell
$ glite-ce-job-cancel https://cecream-cyb.ca.infn.it:8443/CREAM126335182
```

```shell
$ glite-ce-job-status https://cecream-cyb.ca.infn.it:8443/CREAM126335182

2010-07-27 12:18:26,973 WARN - No configuration file suitable for loading. Using built-in configuration

******  JobID=[https://cecream-cyb.ca.infn.it:8443/CREAM126335182]
Status        = [CANCELLED]
ExitCode      = []
Description   = [Cancelled by user]
```

### ARC CE checks

A first test can be done using ARC's `ngstat` command:

```shell
$ /usr/bin/ngstat -q -l -c <CE hostname> -t 20
...
... plenty of output
...
```

If a
[monitoring host of your NGI](http://wiki.nordugrid.org/wiki/Service_Monitoring)
is available, then the probes can easily be executed from there:

Check the status of the CE with:

```shell
$ /usr/libexec/grid-monitoring/probes/org.ndgf/ARCCE-status \
    -H <CE hostname> -x /etc/nagios/globus/userproxy.pem-ops

Status is active
```

Test gsiftp:

```shell
$ /usr/libexec/grid-monitoring/probes/org.ndgf/ARCCE-auth -H <CE hostname> \
    -x /etc/nagios/globus/userproxy.pem-ops

gsiftp OK
```

Test the versions of the CA's:

```shell
$ /usr/libexec/grid-monitoring/probes/org.ndgf/ARCCE-caver -H <CE hostname> \
    -x /etc/nagios/globus/userproxy.pem-ops

version = 1.38 - All CAs present
```

Check the versions of ARC and Globus:

```shell
$ /usr/libexec/grid-monitoring/probes/org.ndgf/ARCCE-softver \
    -H <CE hostname> \
    -x /etc/nagios/globus/userproxy.pem-ops

nordugrid-arc-0.8.3.1, globus-5.0.3
```

Copy a file:

```shell
$ /usr/libexec/grid-monitoring/probes/org.ndgf/ARCCE-gridftp -H <CE hostname> \
    -x /etc/nagios/globus/userproxy.pem-ops

Job finished successfully
```

Submit a test job:

```shell
$ /usr/libexec/grid-monitoring/probes/org.ndgf/ARCCE-jobsubmit \
    -H <CE hostname> \
    --vo ops -x /etc/nagios/globus/userproxy.pem-ops

Job submission successful
```

Check the LFC:

```shell
$ /usr/libexec/grid-monitoring/probes/org.ndgf/ARCCE-lfc -H <CE hostname> \
    -x /etc/nagios/globus/userproxy.pem-ops

Job finished successfully
```

Check the SRM:

```shell
$ /usr/libexec/grid-monitoring/probes/org.ndgf/ARCCE-srm -H <CE hostname> \
    -x /etc/nagios/globus/userproxy.pem-ops

Job finished successfully
```

Before continuing, you may want to make sure that the probes for all services
which the CE intends to offer, do actually succeed.

### Storage Element (SE) checks

Check if `gridftp` server on SE works:

```shell
$ uberftp inaf-se-01.ct.pi2s2.it
```

For STORM SE: check if SRM client works (on the published information you can
find the right port to use)

```shell
$ /opt/storm/srm-clients/bin/clientSRM \
    ping -e httpg://sunstorm.cnaf.infn.it:8444

============================================================
Sending Ping request to: httpg://sunstorm.cnaf.infn.it:8444
============================================================ Request status:
statusCode="SRM_SUCCESS"(0) explanation="SRM server successfully contacted"
============================================================ SRM Response:
versionInfo="v2.2" otherInfo (size=2) [0] key="backend_type"
[0] value="StoRM" [1] key="backend_version"
[1] value="<FE:1.5.0-1.sl4><BE:1.5.3-4.sl4>"
============================================================
```

Try to write on SE. Be sure your UI is pointing to an IS the SE is contained in
(you may use your certification BDII)

1\) Setting a top-bdii that is publishing the SE you have to test

```shell
$ export LCG_GFAL_INFOSYS=<TopBDII hostname>:2170
```

2\) Copy a file from the local filesystem to the SE, registering it in the LFC.
This command output will return a SURL that you can use latter for other tests.

A SURL is a path of the type:
`srm://srm01.ncg.ingrid.pt/ibergrid/iber/generated/2011-02-01/file4034a935-8d7a-48f4-914f-16f2634d4802`

```shell
$ lcg-cr -v --vo <VO>-d<Your SE> \
    -l lfn:/grid/<VO>/test.txt file:</path/to/your/local/file>
```

3\) Create a new replica in other SE (to check the 3rd party transfer between 2
SEs)

```shell
$ lcg-rep -v --vo <VO>-d<Other SE> <SURL>
```

4\) List Replicas

```shell
$ lcg-lr -v --vo <VO> lfn:/grid/<VO>/test.txt
```

5\) Delete all replicas

```shell
$ lcg-del -v --vo <VO>-a<guid>
```

### Job submission

Submit a test job to **Cream-CE** through the **WMS**, i.e. using the
`glite-wms-job-submit` command. In case, submit a mpi test job. The NGI_IT
certification WMS is `gridit-cert-wms.cnaf.infn.it`.

### Registration into 1st level HLR

**NOTE: this step is needed if your infrastructure uses DGAS as accounting
system**

After the site entered in production, it needs to register the site resources in
the HLR. Ask the site admins to open a ticket towards the HLR administrators,
passing them the following information:

- grid queues names, in the form:
  - `gridit-ce-001.cnaf.infn.it:2119/jobmanager-lcgpbs-cert`
- not-grid queues names, in the form:
  - `hostname:queue`
- Name, surname ad certificate subject of each site admin
- Certificate subject of Computing Element

Eventually, the site admins have to open a ticket to DGAS support unit asking to
enable the forwarding of accounting data from the 2° level HLR to APEL.

### Certification Job

The [test job](https://wiki.egi.eu/wiki/Certification_Job_template) checks
several things, like the environment on WN and installed RPMs. Moreover it
performs some replica management tests. With a `grep TEST` you may get a summary
of the results: in case of errors, you have to see in detail what is gone wrong!

As already said, if the site supports any flavour of MPI, launch a MPI test job,
like [this](https://wiki.egi.eu/wiki/MPI_Certification_Job_template).

Don't forget to set a reasonable value in `CPUNumber`: most important is that
your job starts running quickly.

If you want less stuff in the `.out` and `.err` files, in the file
`mpi-start-wrapper.sh` comment the line

```shell
$ export I2G_MPI_START_DEBUG=1
```

A successful output will look like the following one (extract)

```shell
[...]
mpi-start [DEBUG ]: using user supplied startup : '/opt/mpich-1.2.7p1/bin/mpirun '
mpi-start [DEBUG ]: => MPI_SPECIFIC_PARAMS=
mpi-start [DEBUG ]: => I2G_MPI_PRECOMMAND=
mpi-start [DEBUG ]: => MPIEXEC=/opt/mpich-1.2.7p1/bin/mpirun
mpi-start [DEBUG ]: => I2G_MACHINEFILE_AND_NP=-machinefile /tmp/tmp.iBypc12521 -np 6
mpi-start [DEBUG ]: => I2G_MPI_APPLICATION=/home/dteam022/globus-tmp.t3-wn-13.11955.0/https_3a_2f_2falbalonga.cnaf.infn.it_3a9000_2fI06uWaKi1evxL3tTF-DTOg/hello
mpi-start [DEBUG ]: => I2G_MPI_APPLICATION_ARGS=
mpi-start [DEBUG ]: /opt/mpich-1.2.7p1/bin/mpirun -machinefile /tmp/tmp.iBypc12521 -np 6 /home/dteam022/globus-tmp.t3-wn-13.11955.0/https_3a_2f_2falbalonga.cnaf.infn.it_3a9000_2fI06uWaKi1evxL3tTF-DTOg/hello
Process 4 on t3-wn-37.pn.pd.infn.it out of 6
Process 3 on t3-wn-34.pn.pd.infn.it out of 6
Process 1 on t3-wn-13.pn.pd.infn.it out of 6
Process 2 on t3-wn-34.pn.pd.infn.it out of 6
Process 5 on t3-wn-37.pn.pd.infn.it out of 6
Process 0 on t3-wn-13.pn.pd.infn.it out of 6
[...]
```

### Globus checks

> These checks should be executed depending on the services registered in GOCDB
> under a Resource Centre. Not all services are compulsory for a RC, but upon
> registration of new ones, the corresponding tests should be executed.

#### GSISSH

Initialize grid proxy and check if GSISSH works:

```shell
$ grid-proxy-init
$ gsissh USER@HOST -p 2222 /bin/date
(Debug with: USER@HOST -vvv -p 2222 /bin/date)
```

#### GridFTP

Check if upload works:

```shell
$ globus-url-copy file:/tmp/test.txt gsiftp://HOST:2811/tmp/test.txt
(Debug with: globus-url-copy -dbg -v -vb file:/tmp/test.txt gsiftp://HOST:2811/tmp/test.txt)
```

Check if download works:

```shell
$ globus-url-copy gsiftp://HOST:2811/tmp/test.txt file:/tmp/test.txt
(Debug with: globus-url-copy -dbg -v -vb gsiftp://HOST:2811/tmp/test.txt file:/tmp/test.txt)
```

Delete the remote file:

```shell
$ uberftp HOST 'rm /tmp/test.txt'
(Debug with: uberftp HOST 'rm /tmp/test.txt' -debug 3)
```

#### GRAM

Check authentication:

```shell
$ globusrun -a -r HOST:2119
```

Check job submission:

```shell
$ globusrun -s -r HOST:2119 "&(executable="/bin/date")"
```

### Unicore checks

This testing manual assumes that the test instance has not been added to the
“Global” registry. “Global” registry does not have to be global (for the whole
infrastructure) - is a register used by a group of site which work together. For
example each Resource Infrastructure Provider can have own “Global” registry.

It is suggested to add the instance to the “Global” registry only if it was
tested and works properly. For this reason this instruction refers to the local
registry.

#### Preliminary testing

After installation and configuration, start all the services and see if
functioning properly. To avoid errors/warnings in the logs first start the TSI
and the Gateway and then the Unicore/X (requires two other servers to operate).
The first step of verification is to verify proper configuration of log files
for all services whether they running. Logs for Unicore/X and Gateway are in
standard locations `/var/log/unicore/unicorex/unicorex.log` and
`/var/log/unicore/gateway/gateway.log`. In the case where there is no log file,
check the file `/var/log/unicore/unicorex/unicorex-startup.log` or
`/var/log/unicore/gateway/gateway-startup.log` - those file contain the servers'
standard output  output, and can be useful in case of generic, system-wide
issues as missing  Java virtual machine. Log files should be checked carefully
for warnings and errors. They should show only the information about the start
of the service, without any warnings (the WARN label) or errors (the ERROR
label). In case of problems, you should proceed according to the information
found in the log files. If they are unclear you should increase logging detail
(for Unicore/X and Gateway). This is set in the file
`/etc/unicore/gateway/logging.properties` and
`/etc/unicore/unicorex/logging.properties`. UNICORE uses log4j logging
subsystem. When you change the login parameters is not required to restart the
component. After the successful initialization of all services you can begin to
test them in practice. Please connect to the site via any UNICORE client (URC or
UCC). Since the registration of newly created VSite was initially turned off in
the global registry, you should use the local registry. The local registry
address is:
`https://GATEWAY-ADDESS/VSITE_NAME/services/Registry?res=default_registry`. Is
recommended for test script execution, which displays the user. This should be a
user associated with the certificate.

#### Testing using the URC

1. Testing should start from setting up the user's credentials,
2. A local registry should be added in URC Grid Browser view..
3. The registry contents should be listed, by double clicking on its node. It is
   worth to enable the display of all sites by clicking on the Grid Browser the
   "Show" button and selecting from the list "All services". If you see a red
   cross on the service, click on it and see the details of the error message in
   the URC and the error on the server side.
4. If all services are available, you can send the job. At the same time it is
   recommended to monitor the logs Unicore/X and TSI for errors.

#### Testing using the UCC

1. Configure UCC credentials
2. Configure the registry in UCC preferences file (the registry property).
3. Invoke:

```shell
/ucc shell
ucc> connect
You can access 1 target system(s).
ucc> list-sites
VSITE_NAME https://GATEWAY_ADDRESS/VSITE_NAME/services/TargetSystemService?res=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
ucc> list-storages
SHARE https://GATEWAY_ADDRESS/VSITE_NAME/services/services/StorageManagement?res=default_storage
ucc> ls u6://SHARE
/a5063ea0-ecbe-4097-9abc-f55ec9437376
/3f501d37-5851-4c9e-a1da-5ad7b9f16633
/3bf169c2-2149-4564-b827-0b6560a3dd35
...
ucc> list-applications
Applications on target system <VSITE_NAME>
R 2.10.0
BLAST 2.2.22
POVRay 3.6.1
...
```

We should get a message similar to the above. Then test the file transfer:

```shell
ucc> put-file -s LOCAL_FILE_PATH \
    -t https://GATEWAY_ADDRESS/VSITE_NAME/services/StorageManagement?res=default_storage#TARGET_FILE_NAME
```

And job submission:

```shell
ucc> run -s VSITE_NAME JOB_FILE_PATH.u
SUCCESSFUL exit code: 0
```

If an error occurs, you can on each of these commands add the "-v" flag, what
increases  UCC verbosity. As in the URC case it is advised to simultaneously
monitor Unicore / Xa and TSI log files.

##### After testing

If testing was successful, you can unlock the registration system in the global
registry.

## QCG checks

### QCG Computhng checks

The presented tests of QCG-Computing service use the qcg-comp, the client
program for QCG-Computing, that may be installed from provided RPMS. In order to
connect to QCG-Computing the grid proxy must be created.

**Generate user’s proxy:**

```shell
$ grid-proxy-init

Your identity: /C=PL/O=GRID/O=PSNC/CN=Mariusz Mamonski
Enter GRID pass phrase for this identity:
Creating proxy ............................ Done
Your proxy is valid until: Fri Jun 10 06:23:32 2011
```

**Query the QCG-Computing service:**

```shell
# the xmllint is used only to present the result in more pleasant way
$ qcg-comp -G | xmllint --format -

<bes-factory:FactoryResourceAttributesDocument xmlns:bes-factory="http://schemas.ggf.org/bes/2006/08/bes-factory">
… a lot of information …   
</bes-factory:FactoryResourceAttributesDocument>
```

**Submit a sample job:**

```shell
$ qcg-comp -c -J /opt/plgrid/qcg/share/qcg-comp/doc/examples/date.xml

Activity Id: ccb6b04a-887b-4027-633f-412375559d73
```

**Query its status:**

```shell
$ qcg-comp -s -a ccb6b04a-887b-4027-633f-412375559d73

status = Executing

$ qcg-comp -s -a ccb6b04a-887b-4027-633f-412375559d73

status = Finished
exit status = 0
```

### QCG Notification checks

The tests of QCG-Notification require `qcg-ntf-client` program to be installed
in a system. The program is provided in RPM package.

**Create a sample subscription:**

```shell
$ qcg-ntf-client -d \
    -S "cons=http://127.0.0.1:2212 top=http://schemas.qoscosgrid.org/comp/2011/04/notification/topic;//*;Full"

...
INF May 17 14:15:51 1128       0xa0262720 [qcg-client-gsoa] Subscribed, subRef: '810917963'
...
```

**Remove the created subscription:**

```shell
$ qcg-ntf-client -d -U "id=810917963"

...
INF May 17 14:41:48 3318       0xa0262720 [qcg-client-gsoa] Unsubscribed: '810917963'
…
```

**Checking the connection with QCG-Computing:** In one shell run ‘tail -f’ on
the QCG-Computing log file and in the other try to submit a sample job using the
qcg-comp program (as described above). Check the tail output if there are no
error messages on sending notifications. E.g. the following lines means that the
connection problems occurred:

```shell
$ tail -f /opt/qcg/var/log/qcg-comp/qcg-compd.log

INF Oct 04 10:55:33 18929   0x2adadc2abe30 [notification_ws] Sending notify: 320f014c-3181-4daf-bbd9-1824b7d8216a -> Queued
NOT Oct 04 10:55:33 18929   0x2adadc2abe30 [.....ntf_client] FaultCode: 'SOAP-ENV:Client'
NOT Oct 04 10:55:33 18929   0x2adadc2abe30 [.....ntf_client] FaultString: 'smcm:ActivityState'
NOT Oct 04 10:55:33 18929   0x2adadc2abe30 [.....ntf_client] FaultDetail: '<SOAP-ENV:Detail xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/">connect failed in tcp_connect()</SOAP-ENV:Detail>'
ERR Oct 04 10:55:33 18929   0x2adadc2abe30 [notification_ws] Failed to send notification to http://grass1.man.poznan.pl:19011/
```

### QCG Broker checks

The basic tests of QCG-Broker service may be proceeded with help of
`qcg-simple-client`, the software that provides a set of commands for
interaction with QCG-Broker. `qcg-simple-client` may be installed from RPMs.

**Create a sample job description:**

```shell
$ cat > sleep.qcg << EOF
#!/bin/bash

#QCG queue=plgrid
#QCG host=nova.wcss.wroc.pl
#QCG persistent

sleep 30
EOF
```

**Submit a job:**

```shell
$ qcg-sub sleep.qcg

https://qcg-broker.man.poznan.pl:8443/qcg/services/
/C=PL/O=GRID/O=PSNC/CN=qcg-broker/qcg-broker.man.poznan.pl
Your identity: C=PL,O=GRID,O=PSNC,CN=Bartosz Bosak
Enter GRID pass phrase for this identity:
Creating proxy, please wait...
Proxy verify OK
Your proxy is valid until Tue Mar 12 14:50:27 CET 2013
UserDN = /C=PL/O=GRID/O=PSNC/CN=Bartosz Bosak
ProxyLifetime = 24 Days 23 Hours 59 Minutes 58 Seconds

jobId = J1360936230540__0152
```

**Check the job statuses:**

```shell
$ qcg-info

https://qcg-broker.man.poznan.pl:8443/qcg/services/
/C=PL/O=GRID/O=PSNC/CN=qcg-broker/qcg-broker.man.poznan.pl
UserDN = /C=PL/O=GRID/O=PSNC/CN=Bartosz Bosak
ProxyLifetime = 24 Days 23 Hours 59 Minutes 49 Seconds

Command translated to:
"task_info" "J1360936230540__0152" "task"
Note:
UserDN: /C=PL/O=GRID/O=PSNC/CN=Bartosz Bosak
TaskType: SINGLE
SubmissionTime: Fri Feb 15 14:50:31 CET 2013
FinishTime:
ProxyLifetime: PT0S
Status: PREPROCESSING
StatusDesc:
StartTime: Fri Feb 15 14:50:33 CET 2013

Allocation:
HostName: nova.wcss.wroc.pl
ProcessesCount: 1
ProcessesGroupId:
Status: PREPROCESSING
StatusDescription:
SubmissionTime: Fri Feb 15 14:50:32 CET 2013
FinishTime:
LocalSubmissionTime: Fri Feb 15 14:50:37 CET 2013
LocalStartTime:
LocalFinishTime:

$ qcg-info

https://qcg-broker.man.poznan.pl:8443/qcg/services/
/C=PL/O=GRID/O=PSNC/CN=qcg-broker/qcg-broker.man.poznan.pl
UserDN = /C=PL/O=GRID/O=PSNC/CN=Bartosz Bosak
ProxyLifetime = 24 Days 23 Hours 59 Minutes 23 Seconds

Command translated to:
"task_info" "J1360936230540__0152" "task"
Note:
UserDN: /C=PL/O=GRID/O=PSNC/CN=Bartosz Bosak
TaskType: SINGLE
SubmissionTime: Fri Feb 15 14:50:31 CET 2013
FinishTime:
ProxyLifetime: PT0S
Status: RUNNING
StatusDesc:
StartTime: Fri Feb 15 14:50:33 CET 2013

Allocation:
HostName: nova.wcss.wroc.pl
ProcessesCount: 1
ProcessesGroupId:
Status: RUNNING
StatusDescription:
SubmissionTime: Fri Feb 15 14:50:32 CET 2013
FinishTime:
LocalSubmissionTime: Fri Feb 15 14:50:37 CET 2013
LocalStartTime: Fri Feb 15 14:50:47 CET 2013
LocalFinishTime:
$ qcg-info
https://qcg-broker.man.poznan.pl:8443/qcg/services/
/C=PL/O=GRID/O=PSNC/CN=qcg-broker/qcg-broker.man.poznan.pl
UserDN = /C=PL/O=GRID/O=PSNC/CN=Bartosz Bosak
ProxyLifetime = 24 Days 23 Hours 56 Minutes 10 Seconds

Command translated to:
"task_info" "J1360936230540__0152" "task"
Note:
UserDN: /C=PL/O=GRID/O=PSNC/CN=Bartosz Bosak
TaskType: SINGLE
SubmissionTime: Fri Feb 15 14:50:31 CET 2013
FinishTime: Fri Feb 15 14:52:17 CET 2013
ProxyLifetime: PT0S
Status: FINISHED
StatusDesc:
StartTime: Fri Feb 15 14:50:33 CET 2013

Allocation:
HostName: nova.wcss.wroc.pl
ProcessesCount: 1
ProcessesGroupId:
Status: FINISHED
StatusDescription:
SubmissionTime: Fri Feb 15 14:50:32 CET 2013
FinishTime: Fri Feb 15 14:52:12 CET 2013
LocalSubmissionTime: Fri Feb 15 14:50:37 CET 2013
LocalStartTime: Fri Feb 15 14:50:47 CET 2013
LocalFinishTime: Fri Feb 15 14:52:09 CET 2013
```

## Check the functionality of the cloud elements

Sites can provide any (not necessarily all) of the interfaces listed below:

- OCCI for VM Management
- OpenStack Compute for VM Management
- CDMI for Object Storage

### Cloud Compute (OCCI/OpenStack) checks prerequisites

#### AppDB integration

_NOTE:_ Prerequisite: run the following commands is the installation of the EGI
rOCCI CLI environment, according to
[this guide](https://wiki.egi.eu/wiki/Fedcloud-tf:CLI_Environment).

Go to AppDB and look for a OS image member of the `fedcloud.egi.eu` VO (all
sites should support), e.g.
[EGI CentOS 7 image](https://appdb.egi.eu/store/vappliance/egi.centos.7)

Check that the site is visible in to the AppDB "Availability and Usage" panel
for the image. If not, probably the site has not registered the FedCloud VO into
their middleware (vmcatcher) or it did not properly configured the BDII provider
script.

From that "Availability and Usage" panel, click on the Site name, then on the
latest VM Image version, select a resource template (preferably with the
smallest quantity of resources (RAM & CPU)) and click on the "get IDs" button on
the right of the resource template. You will get the "Site Endpoint", "Template
ID" and "OCCI ID". Save these values since they will be needed in the next
steps.

#### Credentials

Generate a set of keys for your user (it is not required to set a phassphrase
for the keys, since these are just temporary keys for the test), make sure to
set key permissions to `400`:

```shell
$ ssh-keygen -t rsa -b 2048 -f tempkey

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in tempkey.
Your public key has been saved in tempkey.pub.
The key fingerprint is:
(...)

$ chmod 400 tempkey
```

Create a simple contextualization script, to setup access keys on the machine
and test contextualization

```shell
$ cat << EOF > ctx.txt
Content-Type: multipart/mixed; boundary="===============4393449873403893838=="
MIME-Version: 1.0

--===============4393449873403893838==
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="deploy.sh"

#!/bin/bash
echo "OK" > /tmp/deployment.log

--===============4393449873403893838==
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="userdata.txt"

#cloud-config
users:
- name: testadm
  sudo: ALL=(ALL) NOPASSWD:ALL
  lock-passwd: true
  ssh-import-id: testadm
  ssh-authorized-keys:
   - `cat tempkey.pub`

--===============4393449873403893838==--
EOF
```

Create a proxy in **RFC** format for your tests:

```shell
$ voms-proxy-init -rfc -voms fedcloud.egi.eu

Your identity: /DC=es/DC=irisgrid/O=ifca/CN=Enol-Fernandez-delCastillo
Creating temporary proxy .......................................................................... Done
Contacting voms2.grid.cesnet.cz:15002 [/DC=org/DC=terena/DC=tcs/OU=Domain Control Validated/CN=voms2.grid.cesnet.cz] "fedcloud.egi.eu" Done
Creating proxy .............................................................................. Done

Your proxy is valid until Fri Nov 14 04:59:26 2014
```

### OCCI checks (eu.egi.cloud.vm-management.occi service type)

_NOTE:_ Prerequisite: run the following commands is the installation of the EGI
EGI CLI environment, according to
[this guide](https://wiki.egi.eu/wiki/Fedcloud-tf:CLI_Environment).

Describe OS templates (`occi_endpoint` is the one retrieved from AppDB). Check
also that the OCCI ID provided by in AppDB is listed:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X -a describe -r os_tpl

[...]
[[/_http://occi.carach5.ics.muni.cz/occi/infrastructure/os_tpl#uuid_ubuntu_server_14_04_lts_fedcloud_dukan_84_| http://occi.carach5.ics.muni.cz/occi/infrastructure/os_tpl#uuid_ubuntu_server_14_04_lts_fedcloud_dukan_84 ]]
title: Ubuntu-Server-14.04-LTS@fedcloud-dukan
term: uuid_ubuntu_server_14_04_lts_fedcloud_dukan_84
location: /mixin/os_tpl/uuid_ubuntu_server_14_04_lts_fedcloud_dukan_84/
[...]
```

Describe the resource templates, and check that the resource ID provided but
AppDB is returned:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X -a describe \
    -r resource_tpl
[...]
[[/_http://schema.fedcloud.egi.eu/occi/infrastructure/resource_tpl#small_| http://schema.fedcloud.egi.eu/occi/infrastructure/resource_tpl#small ]]
title: Small Instance - 1 core and 2 GB RAM term: small
location: /mixin/resource_tpl/small/
```

Start a VM (using the OCCI ID and Resource ID as provided by AppDB for the
mixins, and for context the file created above). The returned ID will be used in
the commands below:

```shell
$ occi -e  <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a create -r compute -M  <resource_id> -M <occi_id> \
    -t occi.core.title="My OCCI VM" \
    -T user_data= <file://$PWD/ctx.txt>

https://carach5.ics.muni.cz:11443/compute/46934
```

Check that the VM is active by describing it (it may take a few minutes):

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X -a describe -r <vm_id>

[...]
occi.compute.state = active
[...]
occi.networkinterface.address = 147.251.3.62
[...]
```

If the VM does not have a public IP, you will need to get an IP for it

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X -a link \
    -r <vm_id> -j /network/public
```

ssh to the machine to the provided IP (the options avoid problems when different
VMs have the same IP, don't use them in production) and check that
contextualization script was executed:

```shell
$ ssh -i tempkey -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
    testadm@ <ip_addr> "sudo cat /tmp/deployment.log"

OK
```

Create a block storage entity of 1 GB and save the ID returned:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a create -r storage -t occi.storage.size=num(1) \
    -t occi.core.title=storage_test

https://carach5.ics.muni.cz:11443/storage/296
```

Describe the newly created storage ID:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a describe -r <storage_id>

[...]
occi.storage.size = 1.0
occi.storage.state = online
[...]
```

Link that storage to the running VM:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a link -r <vm_id> -j <storage_id>

https://carach5.ics.muni.cz:11443/link/storagelink/compute_46936_disk_2
```

Check the device where the storage is attached:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a describe -r <storage_link_id>

[...]
occi.storagelink.deviceid = /dev/xvdb
[...]
```

And login into the machine to create a filesystem, mount it, create a file, and
umount:

```shell
$ ssh -i tempkey -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
    testadm@ <ip_addr>

testadm $ sudo mke2fs <device_id>
mke2fs 1.42.9 (4-Feb-2014) Filesystem label= OS type: Linux
Block size=4096 (log=2) Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks 65536 inodes, 262144 blocks
13107 blocks (5.00%) reserved for the super user First data block=0
Maximum filesystem blocks=268435456 8 block groups
32768 blocks per group, 32768 fragments per group 8192 inodes per group
Superblock backups stored on blocks:   32768, 98304, 163840, 229376

Allocating group tables: done  Writing inode tables: done
Writing superblocks and filesystem accounting information: done

testadm $ sudo mount  <device_id> /mnt
testadm $ touch /mnt/test
testadm $ ls -l /mnt/

total 16 drwx------ 2 root root 16384 Nov 13 17:43 lost+found
-rw-r--r-- 1 root root 0 Nov 13 17:45 test

testadm $ sudo umount /mnt
testadm $ exit
```

Unlink the storage from that VM and start a new one with that storage attached:

```shell
$ occi -a unlink -r  <storage_link_id>
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a create -r compute -M  <resource_id> -M <occi_id> \
    -t occi.core.title="My OCCI VM" \
    -T user_data= <file://$PWD/ctx.txt> \
    -j <storage_id>** https://carach5.ics.muni.cz:11443/compute/46936
```

Check the IP of the new VM and login:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a describe -r <new_vm_id>**

[...]
occi.compute.state = active
[...]
>> location: /link/storagelink/compute_46936_disk_2
[...]
occi.storagelink.deviceid = /dev/xvdb
[...]
occi.networkinterface.address = 147.251.3.63
[...]

$ ssh -i tempkey -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
    testadm@ <new_vm_ip_addr>
testadm $ sudo mount <device_id> /mnt
testadm $ ls -ltr /mnt/

total 16
drwx------ 2 root root 16384 Nov 13 17:43 lost+found
-rw-r--r-- 1 root root 0 Nov 13 17:45 test

testadm $ sudo umount /mnt
testadm $ exit
```

Clean up by unlinking the storage, and removing VMs and storage:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a unlink -r <new_storage_link_id>
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a delete -r <vm_id>
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a delete -r <new_vm_id>
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X \
    -a delete -r <storage_id>
```

Finally check that the VMs and storage are not listed by occi:

```shell
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X -a list -r compute
$ occi -e <occi_endpoint> -n x509 -x $X509_USER_PROXY -X -a list -r storage
```

### OpenStack Compute checks (org.openstack.nova service type)

_NOTE:_ Prerequisite: run the following commands is the installation of the
OpenStack CLI with VOMS extensions, as shown in
[on the API and SDK guide](https://wiki.egi.eu/wiki/Federated_Cloud_APIs_and_SDKs#CLI_2).

Export the following variables on your shell (keystone URL can be obtained from
GOCDB URL of the endpoint)

```shell
$ export OS_AUTH_URL= <keystone URL>
$ export OS_AUTH_TYPE=v2voms
$ export OS_X509_USER_PROXY=$X509_USER_PROXY
```

Get the list of tenants supporting your proxy

```shell
$ keystone_tenants

Tenant id: 999f045cb1ff4684a15ebb338af69460
Tenant name: VO:fedcloud.egi.eu Enabled: True
Description: VO fedcloud.egi.eu
```

Export the tenant name as shown from the command in the following variable:

```shell
$ export OS_PROJECT_NAME="VO:fedcloud.egi.eu"
```

Describe the available flavors, check also that the template ID provided by
AppDB is listed:

```shell
$ openstack flavor list

+----+-----------+-------+------+-----------+-------+-----------+
| ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+-----------+-------+------+-----------+-------+-----------+
| 1  | m1.tiny   |   512 |    0 |         0 |     1 | True      |
| 2  | m1.small  |  2000 |   10 |        20 |     1 | True      |
| 3  | m1.medium |  4000 |   10 |        40 |     2 | True      |
| 4  | m1.large  |  7000 |   20 |        80 |     4 | True      |
| 5  | m1.xlarge | 14000 |   30 |       160 |     8 | True      |
+----+-----------+-------+------+-----------+-------+-----------+
```

Describe available images, again check that the ID provided by AppDb is listed

```shell
$ openstack image list

+--------------------------------------+----------------------------------------------+
| ID                                   | Name                                         |
+--------------------------------------+----------------------------------------------+
| 1414f242-d6d1-4a8c-8b26-8c0ada32f343 | IFCA Fedora Cloud 23                         |
| 8a700834-04b4-4e91-a3d5-9246ef95167e | LW Jupyter-R Ubuntu 14.04                    |
| 7f361fba-21d6-40ca-892d-17aa60b63a66 | IFCA CentOS 7                                |
| f3544cc8-421f-4d93-ac35-eba7fdc75329 | IFCA CentOS 6                                |
...
+--------------------------------------+----------------------------------------------+
```

Start a VM (using the flavor and images checked above and the context file
created previously). The returned ID will be used in the following commands:

```shell
$ openstack server create \
    --flavor <flavor> --image <image id> \
    --user-data ctx.txt test

+--------------------------------------+------------------------------------------------------+
| Field                                | Value                                                |
+--------------------------------------+------------------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                               |
| OS-EXT-AZ:availability_zone          | nova                                                 |
| OS-EXT-STS:power_state               | 0                                                    |
| OS-EXT-STS:task_state                | None                                                 |
| OS-EXT-STS:vm_state                  | building                                             |
| OS-SRV-USG:launched_at               | None                                                 |
| OS-SRV-USG:terminated_at             | None                                                 |
| accessIPv4                           |                                                      |
| accessIPv6                           |                                                      |
| addresses                            |                                                      |
| config_drive                         |                                                      |
| created                              | 2016-03-01T13:07:10Z                                 |
| flavor                               | m1.tiny (1)                                          |
| hostId                               |                                                      |
| id                                   | 5d3ed7d6-d5ac-4f09-a353-0c2bd0fbd0ea                 |
| image                                | IFCA CentOS 7 (7f361fba-21d6-40ca-892d-17aa60b63a66) |
| key_name                             | None                                                 |
| name                                 | test                                                 |
| os-extended-volumes:volumes_attached | []                                                   |
| progress                             | 0                                                    |
| project_id                           | 999f045cb1ff4684a15ebb338af69460                     |
| properties                           |                                                      |
| security_groups                      | [{u'name': u'default'}]                              |
| status                               | BUILD                                                |
| updated                              | 2016-03-01T13:07:11Z                                 |
| user_id                              | a31b8c452b594369a49a8329103e241a                     |
+--------------------------------------+------------------------------------------------------+
```

Check that the VM is active by describing it (it may take a few minutes):

```shell
$ openstack server show <vm id>

+---------------------+--------+
| Field               | Value  |
+---------------------+--------+
(...)
| OS-EXT-STS:vm_state | active |
(...)
+---------------------+--------+
```

If the VM does not have a public IP, you will need to get an IP for it

```shell
$ openstack ip floating pool list

+-------------+
| Name        |
+-------------+
| nova        |
+-------------+

$ openstack ip floating pool create <pool name>

+-------------+----------------+
| Field       | Value          |
+-------------+----------------+
| fixed_ip    | None           |
| id          | 1265           |
| instance_id | None           |
| ip          | 193.146.75.245 |
| pool        | nova           |
+-------------+----------------+

$ openstack ip floating add <ip> <vm id>
```

The VM should have now a public IP available when shown:

```shell
$ openstack server show <vm id>
+-----------+-------------------------------------+
| Field     | Value                               |
+-----------+-------------------------------------+
(...)
| addresses | private=172.16.8.14, 193.146.75.245 |
(...)
+-----------+-------------------------------------+
```

ssh to the machine to the provided IP (the options avoid problems when different
VMs have the same IP, don't use them in production) and check that
contextualization script was executed:

```shell
$ ssh -i tempkey -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
    testadm@ <ip> "cat /tmp/deployment.log"

Warning: Permanently added '193.146.75.245' (ECDSA) to the list of known hosts.

OK
```

Create a storage volume:

```shell
$ openstack volume create --size 1 test

+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | false                                |
| created_at          | 2016-03-01T13:29:44.392423           |
| display_description | None                                 |
| display_name        | test                                 |
| id                  | 8f46046d-cd9b-4219-9ce7-f0abe30ad992 |
| properties          |                                      |
| size                | 1                                    |
| snapshot_id         | None                                 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | None                                 |
+---------------------+--------------------------------------+
```

And show its status:

```shell
$ openstack volume show <vol id>
```

Attach it to the running VM:

```shell
$ openstack server add volume <vm id> <vol id>
```

And check it's attached:

```shell
$ openstack volume show 8f46046d-cd9b-4219-9ce7-f0abe30ad992

+---------------------+---------------------------------------------------------+
| Field               | Value                                                   |
+---------------------+---------------------------------------------------------+
| attachments         | [{u'device': u'/dev/vdb',                               |
|                     | u'server_id': u'21f6123f-926c-4816-b4fb-53df91907a63',  |
|                     | u'id': u'8f46046d-cd9b-4219-9ce7-f0abe30ad992',         |
|                     | u'volume_id': u'8f46046d-cd9b-4219-9ce7-f0abe30ad992'}] |
| availability_zone   | nova                                                    |
| bootable            | false                                                   |
| created_at          | 2016-03-01T13:29:44.000000                              |
| display_description | None                                                    |
| display_name        | test                                                    |
| id                  | 8f46046d-cd9b-4219-9ce7-f0abe30ad992                    |
| properties          |                                                         |
| size                | 1                                                       |
| snapshot_id         | None                                                    |
| source_volid        | None                                                    |
| status              | in-use                                                  |
| type                | None                                                    |
+---------------------+---------------------------------------------------------+
```

And login into the machine to create a filesystem, mount it, create a file, and
umount:

```shell
$ ssh -i tempkey -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
    testadm@ <ip_addr>

testadm $ sudo mke2fs <device_id>
(...)

testadm $ sudo mount <device_id> /mnt
testadm $ touch /mnt/test

testadm $ ls -l /mnt/

total 16 drwx------ 2 root root 16384 Nov 13 17:43 lost+found
-rw-r--r-- 1 root root 0 Nov 13 17:45 test

testadm $ sudo umount /mnt
testadm $ exit
```

Delete the VM:

```shell
$ openstack server delete <vm id>
```

And create a new one with the volume attached directly (substitute vdb for the
same device name shown above):

```shell
$ openstack server create \
    --flavor <flavor> --image <image id> \
    --block-device-mapping vdb= <volume id> \
    --user-data ctx.txt test

+-----------------------------+---------------------------------------------+
| Field                       | Value                                       |
+-----------------------------+---------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                      |
| OS-EXT-AZ:availability_zone | nova                                        |
| OS-EXT-STS:power_state      | 0                                           |
| OS-EXT-STS:task_state       | scheduling                                  |
| OS-EXT-STS:vm_state         | building                                    |
| accessIPv4                  |                                             |
| accessIPv6                  |                                             |
| addresses                   |                                             |
| adminPass                   | 9imSRC9EZhhX                                |
| config_drive                |                                             |
| created                     | 2016-03-01T13:45:21Z                        |
| flavor                      | m1.tiny (1)                                 |
| hostId                      |                                             |
| id                          | 10000d50-239b-4e86-bbd8-224143d6d346        |
| image                       | Image for EGI Centos 6 [CentOS/6/KVM]_egi   |
| key_name                    | None                                        |
| name                        | test                                        |
| progress                    | 0                                           |
| project_id                  | fffd98393bae4bf0acf66237c8f292ad            |
| properties                  |                                             |
| security_groups             | [{u'name': u'default'}]                     |
| status                      | BUILD                                       |
| updated                     | 2016-03-01T13:45:23Z                        |
| user_id                     | 6c254b295af64644904a813db0d3d88a            |
+-----------------------------+---------------------------------------------+
```

Assign the public IP (if it does not have one already):

```shell
$ openstack ip floating add <ip> <vm id>
```

And check that the volume is attached and usable:

```shell
$ ssh -i tempkey -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
    testadm@ <new_vm_ip_addr>

testadm $ sudo mount  <device_id> /mnt
testadm $ ls -ltr /mnt/

total 16
drwx------ 2 root root 16384 Nov 13 17:43 lost+found
-rw-r--r-- 1 root root 0 Nov 13 17:45 test

testadm $ sudo umount /mnt
testadm $ exit
```

```shell
Finally delete VM, volume and IP address (NOTE: use the id of the IP as returned
by openstack ip floating list\!):

$ openstack server delete <vm id>
$ openstack volume delete <volume id>
$ openstack ip floating delete <ip id>
```

### Cloud Storage (CDMI) checks (eu.egi.cloud.storage-management.cdmi service type)

_NOTE:_ Prerequisite: run the following commands is the installation of the
EGI CLI environment, according to
[this guide](https://wiki.egi.eu/wiki/Fedcloud-tf:CLI_Environment).

List the content of the repository:

```shell
$ bcdmi -e <cdmi_endpoint> list /
```

Create a test directory:

```shell
$ bcdmi -e <cdmi_endpoint> mkdir test
{
  "completionStatus": "Complete",
  "objectName": "test/",
  "capabilitiesURI": "/cdmi/AUTH_df37f5b1ebc94604964c2854b9c0551f/cdmi_capabilities/container/",
  "parentURI": "/cdmi/AUTH_df37f5b1ebc94604964c2854b9c0551f/",
  "objectType": "application/cdmi-container",
  "metadata": {}
}
```

Create a test file and upload it to the created directory:

```shell
$ echo "TEST OK" > testfile
$ bcdmi -e <cdmi_endpoint> put -T testfile
test/test.txt**
```

Try to download back the file and compare to previous:

```shell
$ bcdmi -e <cdmi_endpoint> get test/test.txt -o testfile.downloaded
$ diff testfile testfile.downloaded && echo "Files are equal" \
    || echo "Files differ"

Files are equal
```

Delete the file:

```shell
$ bcdmi -e <cdmi_endpoint> delete test/test.txt
```

Check that the file is not present anymore

```shell
$ bcdmi -e <cdmi_endpoint> list test/
```

Upload the file again and recursively delete the directory:

```shell
$ bcdmi -e <cdmi_endpoint> put -T testfile test/test.txt
$ bcdmi -e <cdmi_endpoint> delete -r test/
```

Check that the folder does not exist anymore

```shell
$ bcdmi -e <cdmi_endpoint> list /
```

See Site Certification GIIS Check [HOWTO03](../howto03_site_certificatoin_giis_check).

See to Resource Centre registration and certification procedure
[PROC09](https://confluence.egi.eu/display/EGIPP/PROC09+Resource+Centre+Registration+and+Certification).
