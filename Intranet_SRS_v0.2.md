# Software Requirements Specification (SRS)
# Company Intranet / ERP Lite
Version: 0.2

## 1. Purpose
Develop an intranet for an IT consulting/software company to manage:
- Employees 
- Customers
- Projects
- Timesheets
- Travel expenses
- Treasury (Payment Systems)
- Accounting (Chart of Accounts + General Ledger)
- Reporting
- Dashboards

## 2. Technology Stack
Frontend:
- Angular
- TypeScript
- HTML/CSS
- Angular Material
- OpenID Connect with OAuth 2.0 

Backend:
- .NET 10 Web API
- Entity Framework Core
- Clean Architecture
- Swagger/OpenAPI

Database:
- SQL Server
USe the connection string : Data Source=.\\EXPRESS2025;Initial Catalog=Intra2026;Integrated Security=True;Encrypt=True;TrustServerCertificate=True;

## Common application API

/api/app-lookup/Sex			return the option for the combobox 


## 3. Application authentication and authorization
The use of the intranet is reserved atuthenticated user.
The only anonymous visible page is the login page.
The authentication and authorization is based on the OpenID Connect with OAuth 2.0 standard specification.
The authentication is possible using local user or Microssoft Entra ID user

### Local user management
The data of local user are stored in the AppUser table of the database.The field UserProvider assume value 'LOCAL' self managed user, 'MSENTRA' for Microsoft Entra User.
The role associated to the user is in the AppUserClaim table. In the AppUserClaim table the field ClaimUri is always "http://schemas.microsoft.com/ws/2008/06/identity/claims/role". The name of the role is saved in ClaimValue field.

Two role are possible for a user:
OWNER
EMPLOYEE

Any user can be associated to one or more role.

### Authentication and authorization
The authentication can be made by username/password for local user.
PasswordHash and PasswordSalt are in the AppUser table.

If someone connect using Microsft Entra ID the first logon time the system add a new record to the AppUser table with UserPRoviderType is MSENTRA, PasswordHash and PasswordSalt are blank, AppUserUId is the objectid of MSENTRA, username is the MSENTRA username

### Api reference

/api/auth/login			POST 	the login data for local user
/api/auth/me			GET		return the current logged user
/api/app-user			POST 	create a new user
						GET	 	retrieve the list of user
						PUT  	update user information. If the password is not blank, and the password and confirm passwrod field contain the same value change the password of the user
						
/api/app-user/<id:int>	GET		Rturn the information for user with AppUserId = id
						
### Interface
Login page
'''
+-------------------------------------------------------------+
|						[Company Logo]                        |     
|                 ZZ Soft S.a.s. di S. Carri & C.			  |
+-------------------------------------------------------------+
|                                                             |
|		User name: [___________]                              |
|		User name: [___________]                              |
|								                              |
|		               [Login]                                |
|								                              |
|								                              |
|                           OR                                |
|                                                             |
|            [Access with Microsoft 365 Entra]                |
|                                                             |
+-------------------------------------------------------------+
'''
AppUser list page
'''
+------------------------------------------------------------------------------+
| User name | Name	| Emai | Mobile | Locked | Need Password Change | Disabled |                                                             |
+------------------------------------------------------------------------------+
'''
Rule
Name is the CONCAT(Name , ',', Name2)

AppUser edit page

'''
+--------------------------------------------------------------------+
|	User name														 |
|	[_________]				      									 |
|                                                                    |
|   Name						|		Name2                        |
|	[_________]                 |       [_________]                  |
|                                                                    |
|   Mobile phone			    |		Email                        |
|	[_________]                 |       [_________]                  |
|                                                                    |
|	Password        			| Confirm password  				 | 
|	[_________]     			|	[_________]						 |
|								|									 |	
|  Disabled [_]                 |                                    |
|																	 |
| Info ------------------------------------------------------------- |
|																	 |
| Disabled by:	Disabled At:     									 |
| Locked      LockedAt												 |
|	Record created At         Record created by						 |
|	Record created At		  Record created by						 |
|																	 |
+--------------------------------------------------------------------+
'''

Rule:
In the edit page is user have role EMPLOYEE change only the Name,Name2,Password. See all othe field in read only
The OWNER can edit UserName,Password,Confirm Password,Name,Name2,Email,MobilePhone,Disabled,Locked
Visualize as ReadOnly for every role all other field
When the Disabled is set to 1 update automatically the DisabledAt to current datetime and DisabledBy to user who execute the action
When Locked is set to 1 (true) update automatically LockedAt to current datetime, if set to 0 (false) remove the LockedAt datetime

## 4. Customer

 The customer information is stored in the Customer table.

### Customer list page
'''
+---------------------------------------------------------------------------------+
| Id | Full Name	| Vat Country | VAT Number | Address | Cap | city | Province, Country |
+---------------------------------------------------------------------------------+
'''
Rule
Full Name is the CONCAT(Name , ' , ', Name2) if Sex IN ('M','F'). Full Name is Name if Sex = 'G'

### Customer edit page
'''
+--------------------------------------------------------------------+
| X				 											[SAVE]   |
| FULL name															 |
|																	 |
+--------------------------------------------------------------------+
|																	 |
|																	 |
| ------------------------------------------------------------------ |
|																	 |
|	Record created At         Record created by						 |
|	Record created At		  Record created by						 |
|	Record deleted: [ ]	Record deleted at Record deleted by			 |
+--------------------------------------------------------------------+
''' 
 
 Rule:
 The sex is a combo and option are available from the table EnumLookup with the query "SELECT code,description FROM EnumLookup WHERE name = 'Sex' and disable = 0"
 If name2 is empty force sex = 'G' 
 Full Name is the CONCAT(Name , ' , ', Name2) if Nome2 not is blank. Full Name is Name if Name2 is blank
