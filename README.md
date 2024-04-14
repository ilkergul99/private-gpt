# private-gpt
PrivateGPT repository as a server with minor modifications for the Coding challenge

When setting up a server application that heavily relies on external dependencies and specific configurations like private-gpt, it's crucial to understand the modifications needed to adapt the server to your specific requirements. Here’s an extensive explanation of setting up the private-gpt server:

### Cloning the Repository
First, clone the private-gpt repository from GitHub:
```bash
git clone https://github.com/zylon-ai/private-gpt.git
```
This repository contains the core application for the private-gpt server, which is designed to handle natural language processing tasks using large language models.

### Why Use the Current Repository Version
The choice to use the latest version from the GitHub repository, instead of a specific release like 0.3.0, is due to bugs found in the older version. These bugs particularly affected the server's ability to respond within a given time range, which is a critical functionality, especially when considering the use in environments with limited memory resources.

### Legal and Ethical Considerations
It’s essential to respect the licensing agreements of software. Distributing a modified version of the software without proper adherence to its license can lead to legal and ethical issues. Therefore, modifying and using the repository locally without distributing it further is a safer approach, respecting the original creators' rights and terms.

### Preparing the Development Environment
After cloning the repository, you can open it in an Integrated Development Environment (IDE) like Visual Studio Code to make necessary changes. This setup will allow you to integrate the server more seamlessly with your existing project infrastructure.

### Configuring the Docker Environment
Modify the `docker-compose.yaml` file to ensure that it aligns with your project's network settings and volume management. This step is crucial for integrating the server within a dockerized environment, enabling it to communicate effectively with other services such as a client application or another server like Ollama.

### Adjusting Configuration Files
- **settings.py**: Located in the `./private-gpt/settings/` directory, this file should be adjusted to reflect your specific configurations, such as database connections, service endpoints, and other critical operational parameters.
- **settings.yaml**: This file should also be modified to align with the environmental variables and specific settings that your deployment scenario requires.

### Deployment and Testing
After all configurations are adjusted:
1. Build the Docker container using the modified Dockerfile and docker-compose.yaml. This step ensures that all dependencies and environment settings are correctly encapsulated.
2. Deploy the application in a test environment to verify functionality. Pay close attention to how it handles requests and interacts with other services, especially under different network configurations or load conditions.
3. Perform comprehensive testing to ensure that all endpoints function as expected and that the server meets the performance and reliability standards required for your application.

### Continuous Updates and Maintenance
Keep the server updated with the latest changes from the original repository to benefit from bug fixes, security patches, and performance improvements. Regularly pull updates from the original repository and merge them with your custom configurations, testing extensively each time to ensure compatibility.

### Conclusion
Setting up the private-gpt server involves not only technical adjustments but also a careful consideration of licensing and compatibility with existing systems. By following these detailed steps, you can effectively integrate and use the private-gpt server in your projects, ensuring robust, reliable, and legally compliant operation.


### Docker Configuration

- **Dockerfile**: Configures the Python environment, installs necessary libraries, and sets up the working directory inside the container.
- **docker-compose.yml**: Defines services, configuration for the `client-app` including volume mapping for persistent data storage and port forwarding.

### Docker Network Configuration Explanation

This project utilizes Docker Compose to manage multi-container Docker applications, facilitating clear separation of responsibilities and secure internal communications between services. Here is an overview of the Docker networking setup used:

#### Network Setup

Two Docker networks are configured to handle inter-service communications securely and effectively:

1. **`my-app-network`**:
   - **Type**: External
   - **Purpose**: Facilitates communication between the Client application (`client-app`) and the PrivateGPT service (`private-gpt`).
   - **Security**: Ensures that external interactions are limited to what is necessary, i.e., client to server communication without exposing internal components like Ollama.

2. **`private-gpt_internal-network`**:
   - **Type**: Bridge
   - **Purpose**: Used exclusively for internal communication between the PrivateGPT service and the Ollama service.
   - **Security**: Restricts access to Ollama, ensuring that only PrivateGPT can interact with it. This network isolation prevents external entities, including the client, from accessing sensitive internal services.

#### Service Configuration

- **Client Application (`client-app`)**:
  - **Network**: Connected to `my-app-network` to send requests to the PrivateGPT service.
  - **Ports**: Exposes port 4000 externally, mapped to the internal port 8000 for web access.
  - **Volumes**: Mounts local directories for documents, output, results, and source files, facilitating data persistence and access.
  - **Environment**: Uses `PYTHONUNBUFFERED=1` to force the Python runtime to output directly and avoid buffering.

- **PrivateGPT Service**:
  - **Networks**:
    - Connected to both `my-app-network` (for client interactions) and `private-gpt_internal-network` (for secure communication with Ollama).
  - **Ports**: Listens from port 8080 for requests from the **client-app**
  - **Environment**: Configured with specific profiles and modes tailored to operational requirements.
  - **Volumes**: Includes a volume for local data, ensuring that the service has access to necessary files and configurations.

- **Ollama Service**:
  - **Network**: Only connected to `private-gpt_internal-network` to ensure that all interactions are confined to authorized services.
  - **Volumes**: Mounts a directory for models, which Ollama requires to function.
  - - **Ports**: Listens from port **11434** for requests from **private-gpt**

#### Security and Accessibility

By designing the network configurations as described:
- **External Accessibility**: Only the necessary services are exposed to external access, minimizing potential vulnerabilities.
- **Internal Communication Security**: Use of a private internal network for sensitive components like Ollama enhances security by limiting accessibility to trusted services.

### Pre-Setup: Creating the External Network

Before you can run the Docker Compose configuration, you must ensure that the external network (`my-app-network`) is already created since it is referenced in the Docker Compose file. This network allows the client application to communicate securely with the PrivateGPT service.

To create this external network, execute the following command:

```bash
docker network create my-app-network
```
This command sets up a new network which will be our external network for communication between **client** and **server**

### Launching Services

To start all services configured in the Docker Compose setup, use the following command from the root of the project directory:
```bash
docker-compose up -d --build
```
This command will initialize all services in detached mode, allowing them to run in the background.

### Verifying Setup

After starting the services, it's good practice to check if all containers are correctly connected to their respective networks. You can inspect each network to see which containers are attached:

- **Inspect the external network**:
  ```bash
  docker network inspect my-app-network
  ```

- **Inspect the internal network**:
  ```bash
  docker network inspect private-gpt_internal-network
  ```

These commands provide detailed information about the networks, including which services are connected to them. This helps confirm that the network configuration aligns with the security and operational requirements outlined.

### Network Security and Management

By managing network connectivity as described:
- **Controlled Exposure**: Only necessary services are exposed to the external network, limiting potential attack surfaces.
- **Secured Internal Communications**: The internal network ensures that critical services like Ollama are accessible only to specific components, enhancing the security posture.

For additional security, consider implementing network policies that further restrict traffic between services according to the least privilege principle.