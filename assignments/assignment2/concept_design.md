# Task 3 - Concept Design

## Concepts

**concept** Profile \[TextSettings\]

**purpose** collect basic authentication and user info

**principle** A user can create an account and sign in with a username/password. 

**state**  
a set of Users with:  
    a username String  
    a password String  
    a textPreferences TextSettings

**actions**  
createAccount(username: String, password: String): (user: User)  
&emsp;**requires** username is not an existing username, and password is sufficiently secure  
&emsp;**effects** creates a new User with username and password

deleteAccount(user: User)  
&emsp;**requires** user exists  
&emsp;**effects** removes user from the set of Users

authenticate(username: String, password: String): (user: User)  
&emsp;**requires** username and password both correspond to the same existing User  
&emsp;**effects** returns the user associated with the username and password

<hr>

**concept** Library \[User, TextSettings, FocusStats\]

**purpose**  collect and store documents that users upload

**principle** An existing user can upload and associate documents with their account, to be accessed later 

**state**  
a set of Libraries with:  
    a user User
    a set of documents  

a set of Documents with:  
    a name String  
    a file File  
    a settings TextSettings  
    a stats FocusStats
    a set of Annotations

**actions**  
createLibrary(user: User): (library: Library)  
&emsp;**requires** user is not already associated with a library  
&emsp;**effects** creates a new library with user and an empty set of documents

addDocument(library: Library, document: Document)  
&emsp;**requires** library and document both exist, and document is not already in Library  
&emsp;**effects** adds document to library's set of documents

removeDocument(library: Library, document: Document)    
&emsp;**requires** library exists and document is in library  
&emsp;**effects** removes document from the set of documents and from library  

createDocument(name: String, file: File, library: Library): (document: Document)  
&emsp;**requires** a document with name and file does not already exist in the given library
&emsp;**effects** creates a new Document with name, file, default TextSettings, and default FocusStats, and empty set of Annotations 

openDocument(user: User, document: Document): (document: Document)  
&emsp;**requires** user is in a library with document  

closeDocument(user: User, document: Document): (document: Document)  
&emsp;**requires** user is in a library with document

**notes** assume that File is a natively existing data type in python. Assume that "default" TextSettings and FocusStats will be created by implementing system. 

<hr>

**concept** FocusStats

**purpose** track and aggregate users' reading statistics

**principle** When users read a document, the system automatically tracks the times and lengths of their reading sessions. Users can see statistics on their reading behavior

**state**  
a set of FocusSessions with:  
&emsp;a user User  
&emsp;a document Document  
&emsp;a startTime Datetime  
&emsp;an endTime Datetime | None

a set of FocusStats with:  
&emsp; a set of FocusSessions  
&emsp; a user User

**actions**  
initUser(user: User): (focusStats: FocusStats)  
&emsp;**requires** user exists and does not have a focusStats  
&emsp;**effects** creates a focusStats with an empty set of FocusSessions and user  

startSession(user: User, document: Document, library: Library): (focusSession: FocusSession)  
&emsp;**requires** user has document in their library and user has a focusStats
&emsp;**effects** creates a new focusSession with user, document, startTime = current time, and None endTime

endSession(focusSession: FocusSession): (focusSession: FocusSession)  
&emsp;**requires** focusSession exists and has endTime of None  
&emsp;**effects** sets focusSession endTime to current time, adds focusSession to the user's FocusStats, returns focusSession

removeSession(focusSession: FocusSession)  
&emsp;**effects** removes focusSession from the set of FocusSessions and from the user's FocusStats' set of FocusSessions

viewStats(user: User): (focusStats: FocusStats)  
&emsp;**requires** user is associated with a focusStats object    

<hr>

**concept** Annotations \[Document\]

**purpose** allow users to make notes inside documents

**principle** When users read a document, they can make and view highlighting or text annotations in the document

**state**  

a set of Annotations with:  
&emsp;a document Document  
&emsp;a color Color  
&emsp;a content String  
&emsp;a location Location  


**actions**  
createAnnotation(document: Document, color: Color, content: String, location: Location): (annotation: Annotation)  
&emsp;**requires** document exists, and location is a valid location in the document  
&emsp;**effects** creates and adds annotation with document, color, content, and location to the set of Annotations

deleteAnnotation(annotation: Annotation)  
&emsp;**requires** annotation exists  
&emsp;**effects** removes annotation from the set of Annoations
 
**notes** Location is some representation of location in a file, not specified in the concept but assumed to exist natively in python/file representations

<hr>

**concept** TextSettings \[User, Document\]

**purpose** store text display settings for documents

**principle** When setting up an account, users can create text display preferences for their account. When reading a document, users can also set text settings for individual documents, which will change how the document is displayed 

**state**  
a set of TextSettings with:  
&emsp;a font Font
&emsp;a fontSize number  
&emsp;a lineHeight number  
&emsp;a bionicEnabled bool  
&emsp;a locale User | Document

**actions**  
createSettings(font: Font, fontSize: Number, lineHeight: Number, bionicEnabled: bool, user: User): (settings: TextSettings)  
&emsp;**requires** there is not already a TextSettings with user  
&emsp;**effects** creates a TextSettings with fontSize, lineHeight, bionicEnabled, and locale=user

editSettings(textSettings: TextSettings, font: Font, fontSize: Number, lineHeight: Number, bionicEnabled: bool): (settings: TextSettings)  
&emsp;**requires** textSettings exists  
&emsp;**effects** changes textSettings to have fontSize, lineHeight, and Number

## Syncs

**sync** createUser  
**when** Profile.createAccount (): (user)  
**then**  
&emsp; Library.createLibrary(user)  
&emsp; FocusStats.initUser(user)  
&emsp; TextSettings.createSettings(font: Arial, fontSize: 16, lineHeight: 1.5, bionicEnabled: False, user)  

**sync** viewDocument  
**when** Library.openDocument (user, document): (document)  
**where** document in library  
**then** FocusStats.startSession(user, document, library)

**sync** stopViewingDocument  
**when** Library.closeDocument (user, document): (document)  
**where** focusSession is a FocusSession with no endTime corresponding to user and document  
**then** FocusStats.endSession(focusSession)

## Notes

Aside from initialization, the concepts are fairly independent from each other. Users create a Profile with usernames and passwords, and can set a default TextSettings preferences. Users have their documents in their Library. Document display is controlled by TextSettings, which control elements like font, font size, etc. Users also have analytics (FocusStats) that track their reading sessions.  