## <p align=left>Dockerize ASP.NET Core Application with MSSQL Server Database<br> <br> </p>
| **SL** | **Topic** |
| --- | --- |
| 01 | [Application Setup](#01) |
| 02 | [Application Dockerfile Setup](#02) |
| 03 | [Database Dockerfile Setup](#03) |
| 04 | [Docker-Compose File Setup](#04) |
| 05 | [Environment Variable Setup](#05) |
| 06 | [Build and Up the Docker-Compose](#06) |

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

- This portion of the lines is for the web application. It will build the **Dockerfile** and open the port **80** and keep the application container under a network named **app-network**. It will be dependent on **db** portion.
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
Run ````ls -a```` command, you can see a file named **.env**. Open the file by running ````vi .env`````this command. Here I have declared the database credentials.
````
SA_PASSWORD=rootpa@sw0rdmysql
ACCEPT_EULA=Y
````

### <a name="06">:diamond_shape_with_a_dot_inside: &nbsp;Build and Up the Docker-Compose</a> 
Now our all necessry files are ready. It's time to build and up our docker-compose file to create image and run the container. 
- Run this command to do that: ````docker-compose up -d --build````.
- Run this command to show the list of running container: ````docker ps````. There two container will be running, One is web app container and other one is the database.
- Now go to your browser and enter your server ip or localhost then hit enter button. You will see a page like below:
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-1.png" alt="Login Page"><br>
  Now provide the **Username: Sadik** and **Password: admin**
  This credentials will fetch and check from database and redirect you into the next page.
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-2.png" alt="Main Page"> <br><br>

You have successfully did this. Now if you want to connect the container database into **SQL Server Management Studio (SSMS)** then you need to do this following steps:
- Open SSMS:- Press windows key --> Scroll down and click Microsoft SQL Server Tools --> Click Microsoft SQL Server Management Studio
- Provide the Credentials:
  **Server Type:** Database Engine
  **Server Name:** IP Address of the server/Localhost, Port _(In my case my IP address of the VM is 20.25.66.233 and Port we provided in the docker-compose file 1433)_
  **Authentication:** Select **SQL Server Authentication** from the dropdown
  **Login:** sa _(We provided **sa** as a Login Username)_
  **Password:** rootpa@sw0rdmysql _(We provided this password in the .env file)_
  
  Your Provided information will look like below:
  <br> <br> <img src= "https://github.com/Shadikul-Islam/ASP.NET-Core-Projects/blob/master/Dockerize%20ASP.NET%20Core%20Application%20with%20MSSQL%20Server%20Database/Images/Image-2.png" alt="Main Page"> <br><br>
  
  Now click **Connect**. You will be connected to the database.
  
  
  
  
...
### This documentation is under construction.  I hope it will be completed soon. Thank you for visiting 🙂.
