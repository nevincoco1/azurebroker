# Connecting Snyk UI with on-prem Azure repos

Snyk Broker proxies access between snyk.io and your Git repository, such as Azure Repos.

The Broker server and client establish an applicative tunnel, proxying requests from snyk.io to the Git (fetching manifest files from monitored repositories), and vice versa (webhooks posted by the Git).

The Broker client runs within the user's internal network, keeping sensitive data such as Git tokens within the network perimeter. The applicative tunnel scans and adds only relevant requests to an approved list, narrowing down the access permissions to the bare minimum required for Snyk to actively monitor a repository.

## Usage

The Broker client is published as a set of Docker images, each configured for a specific Git. Standard and custom configuration is performed with environment variables as described below, per integration type.

### Prerequisites

#### Set up the network

To run both the broker client and the broker agent, establish a network connection between them. 

Run docker network create <network>
For example:
`docker network create mySnykBrokerNetwork`
You can confirm that it was created by running `docker network ls`, this will show results like this:
```  NETWORK ID     NAME                 DRIVER     SCOPE
  d1353a2b0f66   mySnykBrokerNetwork  bridge     local 
```
### Set up the Broker Client          

To use the Broker client with [Azure](https://azure.microsoft.com/en-us/services/devops/), run `docker pull snyk/broker:azure-repos` tag. The following environment variables are mandatory to configure the Broker client:

- `BROKER_TOKEN` - the Snyk Broker token, obtained from your Azure Repos integration settings view (app.snyk.io).
- `AZURE_REPOS_TOKEN` - an Azure Repos personal access token. [Guide](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page) how to get/create the token. Required scopes: ensure Custom defined is selected and under Code select _Read & write_
- `AZURE_REPOS_ORG` - organization name, which can be found in your Organization Overview page in Azure
- `AZURE_REPOS_HOST` - the hostname of your Azure Repos Server deployment, such as `your.azure-server.domain.com`. If you are using the Azure DevOps, it is `dev.azure.com`.
- `PORT` - the local port at which the Broker client accepts connections. 
- `BROKER_CLIENT_URL` - the full URL or IP of the Broker client as it will be accessible by your Azure Repos' webhooks, such as `http://my.broker.client:8000` or `http://10.10.10.150:8000`

#### Command-line arguments

You can run the docker container by providing the relevant configuration:

```
docker run --restart=always \
           -p 8000:8000 \
           -e BROKER_TOKEN=<secret-broker-token> \
           -e AZURE_REPOS_TOKEN=<secret-azure-token> \
           -e AZURE_REPOS_ORG=<org-name> \
           -e AZURE_REPOS_HOST=<your.azure-server.domain.com> \
           -e BROKER_CLIENT_URL=http://<my.broker.client:8000> \
           -e GIT_CLIENT_URL=http://code-agent:3000 \
           --network mySnykBrokerNetwork \
           -e ACCEPT=/private/accept.json \
           -v /path/to/acceptdir:/private \
           -e PORT=8000 \
       snyk/broker:azure-repos
```       

### Set up Code Agent

First, pull the code-agent image:
`docker pull snyk/code-agent`

The following environment variables are mandatory to configure the code agent:
**SNYK_TOKEN** - your Snyk API token.
**PORT** - the local port, for which the code agent accepts connections, Default is 3000.
           
To run the code-agent:
``` 
docker run --name code-agent \
     -p 3000:3000 \
     -e PORT=3000 -e SNYK_TOKEN=<Snyk API token> --network mySnykBrokerNetwork \
     snyk/code-agent
```
