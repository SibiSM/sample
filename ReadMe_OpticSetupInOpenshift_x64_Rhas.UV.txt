*****************************************************************

Copyright (c) 2020 Burning Glass Technologies. All rights reserved.

LENS Optic Docker setup (Linux) in RedHat OpenShift (Kubernetes Cluster)

******************************************************************

PREREQUISITES
==============

Minimum hardware requirements and software prerequisites needed in IBM Cloud

A. IBM cloud account should be created with the administrative privilege for the below IBM services
	Infrastructure Service
	Container Registry service
	Kubernetes Service 
	
B. The installation drive for LENS Optic Docker images should have minimum of 25 GB of free hard disk space in IBM Cloud file storage.    
C. IP address should be assigned to cluster. By default, it is assigned while creating OpenShift cluster. 
D. The server should have minimum of 12 GB physical memory and minimum of 6 logical CPU cores for the 2 worker threads in Resume Tagger, 1 worker threads in Posting Tagger, 1 worker thread in SubOccProcessor, 1 RabbitMQ, 1 Logger & 1 ProtocolAdapter.
		Each Resume/Posting worker thread should have minimum of 1 CPU core and 2 GB RAM
		Each SubOccProcessor worker thread should have minimum of 1 CPU core and 200 MB RAM
		Other services (RabbitMQ, Logger & ProtocolAdapter) should have minimum of 1 CPU core and 2 GB RAM
E. Note: The following procedure has been shown as per the current LENS Optic Docker setup in the OpenShift Environment (development/testing environment). 
F. IBM Cloud CLIs, IBM Cloud Container Registry,  Kubernetes CLIs & Docker. The installation should be done by following the installation article -  https://cloud.ibm.com/docs/containers?topic=containers-cs_cli_install
G. Python 2.7 should be installed in the machine. If it has Python 3.x, then configure the virtual environment for Python 2.7.

==================================================================
Steps to install LENS Optic docker images in Kubernetes & OpenShift
==================================================================

	1. Assume the basic installation execution path is "/opt/data/opticus" and change the directory
	
		$ cd /opt/data/opticus
		
	2. Login into OpenShift cluster and provide username, password and region in interactive mode
	
		$ oc login
		
			Username:<< Enter email address >>
			Password:<< Enter password >>
		
	3. Connect to FTP "transfer.burning-glass.com" and download the compressed image files & other supporting files
	
		$ yum install ftp					# Install ftp if it is not there already
		$ ftp transfer.burning-glass.com	# Connect transfer.burning-glass.com ftp and issue credentials
			> username						# Username which is shared with client over email
			> password						# Password which is shared with client over email
			> binary						# Set binary mode
			> cd <path>						# Redirected to downloaded path which is shared with client over email
			> get <file>					# Download each of the compressed image file & other supporting files which you can find in #3.1
			> exit							# Exit once all the files are downloaded

		3.1. Following are the list of files present in this downloaded path

			bgtconfigtool_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
			bgtprotocoladapter_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
			bgtlogger_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
			bgttagger_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
			bgtsuboccprocessor_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
			ReadMe_OpticSetupInOpenshift_x64_Rhas.UV.txt
			lens_talker.tgz


	4. Uncompress the image files in the downloaded path

		$ tar -zxf bgtconfigtool_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
		$ tar -zxf bgtprotocoladapter_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
		$ tar -zxf bgtlogger_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
		$ tar -zxf bgttagger_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz
		$ tar -zxf bgtsuboccprocessor_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar.gz

	5. Login into IBM Container registry by following below article and push images into container registry
		https://cloud.ibm.com/kubernetes/registry/main/start?platformType=openshift 
		
		$ ibmcloud login -a https://cloud.ibm.com
			Email>				# ibmcloud account email for user
			Password>			# ibmcloud account password for user
			
			Select a region (or press enter to skip):		# Choose region if you know already
			1. au-syd
			2. in-che
			3. jp-tok
			4. kr-seo
			5. eu-de
			6. eu-gb
			7. us-south
			8. us-east
			Enter a number>
			
		$ ibmcloud target -g Default				# "Default" is the resource group specified. Change this if your resource group is different
		$ ibmcloud cr region-set us-south			# "us-south" is the region specified. Change this if your region is different
		$ ibmcloud cr namespace-add bgtopticengus
		$ ibmcloud cr namespace-add ibmopticengus
		$ ibmcloud cr login

		Note: Minimum of 8 GB storage space should be there in IBM container registry
		
		5.1. Start Docker if it is not already
		
			$ sudo systemctl start docker		# Remove "sudo" if user has administrative privileges
		
		5.2. Load images into local docker registry
	
			$ docker load --input bgtconfigtool_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar
			$ docker load --input bgtlogger_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar
			$ docker load --input bgttagger_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar
			$ docker load --input bgtprotocoladapter_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar
			$ docker load --input bgtsuboccprocessor_8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN.tar

		5.3. Choose a repository (Example: "us.icr.io/bgtopticengus" based on above namespace and tag the images by executing below commands
		
			$ docker tag 7cd52b034900 us.icr.io/bgtopticengus/bgtconfigtool:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN
			$ docker tag e1a7042e5010 us.icr.io/bgtopticengus/bgtlogger:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN
			$ docker tag dede82f97939 us.icr.io/bgtopticengus/bgttagger:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN
			$ docker tag 18208c1cafb8 us.icr.io/bgtopticengus/bgtsuboccprocessor:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN
			$ docker tag bdfdb3cbc2bc us.icr.io/bgtopticengus/bgtprotocoladapter:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN

		5.4. Push image into IBM container registry once there is enough storage space in registry from https://cloud.ibm.com/kubernetes/registry/main/settings?platformType=openshift
	
			$ docker push us.icr.io/bgtopticengus/bgtconfigtool:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN
			$ docker push us.icr.io/bgtopticengus/bgtlogger:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN
			$ docker push us.icr.io/bgtopticengus/bgttagger:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN
			$ docker push us.icr.io/bgtopticengus/bgtsuboccprocessor:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN
			$ docker push us.icr.io/bgtopticengus/bgtprotocoladapter:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN

		5.5. Run the below command to list the images in IBM container registry and confirm the availability of below images
		
			$ ibmcloud cr images

			Repository                                   Tag                                               Digest         Namespace       Created        Size     Security status
			us.icr.io/bgtopticengus/bgtconfigtool        8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN          99b5890c36d6   bgtopticengus   6 days ago     481 MB   No Issues
            us.icr.io/bgtopticengus/bgtlogger            8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN          7e6351651cb9   bgtopticengus   6 days ago     483 MB   No Issues
            us.icr.io/bgtopticengus/bgtprotocoladapter   8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN          cc750ba7b31a   bgtopticengus   6 days ago     483 MB   No Issues
            us.icr.io/bgtopticengus/bgtsuboccprocessor   8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN          3a6d38877eeb   bgtopticengus   5 days ago     455 MB   No Issues
            us.icr.io/bgtopticengus/bgttagger            8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN          2ceeb1f22575   bgtopticengus   6 days ago     2.6 GB   No Issues

			

	6. Extract the downloaded file bgt-ibm-openshift-opticus-1.0.1.tgz to a temporary folder /opt/data/opticus/deployment. 

		$ mkdir deployment
		$ cd deployment
		$ cp ../bgt-ibm-openshift-opticus-1.0.18.tgz .
		$ tar -zxf bgt-ibm-openshift-opticus-1.0.18.tgz
		$ cp ../lens_talker.tgz .
		$ tar -zxf lens_talker.tgz
		$ ls -R

		The folder should contain the following files.
			bgt-ibm-openshift-opticus-1.0.18.tgz  
			config_generate.sh
			optic-configtool-pod.yml
			optic-configtool-response.txt
			optic-pv-claim-ibmcloud.yml
			optic-us-deployment.yml
			lens_config_image_creator.py 			
			lens_liveness_check.py
			lens_service_pre_flight_check
			lens_talker.py
			Makefile
			rabbitmq_config_unmetered.json
			docker-src/Dockerfile.logger
			docker-src/Dockerfile.protocoladapter
			docker-src/Dockerfile.rabbitmq
			docker-src/Dockerfile.suboccprocessor
			docker-src/Dockerfile.tagger
			docker-src/Makefile
			docker-src/rabbitmqadmin

	7. Create Persistent Volume claim by executing "optic-pv-claim-ibmcloud.yml" and create pvc if it is not created already. To check the availability of PVC, execute the below command

                $ kubectl get pvc

                It shows the list of PVCs. Check the PVC "optic-uv-pv-claim" is present in the list which can be referred from "name: optic-uv-pv-claim" in optic-pv-claim-ibmcloud.yml

                7.1. If configured PVC is not there already, then execute the below to create new PVC

                        $ kubectl apply -f optic-pv-claim-ibmcloud.yml

                7.2. Check the PVC created by executing the below command and wait for couple of minutes for the STATUS to be changed from "Pending" to "Bound" before continuing further. It may take couple of minutes. If it is "Bound", then PV storage allocated by IBM Cloud.

                        $ kubectl get pvc

                        Run this repeatedly to know the STATUS

	8. Create LENS services configuration files and new version of images baked with the config files by executing the below script. The working directory for the following script should contain the above extracted files (#6)

		The input argument to the script will be the directory containing the make file and the Dockerfile.* for each service that we want to make the images with.

		$ python lens_config_image_creator.py docker-src

		After the execution of the above command, we have the updated images with config files baked inside.


	9. Push the new images into the IBM Container registry from the local registry

		$ docker push us.icr.io/ibmopticengus/bgtrabbitmq:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.1
		$ docker push us.icr.io/ibmopticengus/bgtlogger:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.1
		$ docker push us.icr.io/ibmopticengus/bgttagger:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.1
		$ docker push us.icr.io/ibmopticengus/bgtsuboccprocessor:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.1
		$ docker push us.icr.io/ibmopticengus/bgtprotocoladapter:8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.1
	 
	10. Deploy "optic-us-deployment.yml" for creating rabbitmq and LENS services containers. The externally visible port of the service is 32003 (default). Follow "Steps to change the externally visible port of the service" at end of this document if this port needs to be changed.
	
		$ kubectl apply -f optic-us-deployment.yml
		
		10.1. To check the status of pods
		
			$ kubectl get pods
			
		10.2. To get more details about running containers
		
			$ kubectl describe pod <POD NAME>

		10.3. To get the deployment details
		
			$ kubectl get deployment
			
		10.4. To get the Service details
		
			$ kubectl get svc

	11. Now you can send info/tag command to the sample lens_talker.py client by providing the OpenShit Cluster IP address, port 32003, timeout and see the info or tag response
			
			$ python lens_talker.py 
				
			  Usage: lens_talker.py lens_command<info/tag> host_name lens_port time_out_seconds
	
			$ python lens_talker.py info 169.46.85.83 32007 10
				<?xml version="1.0" encoding="utf-8"?>
				<bgtres>
				<info>
				<service name="docker-instance" status="active">
				<language>eng</language>
				<country>USA</country>
				<version>8.0.2.2.126 Optic v4.1.0.0</version>
				</service>
				</info>
				</bgtres>
				
			$ python lens_talker.py tag 169.46.85.83 32003 10
				<?xml version="1.0" encoding="utf-8"?>
				<bgtres>
				<ResDoc>
				<resume xml:space="preserve" canonversion="2" dateversion="2" present="737597" iso8601="2020-06-19"><contact><name><givenname>John</givenname> <surname>Smith</surname></name></contact> </resume>

				<skillrollup version="2.0"/>
				<DataElementsRollup version="8.0.2.2.126 Optic v4.1.0.0">
				<Certification/>
				<CanonCertification/>
				</DataElementsRollup>
				</ResDoc>
				</bgtres>
				
			Note: If IP or port is incorrect or service is not running, then you get "Couldn't connect to LENS server" error.
			
				
Steps to delete pod, deployment, svc, pvc, images in local docker & IBM container registry
==========================================================================================

	1. Delete pod after getting the list of pods
	
		$ kubectl get pods
		
		$ kubectl delete pod <POD NAME>
		
	2. Delete deployment after getting the list of deployments
	
		$ kubectl get deployment
		
		$ kubectl delete deployment <DEPLOYMENT NAME>
		
	3. Delete service after getting the list of services 
	
		$ kubectl get svc
		
		$ kubectl delete svc <SERVICE NAME>		
		
	4. Delete pvc after getting the list of pvc. If we delete this, then persistent-data will be lost
	
		$ kubectl get pvc
		
		$ kubectl delete pvc <PVC NAME>
		
		
Steps to change the externally visible port of the service
==========================================================

	1. Change the nodePort number (nodePort: 32003 (specified by default) -> 32002) by executing the below sed command

		$ sed -i 's/nodePort: 32003/nodePort: 32002/g' optic-us-deployment.yml

	2. Follow the cleanup steps #2 & #3 in "Steps to delete pod, deployment, svc, pvc, images in local docker & IBM container registry" if the deployment is done already.
	
	3. Follow the installation steps "Steps to install LENS Optic docker images in Kubernetes & OpenShift" starting from #10.


Steps to change the number of worker threads count in Resume Tagger/Posting Tagger/Salary Processor
===================================================================================================

	1. Change the below variable in config_generate.sh based on the hardware availability. Please check prerequisites for minimum hardware needed
	
		RESUME_WORKER_COUNT=2
		POSTING_WORKER_COUNT=1
		SALARY_PROCESSOR_WORKER_COUNT=1
		SUBOCC_PROCESSOR_WORKER_COUNT=1
		
	2. Follow the cleanup steps "Steps to delete pod, deployment, svc, pvc, images in local docker & IBM container registry" if the deployment is done already.
	
	3. Increment the image version which baked with config file by executing the below cat command to find the current image version. Then execute sed command to increment version (Example below shows to change "8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.1" to "8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.2") 
	
		$ cat docker-src/Makefile

			IMAGE_NAME_BASE = us.icr.io/ibmopticengus
			VERSION_NAME = 8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.1

		$ sed -i 's/VERSION_NAME = 8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.1/VERSION_NAME = 8.0.2.2.126-Optic-4.1.0.0.UV_eng-USA_CAN_v1.0.2/' docker-src/Makefile
		
	4. Follow the installation steps "Steps to install LENS Optic docker images in Kubernetes & OpenShift" starting from #8 after this update


Steps to change the heartbeat check intervals (livenessProbe) 
=============================================================

	1. Change the initialDelaySeconds, periodSeconds, timeoutSeconds in optic-us-deployment.yml for RabbitMQ container if needed. Following is the default livenessProbe timing 
		
		livenessProbe:
            exec:
              command: ["rabbitmqctl", "status"]
            initialDelaySeconds: 60
            # See https://www.rabbitmq.com/monitoring.html for monitoring frequency recommendations.
            periodSeconds: 60
            timeoutSeconds: 15

	2. Change the initialDelaySeconds, periodSeconds, timeoutSeconds and lens liveness check interval (last argument in command) in optic-us-deployment.yml for RabbitMQ container if needed. Following is the default livenessProbe timing 
	
	    livenessProbe:
            exec:
              command: ["/opt/bgtInstance/lens_liveness_check.py", "localhost", "2003", "20"]
            initialDelaySeconds: 150
            periodSeconds: 120
            timeoutSeconds: 30

	3. Follow the cleanup steps #2 & #3 in "Steps to delete pod, deployment, svc, pvc, images in local docker & IBM container registry" if the deployment is done already.
	
	4. Follow the installation steps "Steps to install LENS Optic docker images in Kubernetes & OpenShift" starting from #10 after this update


Please contact your Burning Glass representative and write to support@burning-glass.com for further clarifications.