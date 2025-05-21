# Retail Recommendation System Demo
## Introduction
This deployment is based on the validated pattern framework, utilizing GitOps for seamless provisioning of all operators and applications. It deploys a Retail Recommendation System that leverages two-towe algorithm training technich to provide personalized item suggestions to customers, enhancing store sales by considering their preferences and demographics.

The pattern harnesses Red Hat OpenShift AI to deploy and serve recommendation at scale. It integrates the Feast Feature Store for feature management, EDB Postgres to store user and item embeddings, and a simple user interface (UI) to facilitate customer interactions with the system. Running on Red Hat OpenShift, this demo showcases a scalable, enterprise-ready solution for retail recommendations.
## Pre-requisites

- Podman
- Red Hat Openshift cluster running in AWS. Supported regions are : us-east-1 us-east-2 us-west-1 us-west-2 ca-central-1 sa-east-1 eu-west-1 eu-west-2 eu-west-3 eu-central-1 eu-north-1 ap-northeast-1 ap-northeast-2 ap-northeast-3 ap-southeast-1 ap-southeast-2 ap-south-1.
- GPU Node to run Hugging Face Text Generation Inference server on Red Hat OpenShift cluster.
- Create a fork of the <TODO add link for the repo after moving it to VP project> git repository.

## Demo Description & Architecture
### Key Features
* UI: Allows users to browse recommendations, add items to cart, purchase, or rate products.
* Feast Feature Store: Manages and serves features for training and real-time inference.
* EDB Postgres with PGVector: Stores user and item embeddings, enabling fast similarity searches.
* Kafka Integration: Records user interactions for continuous learning and dataset updates.
* Red Hat OpenShift AI
* Two-Tower Architecture: Utilizes separate neural networks to generate user and item embeddings for personalized recommendations.
<!-- * Monitoring Dashboard: Provides performance metrics using Prometheus and Grafana.
* GitOps Deployment: Ensures an end-to-end, reproducible setup of the demo. -->

### Workflow

The workflow consists of the following steps:

1. Data Ingestion
* Data originates from parquet files containing users, items, and interactions.
* Feast scans feature definitions, validates them, and syncs metadata to its registry.

2. Training - using the Two-Tower algorithm:

![InGestion and Training](images/Data_ingestion_and_training.drawio.png)

We have two encoders:
* User Tower: Encodes user features (age, gender, preference, ...) into embedding.
* Item Tower: Encodes item features (category, price, ...) into embedding.
  
For each interaction, positive or negative, train the encoders in such a way that positive interactions bring the item and the user closer in cosine similarity in the embedding space, and negative interactions move the user and the item embeddings farther apart.

3. Batch Scoring & Materialization

![Batch Scoring](images/high_level_batch_scoring.drawio.png)

* Generates embeddings for all users and items using trained encoders.
* Computes the latest feature values and precomputes top-k recommendations for each user.
* Stores results in the online store for fast retrieval.

### Components Deployed
* Recommendation UI: A simple web application for users to interact with recommendations.
* Kafka & Kafka connect: ingest user interaction with items from the ui and sent them to Kafka, kafka connect move this intercation events into a EDB database.
* Feast Feature Store: Manages feature definitions and serves data for training and inference.
* EDB Postgres with PGVector: Acts as online (real-time embeddings) stores.
* Kubeflow job: A batch job that train the user and item encoders, then genrate the data generates embeddings into the vector database.

## Deploying the demo

To run the demo, ensure the Podman is running on your machine.Fork the [rag-llm-gitops](https://github.com/validatedpatterns/rag-llm-gitops) repo into your organization

### Login to OpenShift cluster

Replace the token and the api server url in the command below to login to the OpenShift cluster.

```sh
oc login --token=<token> --server=<api_server_url> # login to Openshift cluster
```

### Cloning repository

```sh
git clone https://github.com/<<your-username>>/rec-sys-gitops.git
cd rec-sys-gitops
```

<!-- ### Configuring model

This pattern deploys [IBM Granite 3.1-8B-Instruct](https://huggingface.co/ibm-granite/granite-3.1-8b-instruct) out of box. Run the following command to configure vault with the model Id.

```sh
# Copy values-secret.yaml.template to ~/values-secret-rag-llm-gitops.yaml.
# You should never check-in these files
# Add secrets to the values-secret.yaml that needs to be added to the vault.
cp values-secret.yaml.template ~/values-secret-rag-llm-gitops.yaml
```

To deploy a model that can requires an Hugging Face token, grab the [Hugging Face token](https://huggingface.co/settings/tokens) and accept the terms and conditions on the model page. Edit ~/values-secret-rag-llm-gitops.yaml to replace the `model Id` and the `Hugging Face` token.

```sh
secrets:
  - name: hfmodel
    fields:
    - name: hftoken
      value: null
    - name: modelId
      value: "ibm-granite/granite-3.1-8b-instruct"
  - name: minio
    fields:
    - name: MINIO_ROOT_USER
      value: minio
    - name: MINIO_ROOT_PASSWORD
      value: null
      onMissingValue: generate
``` -->

<!-- ### Provision GPU MachineSet

As a pre-requisite to deploy the application using the validated pattern, GPU nodes should be provisioned along with Node Feature Discovery Operator and NVIDIA GPU operator. To provision GPU Nodes

Following command will take about 5-10 minutes.

```sh
./pattern.sh make create-gpu-machineset
```

Wait till the nodes are provisioned and running.

![Diagram](images/nodes.png)

Alternatiely, follow the [instructions](./GPU_provisioning.md) to manually install GPU nodes, Node Feature Discovery Operator and NVIDIA GPU operator. -->

<!-- ### Deploy application

***Note:**: This pattern supports two types of vector databases, EDB Postgres for Kubernetes and Redis. By default the pattern will deploy EDB Postgres for Kubernetes as a vector DB. To deploy Redis, change the global.db.type to REDIS in [values-global.yaml](./values-global.yaml).

```yaml
---
global:
  pattern: rag-llm-gitops
  options:
    useCSV: false
    syncPolicy: Automatic
    installPlanApproval: Automatic
# Possible value for db.type = [REDIS, EDB]
  db:
    index: docs
    type: EDB  # <--- Default is EDB, Change the db type to REDIS for Redis deployment
main:
  clusterGroupName: hub
  multiSourceConfig:
    enabled: true
``` -->

Following commands will take about 15-20 minutes

### Deploying the pattern

```sh
./pattern.sh make install
```

### 1: Verify the installation

- Login to the OpenShift web console.
- Navigate to the Workloads --> Pods.
- Select the `rec-sys` project from the drop down.
- Following pods should be up and running.

![Pods](images/rag-llm.png) # TODO


### 2: Launch the application

- Click the `Application box` icon in the header, and select `Retrieval-Augmented-Generation (RAG) LLM Demonstration UI`

![Launch Application](images/launch-application.png)

- It should launch the application

  ![Application](images/application.png)

### 3: Generate the proposal document

- It will use the default provider and model configured as part of the application deployment. The default provider is a Hugging Face model server running in the OpenShift. The model server is deployed with this valdiated pattern and requires a node with GPU.
- Enter any company name
- Enter the product as `RedHat OpenShift`
- Click the `Generate` button, a project proposal should be generated. The project proposal also contains the reference of the RAG content. The project proposal document can be Downloaded in the form of a PDF document.

  ![Routes](images/proposal.png)

### 4: Add an OpenAI provider

You can optionally add additional providers. The application supports the following providers

- Hugging Face Text Generation Inference Server
- OpenAI
- NVIDIA

Click on the `Add Provider` tab to add a new provider. Fill in the details and click `Add Provider` button. The provider should be added in the `Providers` dropdown uder `Chatbot` tab.

![Routes](images/add_provider.png)

### 5: Generate the proposal document using OpenAI provider

Follow the instructions in step 3 to generate the proposal document using the OpenAI provider.

![Routes](images/chatgpt.png)

### 6: Rating the provider

You can provide rating to the model by clicking on the `Rate the model` radio button. The rating will be captured as part of the metrics and can help the company which model to deploy in prodcution.

### 7: Grafana Dashboard

By default, Grafana application is deployed in `llm-monitoring` namespace.To launch the Grafana Dashboard, follow the instructions below:

- Grab the credentials of Grafana Application
  - Navigate to Workloads --> Secrets
  - Click on the grafana-admin-credentials and copy the GF_SECURITY_ADMIN_USER, GF_SECURITY_ADMIN_PASSWORD
- Launch Grafana Dashboard
  - Click the `Application box` icon in the header, and select `Grafana UI for LLM ratings`
 ![Launch Application](images/launch-application.png)
  - Enter the Grafana admin credentials.
  - Ratings are displayed for each model.

![Routes](images/monitoring.png)

## Test Plan
GOTO: [Test Plan](./TESTPLAN.md)

## Licenses

EDB Postgres for Kubernetes is distributed under the EDB Limited Usage License
Agreement, available at [enterprisedb.com/limited-use-license](https://www.enterprisedb.com/limited-use-license).

