# Overview

Filler text.

# Azure Service Broker

Here we will prepare Azure with the base services we need for a Spring Starter App.

## Prepare Azure Blob Storage.

1. Create a JSON file called `azure-storage.json`:
    ```json
    {
    "resource_group_name": "my-resource-group",
    "storage_account_name": "mystorageaccountname",
    "location": "westus",
    "account_type": "Standard_LRS"
    }
    ```
1. Create an Azure Storage service instance by running the following command:

    `cf create-service azure-storage standard a-storage -c ./azure-storage.json`

    Look into your Azure portal and check the resource group and the storage account created

# Spring

Filler text.

## Create the Spring Starter App

Here we will create the Spring Starter App for Azure.

1. Go to start.spring.io and bootstrap a project with the following dependencies:
    
    * Web
    * Azure Support
    * Azure Storage

1. Fill out the Project Metadata with these values:
    
| Group  | Artifact  |
|---|---|
| `io.pivotal`  | `azure-storage-client`  |

1. Click the *Generate Project* button and your browser will download the `azure-storage-client.zip` file.
1. Unzip the project to `CN-Workshop/labs/azure`.
1. From your IDE, import this project as an existing Maven project.
1. Now we need to create the application. Open this Java class in your project in your IDE. This is a skeleton application class auto-generated by Spring Initializr for you. `src/main/java/io/pivotal/azurestorageclient/AzureStorageClientApplication.java`
1. Add the @RestController annotation to the Class definition and the method _upload_ to this class from the code snippet below. After you add this code, your class should look exactly like this snippet below.


    ```java
    package io.pivotal.azurestorageclient;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestParam;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.multipart.MultipartFile;

    import com.microsoft.azure.storage.CloudStorageAccount;
    import com.microsoft.azure.storage.blob.CloudBlobClient;
    import com.microsoft.azure.storage.blob.CloudBlobContainer;
    import com.microsoft.azure.storage.blob.CloudBlockBlob;

    @SpringBootApplication
    @RestController
    public class AzureStorageClientApplication {

        // for blob storage
        @Autowired
        private CloudStorageAccount cloudStorageAccount;

        @RequestMapping("/uploadfile")
        public boolean upload(@RequestParam("file") MultipartFile file,
                            @RequestParam("azureContainerName") String azureContainerName) {

            try {
                // Convert File to Byte Array
                final String fileName = file.getOriginalFilename();
                byte[] fileBytes = file.getBytes();

                // Upload File to Azure Storage Container
                CloudBlobClient client = cloudStorageAccount
                        .createCloudBlobClient();
                CloudBlobContainer container = client
                        .getContainerReference(azureContainerName);
                container.createIfNotExists();

                CloudBlockBlob blob = container.getBlockBlobReference(fileName);
                blob.upload(file.getInputStream(), fileBytes.length);

            } catch (Exception e) {
                e.printStackTrace();
            }
            return true;
        }

        public static void main(String[] args) {
            SpringApplication.run(AzureDemoApplication.class, args);
        }
    }
    ```

1. Compile the demo application with Maven:

    ```sh
    ./mvnw clean install
    ```

## Deploy to Cloud Foundry

1. In order to deploy the starter app to Cloud Foundry efficiently, we'll need to create a manifest. Create `CN-Workshop/labs/azure/manifest/yml` and populate it with these values.

    ```yaml
    ---
    applications:
    - name: azure-demo-app
    memory: 1G
    instances: 1
    path: target/MY-APP-JAR.jar
    ```

1. Now the application is ready to be pushed to Cloud Foundry.
    ```sh
    cf push --no-start
    ```

1. Now we have to bind the app to the Service Instance previously created.
    ```sh
    cf bind-service azure-demo-app a-storage
    ```

1. Now we need to start our application.
    ```sh
    cf start azure-demo-app
    ```

## Test Our App
