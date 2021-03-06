LCMPROJECT
==========
 I developed a Grails application for a company called LCM(an imaginary company) that specializes in repairing electrical equipment for large companies and factories. LCM repairs large scale electrical equipment by monitoring their heat and vibration outputs. The company produces pdf reports that contain information about specific machines and their problems complete with detailed photos.
Domain:
I started by created a new Grails project and named it lcm_mithuna. First I created three domain- Company.groovy, Location.groovy, Report.groovy, with field names

	Company.groovy
		nameOfCompany
	Location.groovy 
		nameOfLocation
		address 
		companyName(belongsTo) 
	Report.groovy 
		nameOfReport
		locationName (belongsTo) 
		publication Date 
		field (grad Ext)

Location.groovy has a companyName which belongs to Company.groovy and Report.groovy has a field locationName which belongs to Location.groovy.
 belongsTo defines a "belongs to" relationship where the class specified by belongsTo assumes ownership of the relationship [1]. 
I also set constraints for each field in Domain. 
static constraints = {
		nameOfReport(blank:false)
		field(blank:false)
    }

Controller:
I created three controllers corresponding to three Domains- CompanyController.groovy, LocationController.groovy, ReportController.groovy.
I used def scaffold = true in all the controllers. 
	As a part of graduate Extension, I was asked to create another controller that controls the routing between two actions. I named this controller RouterController.groovy. I was not able to do scaffolding for this Controller as it does not have a domain. But I created a function “upload”, which is linked with the view of this controller.

View:
	When I did ‘scaffolding = true’ in the controllers, the magic IDE eclipse-grails automatically created corresponding views for me. 
	Since I didn’t scaffold RouterController, I created a view for it named upload.gsp. GSP is Groovy Server Page. This upload.gsp has a form for uploading the file. I used 
<g:uploadForm action="upload" method="post">
This form has a field for selecting “Location” from the list of saved Locations and a field to upload a file. This form action will invoke our controller and inside the controller, I first retrieve the file using command
request.getFile('uploadFile')
And then checks whether file is empty. If empty the view page upload will be displayed again with a flash message. If not that file will be saved to a directory using command
f.transferTo(new File('file path’));
I also checked if the file extension is ‘pdf’. If not I displayed a flash message and render to upload view. Else I redirect to list of ReportController.
index.gsp:
	I customized the index.gsp page by removing the side bar containing Application Status and Installed Plugins which is no significance in my project. I also included marketing information about the company LCM. I also moved the controller information from the bottom of the page to the left of the page for more visibility and easy access. 
	I used CSS3 to style my index page.

404error.gsp:

	I created an error page which will load an image (cupid.png) when a 404 error occurs. 
	I then changed error section of the UrlMappings.groovy page to 

		"404"(view:'/404error')
	UrlMapping is used to map different views.
Main.gsp:
	
Inside main.gsp, I removed the grails logo from banner and replaced it with LCM logo.
Main.css
	I made many changes to the main.css file to make the page look beautiful. I changed the color and the banner and the body.  I moved the controller tab from the bottom of the page to the left side and aligned it beautifully using css.

I added spring security plugin to the grails application.

compile ':spring-security-core:1.2.7.3'

This will create Domains – User, Role and UserRole and also Controller login and logout.
I also added 
compile ":famfamfam:1.0.1"
		compile ":mail:1.0.1"
		compile ":jquery:1.8.3"
		compile ":jquery-ui:1.8.24"
So now I have 
Domain: 
	Company.groovy
		nameOfCompany
	Location.groovy
		nameOfLocation
		address
		companyName(belongsTo)
	Report.groovy
		nameOfReport
		locationName (belongsTo)
		publication Date	
	User
		username
		password
		companyNameforUser
	Role
		authority
	UserRole
		user
		role
Controller:
lcm_a20295703
	CompanyController.groovy,
	LocationController.groovy,
	ReportController.groovy.

default
	LoginController.groovy
	LogoutController.groovy


I have a file named BootStrap.groovy inside the conf folder, which I used to create domain objects for Domains –Company, User and Role and connected them using UserRole.create
First I created two companies- LCM and testCompany
def LCM =new Company(nameOfCompany:'LCM').save(flush:true)
def testCompany = new Company(nameOfCompany:'testCompany').save(flush:true)
		
Then I created two  Roles- ROLE_ADMIN and ROLE_USER
def ROLE_ADMIN = new Role(authority:'ROLE_ADMIN').save(flush:true)
def ROLE_USER = new Role(authority:'ROLE_USER').save(flush:true)
				
Finally I created two  User- admin and sampleUser
def admin = new User(username:'admin',password:'password',companyNameforUser:'LCM', enabled:true)
admin.save(flush:true)
def sampleUser = new User(username:'sampleUser',password:'password', companyNameforUser:'testCompany', enabled:true)
sampleUser.save(flush:true)
		
Now link the role and user together 
‘admin’ role for role ‘admin_role’, user ‘admin ‘ and company ‘lcm’
‘user’ role for role ‘user_role’, ‘user’ sampleUser and company ‘testCompany’

UserRole.create admin, ROLE_ADMIN, true
		
UserRole.create sampleUser, ROLE_USER, true
		
Now I arranged my scaffolded view. I created a new layout named subpage which has a slightly different layout than the main layout. 
Then I deleted my controllers- Company, Location and Report. Then I generated two new controllers-Company and Location.   I added @Secured(['ROLE_ADMIN']) to all the actions so that only users with admin privileges  can access Company and Location views.
Next I added links to login and logout to the main and subPage layout, which included functionalities like
	Once a user is logged in, a logout link is available 
	If no user is logged in, a login link is available 
For that I used 
<sec:ifLoggedIn>	and	<sec:ifNotLoggedIn>
Which helps in creating a link for login and logout and also redirects to login page when login link is clicked and to home page when logged out.
USER CREDENTIALS:
Admin
Username	:	admin
Password	:	********
Sample User
Username	:	sampleUser
Password	:	********


Now I created a new controller (not generate) to include report functionality and named it ReportController. I included 
	Action index, which redirects to action listReport.
	Action listReport, which lists all the reports. 
To display the reports posted for the company of the current user I first obtained the current username and created an object of domain User using the current username. Then I created an object of domain Company using the User object. With this company object I created an object of domain Location. Then I used a closure to iterate through the Location object to get each entry and created an object of domain Report using each location object and saved it to an array reports

def currentUser = springSecurityService.getPrincipal().getUsername()		
def companyByUser= User.findAllByUsername(currentUser)	
def company = Company.findAllByNameOfCompany(companyByUser[0].toString()) 
def locations = Location.findAllByCompanyName(company[0]) 
def reports=[]
locations.each {oneUserLocation->
	println oneUserLocation
		reports += Report.findAllByLocationName(oneUserLocation) 

		}
For calculating and checking whether the records were published with-in last one year, I first retrieved the records publication date, calculated current date  and converted  publication date and current date to time in millisecond.
def pubTime=pubDate.time
I calculated the difference of these two times in millisecond and converted millisecond time to number of days by dividing the difference value by number 86400000 (1day = 24hrs*60min *60seconds *1000ms). If number of days hence obtained is less than 365 add to an array ‘reportNameArray’ for displaying.
	Action upload, redirects to save action
	Action save, was created for saving the. First I used 
request.getFile('uploadFile') 
def filename = f.originalFilename was used to get the original name of file submitted. 
I used the split method to get the Content type of the file uploaded and concatenated it to the nameOfReport.
Then f.transferTo(filepath) was used to transfer the file to a local folder.
reportInstance.save(flush: true)  helps in saving the instance. Now this action redirects to another action showReport.
	Action showReport which displays the details of the new Report created with the help of showReport.gsp and the id passed to it from the save() action. If no id is passed then it will be redirect to the listReport() action. 
	Action deleteReport, for deleting the Report using the id passed. If no id is passed then it will be redirect to the listReport() action. 
	Action error which displays an error message and provide link to return to the listReport view.
	Action to download files uploaded using the id passed. I retrieved the nameOfReport(which was the name of file when saved) using 
def filename =down.nameOfReport. 
With this I checked whether the file uploaded folder have the file requested and if so I downloaded the file using
response.outputStream << fileForDownload.bytes

In order to display the reports in chronological order, I used a sort() function inside my listReport.gsp. The locationInstance from listReport() action is a map of <key,value>. I first did a for-each of the key and then a for-each of the values for each key to obtain the result. In doing so I used the sort() to sort it in the newest first order.

FUTURE IMPLEMENTATIONS:
I would like to add an edit page for the report controller.



Reference:
[1] http://grails.org/doc/latest/
