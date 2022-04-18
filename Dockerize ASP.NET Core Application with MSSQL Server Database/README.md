## <p align=left>Dockerize ASP.NET Core Application with Microsoft SQL Server Database<br> <br> </p>
| **SL** | **Topic** |
| --- | --- |
| 01 | [Application Setup](#01) |
| 02 | [Application Dockerfile Setup](#02) |
| 03 | [Database Dockerfile Setup](#03) |
| 04 | [Docker-Compose File Setup](#04) |
| 05 | [Environment Variable Setup](#05) |
| 06 | [Build and Up the Docker-Compose](#06) |
| 07 | [Restore the Database](#07) |
| 08 | [Create Image from Container](#08) |
| 09 | [Upload Images into Docker Hub](#09) |
| 10 | [Run Application from Docker Hub Image](#10) |
| 11 | [Summary of What We Have Done](#11) |

### <a name="01">:diamond_shape_with_a_dot_inside: &nbsp;Application Setup</a> 
- Download the application source code by running this command: ````git clone https://github.com/Projects-of-Shadikul/Project-10.git````.
- Now go to the folder by running this command: ````cd Project-10````.
- Now you can see all of the files by running  ````ls -a```` command.
- The most important thing is we need to know the place where we need to set the database credentials of this project. In the **Models** folder, we have a file named 
**MyDBContext**. We need to open that file. Run this command to open it: ````vi Models/MyDBContext.cs````. We can see **SqlConnection** under the **MyDBContext**
class. Just change this line like this: ````SqlConnection con = new SqlConnection("Data Source=db;Initial Catalog=dbbackup;Integrated Security=false; User Id=sa; Password=rootpa@sw0rdmysql");````.
Here our **Database Source** will be ````db```` which we will declare on the **docker-compose** later. **Initial Catalog** will be the database name. Here our database name is ````dbbackup````. **User Id** is the username which is ````sa```` and **Password** is the ````rootpa@sw0rdmysql```` in our case. You can set your own password which need to set it in the **docker-compose .env file** later.

### <a name="02">:diamond_shape_with_a_dot_inside: &nbsp;Application Dockerfile Setup</a> 
By running this command: ````vi Dockerfile```` you can open the Docekerfile and you can see my Dockerfile of this project. Let's discuss in detail what the Dockerfile will do.
- This portion of the lines will create an intermediate image from the base image and expose the ports.
```
FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
```
- This portion of the lines will copy the code into the working source directory, restore the nugets packages, build the code and publish the code into Publish directory.
````
FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
WORKDIR /src
COPY ["./K8STestApp.csproj", "./"]
RUN dotnet restore "./K8STestApp.csproj"
COPY . .
WORKDIR "/src/K8STestApp"
RUN dotnet build "/src/K8STestApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "/src/K8STestApp.csproj" -c Release -o /app/publish
````
- This portion of the lines will copy the publish directory into the final directory and it will create the final production-ready image which runs when the container is started running.
````
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "K8STestApp.dll"]
````
Let's see the full Dockerfile.
````
FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build
WORKDIR /src
COPY ["./K8STestApp.csproj", "./"]
RUN dotnet restore "./K8STestApp.csproj"
COPY . .
WORKDIR "/src/K8STestApp"
RUN dotnet build "/src/K8STestApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "/src/K8STestApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "K8STestApp.dll"]
````

### <a name="03">:diamond_shape_with_a_dot_inside: &nbsp;Database Dockerfile Setup</a> 
Now we need to setup the Database Dockerfile to build the database.
- Run this command ````vi dbDockerfile```` to open my Dockerfile that was created for the database.
- This Dockerfile will create an **MSSQL Server** image, set **home** as a working directory, and copy the database backup file into that directory.
````
FROM mcr.microsoft.com/mssql/server
WORKDIR /home
COPY ./dbbackup.bak .
````

### <a name="04">:diamond_shape_with_a_dot_inside: &nbsp;Docker-Compose File Setup</a>
By running this command: ````vi docker-compose.yml```` you can open the docker-compose and you can see my docker-compose file of this project. Let's discuss in detail what the docker-compose.yml file will do.

- This portion of the lines is for the web application. It will build the **Dockerfile** and open the port **80** and keep the application container under a network named **app-network**. It will be dependent on **db**.
````
version: "3.9"
services:
    web:
        build: .
        ports:
            - "80:80"
        depends_on:
            - db
        networks:
            - app-network
````
- This portion of the lines is for the database. It will build the **dbDockerfile**. Set the Database **sa** password. Open port **1433** and keep the database container under a network named **app-network**.

````
    db:
        build:
           context: ./
           dockerfile: dbDockerfile
        environment:
            SA_PASSWORD: ${SA_PASSWORD}
            ACCEPT_EULA: ${ACCEPT_EULA}
        ports:
         - "1433:1433"
        networks:
            - app-network
````

- This portion of the lines is defying the Network as a **bridge network** for the container of the **app-network**.

````
networks:
  app-network:
    driver: bridge
````

### <a name="05">:diamond_shape_with_a_dot_inside: &nbsp;Environment Variable Setup</a> 
Run ````ls -a```` command, you can see a file named **.env**. Open the file by running ````vi .env```` this command. Here I have declared the database credentials.
````
SA_PASSWORD=rootpa@sw0rdmysql
ACCEPT_EULA=Y
````

### <a name="06">:diamond_shape_with_a_dot_inside: &nbsp;Build and Up the Docker-Compose</a> 
Now all the necessary files are ready. It's time to build and up our docker-compose file to create the image and run the container. 
- Run this command to do that: ````docker-compose up -d --build````.
- Run this command to show the list of running containers: ````docker ps````. There are two containers that will be running, One is the web app container and the other one is the database.
- Now go to your browser and enter your server IP or localhost then hit enter button. You will see a page like the below:
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-1.png" alt="Login Page"><br>
Our web application is running successfully. Now it's time to restore the database.

### <a name="07">:diamond_shape_with_a_dot_inside: &nbsp;Restore the Database </a>  
Now we need to connect the container database to **SQL Server Management Studio (SSMS)**. To connect we need to do the following steps:
- Open SSMS:- Press windows key --> Scroll down and click Microsoft SQL Server Tools --> Click Microsoft SQL Server Management Studio
- Provide the Credentials:
  **Server Type:** Database Engine
  
  **Server Name:** IP Address of the server/localhost, Port _(In my case my IP address of the VM is 20.25.66.233, and Port we provided in the docker-compose file 1433)_
  
  **Authentication:** Select **SQL Server Authentication** from the dropdown
  
  **Login:** sa _(**sa** will be the database Login Username)_
  
  **Password:** rootpa@sw0rdmysql _(We provided this password in the .env file before)_
  
  Your Provided information will look like the below:
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-3.png" alt="DB Credentials"> <br><br>
  Click **Connect**. You will be connected to the database.
  
Now we will restore the database that we copied into the database container from the host machine by building the **dbDockerfile**. To restore the database we need to follow these steps:
- Go to SSMS. We already connected with the DB container so just right-click the **Database** and click the **Restore Database** options. 
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-4.png" alt="Restore Database"> <br><br>
- A new window will appear, Select **Device** and click on the right side three dots. 
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-5.png" alt="Restore Database"> <br><br>
  Again a new window will open click on **Add**. 
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-6.png" alt="Restore Database"> <br><br>
  Again a new window will open which will show the file system of this container. Scroll up/down and select the **home** folder. On the right side, you will see the database backup file named **dbbackup.bak**.
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-8.png" alt="Restore Database"> <br><br>
  Select the backup file and click Ok, then again click Ok. Now you can see that your dbbackup.bak file has been selected and the database name of this file is **mytestdb**.
<br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-7.png" alt="Restore Database"> <br><br>
- Click Ok. Your database will be successfully restored. You can expand databases and see the **mytestdb** database.
- Now again go to the browser, On the application login page provide the **Username: _Sadik_** and **Password: _admin_**. These credentials will fetch and check from the database and redirect you to the next page.
<br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-2.png" alt="Main Page"> <br><br>

This is the main page of this sample application.

### <a name="08">:diamond_shape_with_a_dot_inside: &nbsp;Create Image from Container</a> 
We have the application and database containers which is perfectly working for our project. Now we will create our own application image from those containers. To do that we need to follow the following steps:
- First, run this command ````docker ps```` to check the running containers.
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-10.png" alt="Docker Running Containers"> <br><br>
  We can see that our container's names are **project-10_web_1** and **project-10_db_1**.
- Run this command to create an image from the container: ````docker commit ContainerName ImageName````. In our case the command will be for Web Container: ````docker commit project-10_web_1 dot-net-core-app````. For Database Container: ````docker commit project-10_db_1 dot-net-core-db````. Here our application image name will be **dot-net-core-app** and the database image name will be **dot-net-core-db**.
- Our application image has been created. We can check it by running this command: ````docker images````.
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-11.png" alt="Docker Images"> <br><br>
- Now we have to give the tag of both images. I will provide the same tag as the image name with the version. Run this command to give a tag: ````docker image tag ImageName DockerHubUserName/ImageTag:Version````. In our case it will be for the web image: ````docker image tag dot-net-core-app shadikul/dot-net-core-app:v1````. For DB image: ````docker image tag dot-net-core-db shadikul/dot-net-core-db:v1````. After running this command, Images will be created with the tag. You can see this by running the ````docker images```` command like below:
 <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-12.png" alt="Docker Images"> <br><br>
 
### <a name="09">:diamond_shape_with_a_dot_inside: &nbsp;Upload Images into Docker Hub</a> 
- Now we will login into https://hub.docker.com from the browser, and also login from our terminal where we are running all of our docker commands. To login docker hub from the terminal you need to follow the below steps:
  -  Run this command: ````docker login````
  -  It will ask for the username and password. Provide those credentials. Now you have logged in successfully.
- Now it's time to upload our images into docker hub. Run this command: ````docker image push DockerHubUserName/ImageTag:Version````. In our case we need to run these commands to upload our two images one by one:
  ````
  docker image push shadikul/dot-net-core-app:v1
  docker image push shadikul/dot-net-core-db:v1
  ````
  You will see like this:
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-13.png" alt="Docker Push"> <br><br>
- Our images have been uploaded into docker hub successfully. To check this visit https://hub.docker.com/repositories from a browser and you can see your images like below: 
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-14.png" alt="Dockerhub Images"> <br><br>

### <a name="10">:diamond_shape_with_a_dot_inside: &nbsp;Run Application from Docker Hub Image</a> 
So, we are on the way to the end. At this moment we have our own application and database image. So if we run those images as a container then we can see our application easily. This step is necessary for the project when Developer complete their development and needs to test by the testing team. We can give them those images with a docker-compose file and the testing team can easily run that application without any issue. Let's do this.
- **Prepare a Docker-Compose file:**
  First, we need to create a new folder by running this command: ````mkdir docker-project````. Go inside that folder by running this command: ````cd docker-project````. Now create a docker-compose file by running this command: ````vi docker-compose.yml````. Now let's write docker-compose file for our web application image.
  ````
  version: "3.9"
  services:
    web:
        image: "shadikul/dot-net-core-app:v1"
        ports:
            - "80:80"
        depends_on:
            - db
        networks:
            - app-network
  ````
  We are pulling our **dot-net-core-app:v1** image that we pushed before. We run this application in **80 ports** and which will depend on db and the network will be **app-network**. We already discussed all of those things above. Now the turn is for the database.
  ````
    db:
        image: "shadikul/dot-net-core-db:v1"
        environment:
            SA_PASSWORD: ${SA_PASSWORD}
            ACCEPT_EULA: ${ACCEPT_EULA}
        ports:
            - "1433:1433"
        networks:
            - app-network
   ````
  We are pulling our **dot-net-core-db:v1** image that we pushed before. We run this application in **1433 ports** and the network will be **app-network**. We already discussed all of those things above. Now we need to defien the Network part.
  ````
  networks:
    app-network:
      driver: bridge
  ````
  Now our docker-compose.yml file is ready. Put all of those parts together like below and Save it.
  ````
  version: "3.9"
  services:
    web:
        image: "shadikul/dot-net-core-app:v1"
        ports:
            - "80:80"
        depends_on:
            - db
        networks:
            - app-network
    db:
        image: "shadikul/dot-net-core-db:v1"
        environment:
            SA_PASSWORD: ${SA_PASSWORD}
            ACCEPT_EULA: ${ACCEPT_EULA}
        ports:
            - "1433:1433"
        networks:
            - app-network
  networks:
    app-network:
      driver: bridge
  ````
  
- **Prepare a .env file:**
  Now we need to define the **.env** file for database credentials. Run this command ````vi .env```` and paste the below credentials.
  ````
  SA_PASSWORD=rootpa@sw0rdmysql
  ACCEPT_EULA=Y
  ````
  Now all things are ready to run our application.
  
- **Create Container and Run the Application:**
  Run this command ````docker-compose up -d --build````. Your application will be up and running you can see your application from the browser. Now we need to restore the database. You can follow this step again [Restore the Database](#07) to restore. After completing the restore go to the application from the browser and login with the credentials. You should login successfully.

### <a name="11">:diamond_shape_with_a_dot_inside: &nbsp;Summary of What We have Done</a>
Let's recap what we have done.
1. First, We download our application by using git clone.
2. We setup our application Dockerfile and database Dockerfile.
3. We setup our docker-compose file and also setup our .env file.
4. We build and Up our docker-compose file and we can see our application from the browser.
5. We connect our database container to SSMS and restore our database.
6. We create our own image and upload it into the docker hub.
7. Finally, we run our applicaion using our uploaded docker hub image.



#### :diamond_shape_with_a_dot_inside: &nbsp;That’s it. We have learned How to Dockerize ASP.NET Core Application with Microsoft SQL Server Database. :diamond_shape_with_a_dot_inside: &nbsp;Happy Learning. :diamond_shape_with_a_dot_inside: &nbsp;
