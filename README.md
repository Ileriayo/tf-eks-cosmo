# Cosmo Router - Infrastructure Setup

## Setup
1. Initialize terraform in the root directory
```sh
terraform init
```

2. Login to the Wundergraph UI and generate an API key. Once done, set it as an environment variable
```sh
export COSMO_API_KEY="cosmo_e04a4977ff9ba563e6e308d64ec1b47b"
```

3. Create a federated graph
```sh
npx wgc federated-graph create my-graph --namespace default --label-matcher team=A --routing-url http://localhost:3002/graphql
```

4. Create a subgraph
```sh
npx wgc subgraph publish my-subgraph --namespace default --schema schema.graphql --label team=A --routing-url http://localhost:3002/graphql
```

5. Then generate the graph api auth token
```sh
npx wgc router token create my-token -n default -g production
```

`Note`: A token will be displayed to stdout. Store it somewhere!

5. Create a `helm/values.yaml` file from the `helm/values.yaml.example` file. Then set the auth token here
```yaml
configuration:
  # -- The router token is used to authenticate the router against the controlplane (required)
  graphApiToken: "replace-me"
```

2. Apply the terraform code to create a VPC, an EKS Cluster, generate a kubconfig and install the Cosmo stack onto the Cluster
```
terraform apply
```


## Infra Architecture
This codebase consists of terraform scripts that provisions an EKS Cluster, and uses the helm provider to install the Cosmo Stack onto a Kubernetes Cluster.

- `vpc.tf`  
-- creates a vpc for which the EKS cluster resides    

- `main.tf`  
-- contains a module `eks` to provision an EKS cls=uster     
-- contains another module to generate the kubeconfig file, and then the file is stored locally using the `local_file` resource     
-- creates a helm release (using the generated kubeconfig file to authenticate to the cluster)  

Note: The helm chart used in the codebase was gotten from `https://artifacthub.io/packages/helm/cosmo-platform/cosmo`

## Future plans
1. Use a remote backend for terraform state management
2. Deploy the infrastructure in the AWS region closest to the customers
2. Follow the setup for production deployments on Cosmo docs

## Troubleshooting
- The postgres pod failed to start and was stuck in a pending state, and thus other components like the controlplane coould not connect to the database and kept crashing