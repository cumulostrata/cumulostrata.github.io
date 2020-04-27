<img src="/README-static-assets/cumulostrata_logo.png" width="450">

# Cumulostrata - GCP to Splunk logging automation Terraform generator

This project is designed to simplify the configuration of pushing data to Splunk using the GCP Dataflow [Pub/Sub to Splunk](https://cloud.google.com/dataflow/docs/guides/templates/provided-streaming#pubsub-to-splunk) template. The fully local configuration website (no external network requests) allows you to provide GCP and Splunk details, then select common data sources and provided custom Stackdriver/GCP Logging query filters to stream to Splunk. A [Terraform](https://www.terraform.io/) template is generated based on your selections, which can be run within GCP's Cloud Shell CLI, or on a local CLI tool to deploy logging automation.

Try out the configuration site here: https://cumulostrata.github.io, or download this repository and open the `index.html` file.

Wondering why the name Cumulostrata? It's a mashup of *[Cumulonimbus clouds](https://en.wikipedia.org/wiki/Cumulonimbus_cloud)* and Terraform [*stratum*](https://en.wikipedia.org/wiki/Stratum).

## Prerequisites and GCP side considerations.
1. **Ensure the user deploying the template has the appropriate roles and permissions to deploy the Terraform template:**
	-  A full list of roles and permissions is being developed, however the user running the template must be able to create resources for the following GCP services: *Pub/Sub, Logging, IAM, Dataflow, GCS*
2. **Enabling audit logging (Data Access):**
	-	Most audit logs in GCP are enabled by default, however Data Access logs must be turned on in each project being monitored. See more documentation about enabling Data Access audit logs [here](https://cloud.google.com/logging/docs/audit/configure-data-access).
	- The query filter automatically used to capture Cloud Audit logs is below (if selected in the configuration site):
		- `log_name:"projects/[PROJECT_ID]/logs/cloudaudit.googleapis.com"`
	
3. **Enabling VPC flow logs:**
	-	VPC flow logs must be turned on in each subnet, each configured subnet has an aggregation interval, sample rate, among other configurations. See more documentation about enabling VPC flow logs [here](https://cloud.google.com/vpc/docs/using-flow-logs#enabling_vpc_flow_logging).
	-	The query filter automatically used to capture VPC flow logs is below (if selected in the configuration site):
		-	`resource.type="gce_subnetwork"  AND  
log_name="projects/[PROJECT_ID]/logs/compute.googleapis.com%2Fvpc_flows"`

4. **Determine other data sources relevant to your requirements and use cases:**
	- The template generation tool provides an option for custom query filters, any events matching these filters will be sent to Splunk once the logging infrastructure is fully deployed.
	- See the [Query filter library](https://cloud.google.com/logging/docs/view/query-library#networking-filters) for ideas of possible query filters
	- Note that each query filter will be `OR`ed together. i.e. `[Custom filter 1] OR [Custom fitler 2]`

## Deploying the generated Terraform template
1. The Cloud Shell in the GCP console is the simplest way to deploy the generated Terraform template. The Cloud Shell has Terraform already installed, so the only steps required are to upload the file using the upload file utility (See the 3 dot `More`) menu and select `Upload File`
	- The same steps should work outside of Cloud Shell, assuming Terraform in installed and GCP credentials have been set using an environment variable. See more documentation [here](https://www.terraform.io/docs/providers/google/guides/getting_started.html).
2. `mkdir splunk_terraform`
3. `mv customized_splunk_gcp_template splunk_terraform`
4. `cd splunk_terraform`
5. `terraform init`
6. `terraform apply`
	- Type `yes` when prompted to confirm
	- **If this fails (common), run `terraform apply` again once**
7. To roll back the deployment `terraform destroy`
	-	Type `yes` when prompted to confirm