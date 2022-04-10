## <p align=left>Dockerize ASP.NET Core Application with MSSQL Server Database<br> <br> </p>
| **SL** | **Topic** |
| --- | --- |
| 01 | [Application Source Code Download](#01) |
| 02 | [Azure DevOps Core Services](#02) |
| 03 | [What is Azure Pipeline?](#03) |
| 04 | [What is Azure Repository (Repos)?](#04) |
| 05 | [What is Powershell Scripting?](#05) |
| 06 | [How to Run Powershell Script in Azure Pipeline?](#06) |

### <a name="01">:diamond_shape_with_a_dot_inside: &nbsp;Application Source Code Download?</a> 
- Download the application source code by running this command: ````git clone https://github.com/Shadikul-Islam/ASP.NET-Core-Projects.git````.
- Now go to the folder by running this command: ````cd ASP.NET-Core-Projects/'Dockerize ASP.NET Core Application with MSSQL Server Database' ````.
- Now you can see the all of the file by running ````ls -a```` command.
- Most important thing is we need to know the place where we need to set the database credentials of this project. In the **Models** Folder we have a file named 
**MyDBContext**. We need to open that file. Run this command to open the vi editor: ````vi Models/MyDBContext.cs````. We can see **SqlConnection** under the **MyDBContext**
class. Just change this line like this: ````SqlConnection con = new SqlConnection("Data Source=db;Initial Catalog=dbbackup;Integrated Security=false; User Id=sa; Password=rootpa@sw0rdmysql");`````.
Here our Database Source will be ````db```` which we will declare on the **Dockerfile** later. 

