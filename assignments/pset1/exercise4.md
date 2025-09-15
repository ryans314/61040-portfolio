# Exercise 4 - Defining Familiar Concepts

## URL Shortener

**concept** URLShortener

**purpose** shorten URLs to be easier to type/understand 

**principle** a user creates a shortened URL by providing a base URL to link to, and optionally a suffix to be used for the shortened URL 

**state**  
a set of ShortURLs with  
&emsp; a suffix String  
&emsp; a redirect String  

**actions**  
create(redirect: String, suffix: String | None): ShortURL  
&emsp;**requires** redirect must be a valid URL and there must not be another ShortURL with suffix  
&emsp;**effects** creates a ShortURL that redirects to redirect and has the URL suffix suffix, or a random unique 7-character suffix if none is provided

**Notes:**  
- 7-characters is what bit.ly uses, and provides ~3.5 trillion options for random URL suffixes
- The full URL should be something like "service.name/suffix," but the concept is independent of the service name, and thus does not specify that. 


## Billable Hours Tracking

**concept** HoursTracker \[Employee\]

**purpose** track employees' billable hours

**principle** after adding projects, employees mark the starts and ends of their work sessions, tracking their hours on the project.

**state**  
a set of Projects with  
&emsp;a set of Sessions  
&emsp;a name String

a set of Sessions with  
&emsp;an employee Employee  
&emsp;a start Datetime  
&emsp;an end Datetime | None
  
**actions**  
createProject(name: String): (project: Project)    
&emsp;**requires** no other Projects with name exist  
&emsp;**effects** creates a Project associated with name and no sessions

startSession(employee: Employee, projectName: String): (session Session)  
&emsp;**requires** projectName is the name of an existing Project, and there is not an ongoing session for the same employee and project  
&emsp;**effects** creates a Session for the employee working on the project, with a start time of the current Datetime and None for an end time

endSession(employee: Employee, projectName: String, endTime: Datetime | None): (session Session)  
&emsp;**requires** projectName is the name of an existing Project, there is an ongoing session for the same employee and project, and if an endTime is provided, it must be greated than the start Datetime and before the current Datetime  
&emsp;**effects** modifies session to set the end Datetime to the endTime, or the current Datetime if no endTime is provided

**notes:**  
- Assume that the Employee concept is implemented elsewhere, and that it functions as a unique identifier for users
- endSession accounts for employees forgetting to end their sessions, since they can put in an endTime. It also prevents employees from setting the endTime in advance, since endTime must be before the current time. 
- Employees *can* have multiple sessions at once, just not multiple on the same project. This is to allow the possibility for tasks applicable to two projects to count for both projects at the same time.

## Time-Based One-Time Password (TOTP)

**concept** TimeBasedOneTimePassword (TOTP) \[User\]

**purpose** allow users to authenticate with automatically changing time-based tokens

**principle** users can authenticate 3rd party apps with a time-based password, which changes every 30-60 seconds, or every time it is used

**state**  
a set of users with  
&emsp; a username String  
&emsp; a password TOTP

a set of TOTPs with  
&emsp; a user User  
&emsp; a token String  

**actions**  
regenerate(user: User): (totp: TOTP)  
&emsp;**requires** user exists  
&emsp;**effects** creates a new totp associated with user and a random cryptographically generated token String, deletes the users old password, and sets the user's password to totp

authenticate(username: String, token: String): (totp: TOTP)  
&emsp;**requires** username correlates with a user, and token correlates to the user's password's token.  
&emsp;**effects** creates a new totp associated with the user and a random cryptographically generated token String, deletes the users old password, and sets the user's password to totp

**notes:**  
- The TOTP has a number of security improvements:
    - Prevents keyloggers from obtaining passwords, as well as password leaks, since passwords are regenerated every time the password is used (and every minute).
    - Prevents social engineering attacks (like writing password on a sticky note, or having the user tell you their password), since the user does not immediately know the password  
- The TOTP also still leaves some attacks open:
    - Phishing attacks will still work, since authentication is only called by the malicious actors, not the user themselves
    - If a device is compromised and attackers can access the auth app, then they will still be able to use the tokens and gain access