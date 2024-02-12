# Prometheus and Grafana Lab

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Python 3.6](https://img.shields.io/badge/Python-3.6-green.svg)](https://shields.io/)

This repository contains the practice code for the **Prometheus and Grafana** lab in the Coursera.org course [**IBM-CD0267EN-SkillsNetwork Application Security and Monitoring**](https://www.coursera.org/learn/application-security-and-monitoring)

## Contents

This lab contains a Kubernetes manifests to deploy Prometheus and Grafana into the OpenShift lab environment.

It also contains Python Flask backend that has been instrumented with a `/metrics` endpoint to be consumed by Prometheus.

## Authors

[John J. Rofrano](https://www.coursera.org/instructor/johnrofrano) Senior Technical Staff Member, DevOps Champion, @ IBM Research  

## License

Copyright (c) IBM Corporation. All rights reserved.

Licensed under the Apache License. See [LICENSE](LICENSE)

This repo is part of the Coursera.org course [IBM-CD0267EN-SkillsNetwork Application Security and Monitoring](https://www.coursera.org/learn/application-security-and-monitoring/)

---

## <h3 align="center"> © IBM Corporation 2022. All rights reserved. <h3/>



Monitoring with Grafana
cognitiveclassui
Estimated time needed: 30 minutes

Welcome to the Monitoring with Grafana lab. In this lab you will learn to use Grafana as a visualization tool and dashboard for Prometheus.

Learning Objectives
In this hands-on lab, you will:

Deploy Prometheus to OpenShift
Deploy Grafana to OpenShift
Connect Prometheus as a datasource for Grafana
Create a dashboard with Grafana
Prerequisites
There is some setup to do before you can proceed with the lab. In this prerequisite step you will clone the GitHub repository that contains the Kubernetes manifests needed to deploy Prometheus and Grafana to an OpenShift clutser in the lab environment.

Your Task
First, if you do not already have a terminal open, open a terminal using the top menu item Terminal -> New Terminal and cd (change directory) into the default /home/project folder:

1
cd /home/project
Copied!Executed!
Then use the git clone command to clone this repository:

1
git clone https://github.com/ibm-developer-skills-network/ondaw-prometheus-grafana-lab.git
Copied!Executed!
Finally, change into the ondaw-prometheus-grafana-lab folder:

1
cd ondaw-prometheus-grafana-lab
Copied!Executed!
(Optional) Make your command prompt shorter with this bash command:

1
export PS1="\[\033[01;32m\]\u\[\033[00m\]:\[\033[01;34m\]\W\[\033[00m\]\$ "
Copied!Executed!
You are now ready to start the lab.

Step 1: Deploy node exporters
In this step, you will deploy 3 Node Exporters, which will be used as monitoring targets.

Your Task
Use the oc create deployment command to deploy 3 node exporters, named node-exporter1, node-exporter2, and node-exporter3, all listening on port 9100.

1
2
3
oc create deployment node-exporter1 --port=9100 --image=bitnami/node-exporter:latest
oc create deployment node-exporter2 --port=9100 --image=bitnami/node-exporter:latest
oc create deployment node-exporter3 --port=9100 --image=bitnami/node-exporter:latest
Copied!Executed!
Next, use the oc expose command to create services that expose the 3 node exporters so that Prometheus can communicate with them:

1
2
3
oc expose deploy node-exporter1 --port=9100 --type=ClusterIP
oc expose deploy node-exporter2 --port=9100 --type=ClusterIP
oc expose deploy node-exporter3 --port=9100 --type=ClusterIP
Copied!Executed!
Check that the pods are up and running with the following oc command:

1
oc get pods
Copied!Executed!
Results
You should see that all of the pods for the three node-exporters are in a running state like the image below:

grafana_node_exporter_running

You are now ready to deploy Prometheus.

Step 2: Deploy Prometheus
In this step, you will confiugure and deploy Prometheus. While normally you would modify configuration files to configure Prometheus, this is not the case for Kubernetes. For a Kubernetes environment the proper appoach is to use a ConfigMap. This makes it easy to change the configuraton later. You will find the configuration files from which to make the ConfigMap in a folder named ./config.

You will also need Kubernetes manifests to describe the Prometheus deployment and to link the ConfigMap with the Prometheus. These manifests can be found in the ./deploy folder.

Your Task
Use the following commands to create a ConfigMap and deploy Prometheus:

First, create a ConfigMap called prometheus-config that is needed by Prometheus from the prometheus.yml and the alerts.yml file in the ./deploy folder.

1
2
3
oc create configmap prometheus-config \
   --from-file=prometheus=./config/prometheus.yml \
   --from-file=prometheus-alerts=./config/alerts.yml
Copied!Executed!
You should see a message that the prometheus-config was created.

Now deploy Prometheus using the deploy/prometheus-deployment.yaml deployment manifest.

1
oc apply -f deploy/prometheus-deployment.yaml
Copied!Executed!
You should see a message that the prometheus-volume-claim, deployment.apps/prometheus, and service/prometheus were all created.

Finally, use the oc command to check that the prometheus pod is running.

1
oc get pods -l app=prometheus
Copied!Executed!
Results
You should see that the prometheus pod is in a running state.

grafana_prometheus_running

You are now ready to deploy Grafana.

Step 3: Deploy Grafana
Now that you have 3 node exporters to emit metrics, and Prometheus to collect them, it is time to add Grafana for dashboarding. You will deploy Grafana into OpenShift and create a route which will allow you to open up the Grafana web UI and work with it.

Your Task
First, deploy Grafana using the deploy/grafana-deployment.yaml deployment manifest.

1
oc apply -f deploy/grafana-deployment.yaml
Copied!Executed!
You should see a message that both the grafana deployment and service have been created.

Next, use the oc expose command to expose the grafana service with an OpenShift route. Routes are a special feature of OpenShift that makes it easier to use for developers.

1
oc expose svc grafana
Copied!Executed!
Your should see the message: “route.route.openshift.io/prometheus exposed”

Use the following oc patch command to enable TLS and the https:// protocol for the route.

1
oc patch route grafana -p '{"spec":{"tls":{"termination":"edge","insecureEdgeTerminationPolicy":"Redirect"}}}'
Copied!Executed!
Then use the oc get routes command to check the URL that the route was assigned. You will be able to interact with Grafana using this URL.

1
oc get routes
Copied!Executed!
Finally, use the oc command to check that the grafana pod is running.

1
oc get pods -l app=grafana
Copied!Executed!
Results
You should see that the grafana pod is in a running state.

grafana_pod_running

You should also see the URL of the route that you created.

grafana_route

You are now ready to configure Grafana.

Step 4: Log in to Grafana
Now that Prometheus and Grafana are deployed and running, it is time to configure Grafana. In order to do this, you will need the URL from the route that you created in the last step.

Note: You do not need a Grafana account in order to complete this step.

Your Task
Use the oc describe command along with grep to extract the URL of the Requested Host for the grafana route:

1
oc describe route grafana | grep "Requested Host:"
Copied!Executed!
You will see the words “Requested Host:” in red followed by a URL. It will start with the word “grafana-“ followed by the rest of the URL.

Copy the URL after the words “Requested Host:” to the clipboard and paste it into a new web browser window outside of the lab environment.

You should see the Grafana log in page:

grafana_login

From the Grafana login screen, log in with the default userid admin and default password admin, and then click the [Log in] button. You will be prompted to change your password. You can press the skip link to bypass that for now.

Results
You should see the Grafana home page:

Image of Grafana home page

Step 5: Configure Grafana
Once you have logged in to Grafana, you can configure Prometheus as your datasource.

Your Task
At the home page, select the data sources icon.

Image of Grafana data sources icon

At the Add data source page select Prometheus.



On the Prometheus configuration page, set the Prometheus URL to http://prometheus:9090 which is the name and port of the Prometheus service in OpenShift.

grafana_prometheus_url

Scroll down to the bottom of the screen and click the Save & test button.

grafana_save_test

Results
You should see a message, confirming that the data source is working:



Step 6: Create a Dashboard
Now you can create your first dashboard. For this lab, you will use a precreated template provided by Grafana Dashboard. This template is identified by the id 1860.

Your Task
On the Grafana homepage, click the Dashboards icon, and select + Import from the menu to start creating the dashboard.



Next,enter the template identifier id 1860 in the space provided and click the [Load] button to import that dashboard from Grafana.



You will see the default name for the template, Node Exporter Full, displayed. You are allowed to change it if you want to. The change will be local and valid only for your instance.



At the bottom of the page, choose Prometheus (Default) as the data source and click Import.



This completes importing a dashboard. Next you will see what the dashboard allows you to monitor.

Step 7: View the Dashboard
You should now see the General / Node Exporter Full dashboard which shows CPU Busy, Sys Load, RAM used and other information. This dashboard will allow you to get a broad overview of how the system is performing, as well as drilling down into individual nodes to see how they are performing. Now, try a few things out.

Things to try
You can hover over the graph to get specific information at a given time.



Choose a different host to observe the details about that host.



Choose to visualize the metrics over different time ranges.



As you can imagine, this could be monitoring an application that you have deployed into Kubernetes or OpenShift so that you can see the CPU, Memory, and other metrics to determine the overall health of your application.

