# AspDotNetCoreSampleWebAppDocker
### How to Run
1. Download the project & Extract the zip file. 
2. Build the Dockerfile to create an Image by running this command:
   ```docker build -t ImageName . ```
3. Now Run this comand to create container with defined port 8080:
   ```docker run -d -p 8080:80 --name ContainerName ImageName ```
4. Now copy your IP and use port 8080 (Example: http://10.10.10.10:8080 ). If you are using localhost then browse like this: http://localhost:8080
5. You will see your sample asp.net core application webpage which is hosted by docker.

#### Happy Learning!
