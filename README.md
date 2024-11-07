# Customer Demos - MI Version

## Exposing a SOAP Endpoint as a REST API

1. **Set up Visual Studio Code**  
   Launch Visual Studio Code with the **Micro Integrator (MI) for VS Code** extension installed.

2. **Configure the Back-end Service**  
   - Download the [back-end service](https://github.com/wso2-docs/WSO2_EI/blob/master/Back-End-Service/axis2Server.zip).
   - Extract the zip file and navigate to the `axis2Server/bin/` directory.
   - Start the SimpleStockQuote back-end service:
     - **MacOS/Linux**: `sh axis2server.sh`
     - **Windows**: `axis2server.bat`

3. **Invoke the API**  
   Once the server is up, invoke the REST API by running the following command:

   ```bash
    curl -v http://127.0.0.1:8290/stockquote/view/IBM
   ```

## Secure a REST API

### Steps to Secure a REST API in the Micro Integrator

Follow these steps to set up a secure REST API using the Micro Integrator.

1. **Create a Micro Integrator Project**

   - Set up your MI project as outlined in the initial steps of the project setup guide.

2. **Configure an RDBMS User Store**

   - To use a relational database as the user store, add the JDBC driver for your database to the `<MI_HOME>/lib` directory.

   - Configure the Micro Integrator to connect to the RDBMS user store by modifying the `deployment.toml` file (located in the `<MI_HOME>/conf` directory). Below is a sample configuration for connecting to a MySQL database:

     ```toml
     [[datasource]]
     id = "WSO2CarbonDB"
     url = "jdbc:mysql://localhost:3306/userdb"
     username = "root"
     password = "root"
     driver = "com.mysql.jdbc.Driver"
     pool_options.maxActive = 50
     pool_options.maxWait = 60000
     pool_options.testOnBorrow = true
     ```

     Replace `url`, `username`, and `password` with the appropriate credentials for your database.

3. **Set Up the User Store Manager**

   - In the `deployment.toml` file, add the JDBC user store manager settings under the `[user_store]` heading to manage authentication. Below is a sample configuration:

     ```toml
     [user_store]
     class = "org.wso2.micro.integrator.security.user.core.jdbc.JDBCUserStoreManager"
     type = "database"

     # Set to true if you want to disable write access to the user store.
     read_only = true
     ```

   - This configuration enables the Micro Integrator to authenticate users against the specified RDBMS database.

4. **Deploy and Secure the API**

   - Once the RDBMS user store is configured, deploy the secured API as part of your MI project. The Micro Integrator will now use the RDBMS user store to authenticate requests made to the REST API.

### Testing the Secured API

Once the above configurations are in place, you can test your secured API by invoking it with valid credentials stored in your RDBMS user store. Make sure to include authentication headers (e.g., Basic Auth) in your requests.

## Deploy the MI Project

## Deploying on Docker

### 1. Build the Docker Image

- Follow the [Docker build guide](https://mi.docs.wso2.com/en/latest/develop/deploy-artifacts/#build-docker-image) to create your image.

### 2. Run the Docker Image

- Use the following command to start the Docker container for your MI project:

   ```bash
   docker run -p 8080:8080 -p 8253:8253 -p 8290:8290 -p 3306:3306 -p 9000:9000 soap_as_rest:1.0.0
   ```

```curl
curl --location https://localhost:8290/stockquote/view/IBM
```

## Deploying on Kubernetes

### Prerequisites

To deploy your Micro Integrator (MI) project on Kubernetes, ensure the following prerequisites are met:

1. **Install required tools:** Ensure **Git**, **Helm**, and **kubectl** (Kubernetes client) are installed on your system.
2. **Set up a Kubernetes cluster** if not already configured.
3. **Install the NGINX Ingress Controller** to manage external access to services.

Refer to the [WSO2 MI Kubernetes setup guide](https://mi.docs.wso2.com/en/latest/install-and-setup/setup/deployment/deploying-micro-integrator-with-helm/) for more detailed information.

### Setup Steps

1. **Clone the Helm Resources**

   - Open a terminal, navigate to your preferred directory, and clone the Helm resources repository:

     ```bash
     git clone https://github.com/wso2/helm-mi.git
     ```

   - This command will create a local copy of the `helm-mi` repository, which includes the necessary Helm resources for deploying WSO2 MI on Kubernetes.

2. **Configure Docker Image Details**

   - Open the `values.yaml` file (such as `values_local.yaml`) in the Helm resources directory, and specify your Docker registry and image configuration. This enables Kubernetes to pull the correct MI image:

     ```yaml
     containerRegistry: "<docker_registry>"
     wso2:
         deployment:
             image:
                 repository: "<custom_mi_repository>"
                 tag: "<custom_image_tag>"
     ```

3. **Deploy Using Helm**

   - Use Helm v3 to deploy the MI project by executing the following command:

     ```bash
     helm install mi-test ./ -f ./values_local.yaml -n mi
     ```

   - This command will deploy the MI instance under the namespace `mi` with the specified configurations.

4. **Retrieve the External IP**

   - After deployment, get the external IP (EXTERNAL-IP) of the Ingress resource by running:

     ```bash
     kubectl get ing -n mi
     ```

   - This IP allows you to access the deployed MI services externally.

5. **Set up Port Forwarding (if required)**

   - If using **Rancher Desktop** or a similar tool, navigate to **Port Forwarding** and assign a local port for pass-through HTTPS that matches your configuration. This allows local access to the Kubernetes service.

6. **Invoke the API on Kubernetes**

   - To test the deployed API, open a terminal and execute:

     ```bash
     curl --location https://localhost:<LOCAL-PORT>/stockquote/view/IBM
     ```

   - Replace `<LOCAL-PORT>` with the actual port configured for local access. This command confirms that your API is successfully running within the Kubernetes cluster.

## Additional Resources

For further reading and detailed examples on configuring, deploying, and securing REST APIs with WSO2 Micro Integrator, check out the following resources.

- [Enabling REST to SOAP](https://mi.docs.wso2.com/en/latest/learn/examples/rest-api-examples/enabling-rest-to-soap/)
- [Securing REST APIs](https://mi.docs.wso2.com/en/latest/learn/examples/rest-api-examples/securing-rest-apis/)
- [Setting up a User Store in MI](https://mi.docs.wso2.com/en/latest/install-and-setup/setup/user-stores/setting-up-a-userstore-in-mi/)
- [Applying Security to an API](https://mi.docs.wso2.com/en/latest/develop/advanced-development/applying-security-to-an-api/)
- [Deploying Artifacts in WSO2 Micro Integrator](https://mi.docs.wso2.com/en/latest/develop/deploy-artifacts)
- [Docker Sample](https://mi.docs.wso2.com/en/latest/learn/samples/docker-sample/)
- [Deploying Micro Integrator with Helm](https://mi.docs.wso2.com/en/latest/install-and-setup/setup/deployment/deploying-micro-integrator-with-helm/)

For more details on deploying WSO2 Micro Integrator on Kubernetes, consult the [official WSO2 documentation](https://mi.docs.wso2.com/en/latest/).
