## <p align=left>Dockerize ASP.NET Core Application with MSSQL Server Database<br> <br> </p>
| **SL** | **Topic** |
| --- | --- |
| 01 | [Application Setup](#01) |
| 02 | [Dockerfile Setup](#02) |
| 03 | [](#03) |
| 04 | [](#04) |
| 05 | [](#05) |
| 06 | [](#06) |

### <a name="01">:diamond_shape_with_a_dot_inside: &nbsp;Application Setup</a> 
- Download the application source code by running this command: ````git clone https://github.com/Shadikul-Islam/ASP.NET-Core-Projects.git````.
- Now go to the folder by running this command: ````cd ASP.NET-Core-Projects/'Dockerize ASP.NET Core Application with MSSQL Server Database'````.
- Now you can see all of the files by running  ````ls -a```` command.
- The most important thing is we need to know the place where we need to set the database credentials of this project. In the **Models** folder, we have a file named 
**MyDBContext**. We need to open that file. Run this command to open it: ````vi Models/MyDBContext.cs````. We can see **SqlConnection** under the **MyDBContext**
class. Just change this line like this: ````SqlConnection con = new SqlConnection("Data Source=db;Initial Catalog=dbbackup;Integrated Security=false; User Id=sa; Password=rootpa@sw0rdmysql");````.
Here our **Database Source** will be ````db```` which we will declare on the **docker-compose** later. **Initial Catalog** will be the database name. Here our database name is ````dbbackup````. **User Id** is the username which is ````sa```` and **Password** is the ````rootpa@sw0rdmysql```` in our case. You can set your own password which need to set it in the **docker-compose .env file** later.

### <a name="02">:diamond_shape_with_a_dot_inside: &nbsp;Dockerfile Setup</a> 
By running this command: ````vi Dockerfile```` you can open the Docekerfile and you can see my Dockerfile of this project. Let's discuss in detail what the Dockerfile will do.
- These portions of the lines will create an intermediate image from the base image and expose the ports.
```Bash
FROM mcr.microsoft.com/dotnet/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
```
- These portions of the lines will copy the code into the working source directory, restore the nugets packages, build the code and publish the code into Publish directory.
````Bash
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
- These portions of the lines will copy the publish directory into the final directory and it will create the final production-ready image which runs when the container is started running.
````Bash
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "K8STestApp.dll"]
````
Let's see the full Dockerfile.
````Bash
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

