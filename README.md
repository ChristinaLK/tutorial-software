[title]: - "Software access using HTCondor or the Web"


Overview
--------
This module will illustrate how software can be transferred and accessed in OSG Connect. Different methods to access software without transferring it are described in [Distributed software access using the OASIS service](http://support.opensciencegrid.org/support/solutions/articles/5000634394).

In the section OSG Connect Data, you have seen the same mechanisms used to transfer data. The techniques and the recommendation are the same, e.g. use HTCondor 's built-in transfer mechanisms only for relatively small amounts of data and binaries to transfer (<100MB) or if you need to do ad-hoc job submissions.

Transferring software is very similar. When using HTCondor to transfer software keep in mind that the *Executable* specified in the submit file is transferred by default. Then remember that you must have execute permission on the software files, and that the PATH (folders where executables can be found) may be set differently on the remote host (pay attention on where the software is)

Starting Up
-----------
First  login to your osgconnect login node. Then, to create a working directory, you can either run 'tutorial software' or type the following:
	$ ssh <your_osg_connect_username>@<your_osg_login_node>
	$ mkdir -p tutorial-software
	$ cd tutorial-software
Then make sure that you have a public directory exported via Web. This directory should have been created for you and be accessible at an URL like https://stash.osgconnect.net/+username/ where "username" is your user name on OSG Connect :
	$ ls -al /public/<username>
Distributing Applications Using Stash and the OSG Connect Web Server
--------------------------------------------------------------------
This example is very similar to the one in Access Stash remotely from your job using HTTP. You will be preparing the input files for the job, including an executable, and transferring them via HTTP. When you are transferring an executable instead of data you have to pay attention because Web servers or commands like wget frequently change the file permissions and this can cause the job to fail. To make sure that your program is executable either manually set the permission on the transferred file (chmod +x filename) or transfer a tar bundle and uncompress it.

Prepare the file bundle in your public html space (/public/<username>):
	[user@login01 tutorial-software]$ ls -al /public/<username>
	...
	[user@login01 tutorial-software$ tar cvzf words.tar.gz distribution random_words
	distribution
	random_words
	[user@login01 tutorial-software]$ mv words.tar.gz /public/<username>/
	[user@login01 tutorial-software]$ chmod +r /public/<username>/words.tar.gz
words.sh is the auxiliary script to run the job:
	#!/bin/bash
	# HTCondor will transfer the file for us. Uncomment the following if you prefer to transfer form the script
	# wget --no-check-certificate http://stash.osgconnect.net/+username/words.tar.gz
	tar xzf words.tar.gz
	cat random_words | ./distribution
Here is the submit file *transfer.submit*. Note that HTCondor allows you to specify an URL as input file and it will download it for you. **Substitute your user name for username in the transfer_input_files line:**
	########################
	# Submit description file for short test program using http transfer
	########################
	Universe       = vanilla
	Executable     = app_script.sh
	# Uncomment the following line to use the remote eecutable
	#transfer_executable = False
	Error   = log/words.err.$(Cluster)-$(Process)
	Output  = log/words.out.$(Cluster)-$(Process)
	Log     = log/words.log.$(Cluster)
	should_transfer_files = YES
	transfer_input_files = http://stash.osgconnect.net/+username/words.tar.gz
	when_to_transfer_output = ON_EXIT
	Queue 5
Submit the job:
	[user@login01 tutorial-software]$ condor_submit words.submit
	Submitting job(s).....
	5 job(s) submitted to cluster 14038.
Once the jobs are completed, you can look at the output in the logs directory and verify that the job ran correctly:
	[user@login01 tutorial-software]$ cat log/words.out.14038-2
	Ashkenazim |45 (0.44%) +++++++++++++++++++++++++++++++++++++++++++++++++++++++++
	BIOS       |45 (0.44%) +++++++++++++++++++++++++++++++++++++++++++++++++++++++++
	Anaheim    |44 (0.43%) +++++++++++++++++++++++++++++++++++++++++++++++++++++++
	Aymara     |44 (0.43%) +++++++++++++++++++++++++++++++++++++++++++++++++++++++
	Arthurian  |43 (0.42%) ++++++++++++++++++++++++++++++++++++++++++++++++++++++
	Anaxagoras |43 (0.42%) ++++++++++++++++++++++++++++++++++++++++++++++++++++++
	Bactria    |43 (0.42%) ++++++++++++++++++++++++++++++++++++++++++++++++++++++
	Alexis     |43 (0.42%) ++++++++++++++++++++++++++++++++++++++++++++++++++++++
	Ariel      |43 (0.42%) ++++++++++++++++++++++++++++++++++++++++++++++++++++++
	Aubrey     |42 (0.41%) +++++++++++++++++++++++++++++++++++++++++++++++++++++
	Baryshnikov|42 (0.41%) +++++++++++++++++++++++++++++++++++++++++++++++++++++
	Bahia      |42 (0.41%) +++++++++++++++++++++++++++++++++++++++++++++++++++++
	Angstrom   |42 (0.41%) +++++++++++++++++++++++++++++++++++++++++++++++++++++
	Asoka      |42 (0.41%) +++++++++++++++++++++++++++++++++++++++++++++++++++++
	Alcatraz   |41 (0.40%) ++++++++++++++++++++++++++++++++++++++++++++++++++++

## Getting Help
For assistance or questions, please email the OSG User Support team  at `support@opensciencegrid.org` or visit the [help desk and community forums](http://support.opensciencegrid.org).
