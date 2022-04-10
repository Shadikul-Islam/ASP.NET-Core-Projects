## <p align=left>Dockerize ASP.NET Core Application with MSSQL Server Database<br> <br> </p>
| **SL** | **Topic** |
| --- | --- |
| 01 | [Application Source Code Download](#01) |
| 02 | [](#02) |
| 03 | [](#03) |
| 04 | [](#04) |
| 05 | [](#05) |
| 06 | [](#06) |

### <a name="01">:diamond_shape_with_a_dot_inside: &nbsp;Application Source Code Download?</a> 
- Download the application source code by running this command: ````git clone https://github.com/Shadikul-Islam/ASP.NET-Core-Projects.git````.
- Now go to the folder by running this command: ````cd ASP.NET-Core-Projects/'Dockerize ASP.NET Core Application with MSSQL Server Database'````.
- Now you can see the all of the file by running ````ls -a```` command.
- Most important thing is we need to know the place where we need to set the database credentials of this project. In the **Models** folder we have a file named 
**MyDBContext**. We need to open that file. Run this command to open it: ````vi Models/MyDBContext.cs````. We can see **SqlConnection** under the **MyDBContext**
class. Just change this line like this: ````SqlConnection con = new SqlConnection("Data Source=db;Initial Catalog=dbbackup;Integrated Security=false; User Id=sa; Password=rootpa@sw0rdmysql");````.
Here our **Database Source** will be ````db```` which we will declare on the **docker-compose** later. **Initial Catalog** will be the database name. Here our database name is ````dbbackup````. **User Id** is the username which is ````sa```` and **Password** is the ````rootpa@sw0rdmysql```` in our case. You can set your own password which need to set it in the **docker-compose .env file** later.

