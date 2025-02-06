### NOTE: MY POSTINGS REFLECT MY OWN VIEWS AND DO NOT NECESSARILY REPRESENT THE VIEWS OF MY EMPLOYER

# Azure Event Driven Discovery with Reader Privileges.  

## Overview 

Cloud Discovery is an incredibly powerful tool for providing visibility into your technology portfolio regardless of where it sits.  However, traditional schedule based discovery may not trigger quickly enough to capture changes within the CMDB for today's modern operational patterns.  As a result, ServiceNow has developed Event Driven Discovery for cloud platforms to enable near real time updating of your CMDB.   

In order to enable this in an easier fashion we've provided the "Azure Alert Configuration" tool which configures both the Azure requirements and the ServiceNow requirements.  However, that method requires the service principle to have contributor rights to the subscriptions you'd like to monitor.  This is problematic for a lot of organizations as it goes against their defined security policies.  

To that end I've worked alongside one of our great healthcare customers to come up with this process that will enable Azure users to deploy standard discovery using only the "Reader" role and then manually deploy the alert logs to configure Event Driven Discovery.  This allows the near real time updating of the CMDB while still respecting the organizational security policies around service principle rights.  

There are two files that I've added to my [Git Repository](https://github.com/MTGServiceNow/ServiceNowAzureTools) specifically for this purpose.  You'll need to download both files and customize EvDrDiscoParameters.json to match the parameters for your environment.  Ensure that you configure all the variables within the <> (and remove the <>) for your specific environment.  

## Steps  

1.  Configure schedule based discovery based on the documentation [here](https://docs.servicenow.com/bundle/quebec-it-operations-management/page/product/discovery/concept/azure-cloud-discovery.html).  Ensure that the service principal has reader permissions. 
2.  Create a new user for event driven discovery.  Ensure you know the username and password.  Ensure that it has the sn_cmp.cloud_event_integration role.  
3.  Edit the Scripted REST API configuration for "Cloud Event" to ensure the event post doesn't require authentication.  
4.  Deploy the ARM template in Azure.  
	* Use either the custom template deployment in the portal or the az cli command.  
		* Portal: 
			* Navigate to the resource group where you'd like to deploy the Log Alerts.  
				* Note that this resource group just houses the LogAlerts.  The scope of the alerts will be set to the subscription that contains this resource group.  
			* In the overview screen click "+ Add" at the top.  
			* Type "custom template" into the "Search the Marketplace" box.  
			* Click on "Template deployment (deploy using custom templates)"
			* Click the "Create" button	
			* Click "Build your own template in the editor"
			* Click "Load file" and navigate to the template provided in my git repository.  
			* Click "Save"
			* In the newly updated screen with parameters fields click "Edit parameters"
			* Click "Load file" and navigate to the location of the parameters file provided in my git repository
			* Click "Save"
			* Review and create the deployment.  
		* CLI
			* `az login`
			* `az deployment group create --name <name for the deployment> --resource-group <RG to deploy in> --template-file <template file> --parameters <parameters file> --subscription <subscription name>`
5. Go back to your Servicenow instance to verify that cloud events are posting appropriately
	* In the filter navigator type in "sn_cmp_cloud_event.LIST". This will open a new tab with the cloud event table.  This table is where the events are written by the POST api calls.  Once the events are entered here a background process reads them and updates the CMDB accordingly.  

Once you've completed this you'll need to repeat steps 4 and 5 until you've configured this for each subscription you want to discover.  This is a limitation of the LogAlert structure within Azure.  The largest scope that can be provided to a LogAlert is a subscription.  You cannot deploy these at a management group level.  

