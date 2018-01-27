
Description of the file structure of the repository :

output  : testing results of our application
runningSpark : files required to setup the spark cluster
src/main/java/com/hortonworks/example : code for the monte carlo simulator application
stockData :  input data for the simulator collected from the finanice API
target :  build folder for the application
README.md : read me file for the submission
companies_list.txt : the list of companies used
downloadData.py : the script to extract data from the finance API
downloadHistoricalData.sh : the script to extract data from the finance API
pom.xml :  build file for the application

Instructions for a quick test of our application:

Connect to the pis using ssh.
Listing the commands for every pi below:


1.	Perform the following on Master node.
	sudo su

	kubeadm reset

	kubeadm init --pod-network-cidr 10.244.0.0/16

	su pirate

	sudo cp /etc/kubernetes/admin.conf $HOME/

	sudo chown $(id -u):$(id -g) $HOME/admin.conf

	export KUBECONFIG=$HOME/admin.conf

	curl -sSL https://rawgit.com/coreos/flannel/v0.7.1/Documentation/kube-flannel-rbac.yml | kubectl create -f -

	curl -sSL https://rawgit.com/coreos/flannel/v0.7.1/Documentation/kube-flannel.yml | sed "s/amd64/arm/g" | kubectl create -f -

	sudo iptables -P FORWARD ACCEPT

2.	Perform the following on the slave nodes.
	sudo su

	kubeadm reset

	<kubeadm_join_command_issued_by_master_node> // once it is initialized it will give such a command for slaves to join.

	sudo iptables -P FORWARD ACCEPT

3.	On the master node, do the following to set up the Spark cluster:
	cd spark

	kubectl create -f spark-master.yaml

	kubectl get pods

	//identify the 'spark-master' pod.

	//enter the shell of the master using:

	kubectl exec -it <name_of_master_pod> bash

	tail -f spark/logs/_(whatever is the name of the only file here)

	//wait till the screen shows "I HAVE BEEN ELECTED: I AM ALIVE!"

	exit // exits master shell.

	kubectl create -f spark-master-service.yaml

	kubectl create -f spark-worker.yaml

	kubectl get pods

	//identify the 'spark-master' pod.

	//enter the shell of the master using:

	kubectl exec -it <name_of_master_pod> bash

	tail -f spark/logs/_(whatever is the name of the only file here)

	//wait till the screen shows "Registered worker..."

	spark-submit --properties-file s3.properties --class com.hortonworks.example.Main --master spark://spark-master:7077 --num-executors 4 --driver-memory 1024m --executor-memory 1024m --executor-cores 4 --queue default --deploy-mode cluster --conf spark.eventLog.enabled=true --conf spark.eventLog.dir=file:///eventLogging mc.jar s3a://spark-cloud/input/companies_list.txt s3a://spark-cloud/input/*.csv s3a://spark-cloud/output

	//wait till the screen shows "Registering app monte-carlo-var-calculator" and "Launching executor app.. on worker.."

	exit //exit master's shell

	kubectl get pods

	//pick the first worker in the list, copy it's name

	kubectl exec -it <spark_worker_name> bash

	cd spark/work

	ls

	//will show a "driver" file directory

	cd "driver.." whatever the name is

	tail -f stdout

	//you will see the output of our program here.

	//to check the output folder specified while running the program please run the following command to
	// download the output directory from s3. The aws cli interface is configured with our credentials. They
	// can be replaced.

	aws s3 cp s3://path-of-s3-repo/ output --recursive

	//This will download the output directory to a folder named 'output' in the directory from which this command is executed.
