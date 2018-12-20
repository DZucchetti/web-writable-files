# web-writable-files

A [group led by Google and Mozilla](https://www.techrepublic.com/article/google-mozilla-working-on-letting-web-apps-edit-files-despite-warning-it-could-be-abused-in-terrible/) is working to make it easy to edit files using browser-based web apps but wants advice on how to guard against the "major" security and privacy risks.
An early draft propose an [API to allow web/browser based applications to access the file system](https://github.com/WICG/writable-files), for reading and writing files.
[The maintainer is seeking comments](https://discourse.wicg.io/t/writable-file-api/1433), especially related to use cases. 

## Offline Mobile Apps
Our company, [banana.ch](https://www.banana.ch/) has an ongoing research project  with a SUPSI, a local university, that is aimed at allowing Banana Accounting software to concurrently edit accounting files on mobile devices that are not connected to the internet. Banana is a spreadsheet powered accounting software, that means that it part a spreadsheet and part an accounting software. The  user interface and the way the files are modified are similar to Excel. Contrary to typical accounting software that use SQL databases, in Banana, all the accounting data is read from the file into memory and once the file is saved, the original file is competely overwritten. All data is available locally, so the user can work and do their accounting even if the device is not connected. The problem arise when more people work on the same accounting file at the same time. 
Like in Excel or Word, with the typical approach to read and write to file, when the user saves the work the previous version is overwritten. The aim of the research project is to integrate the different changes in the file. An advanced merge process, similar to the merge or programming source files, but that also works with structured accounting data. 

During the project we have defined a modern use case, that considers that users work on mobile devices, connected or not connected to internet. We explain it herafter, why we think that it may be interesting approch to consider for creating a new API to access files on the web browser. 

Mobile devices are becoming more powerful. The difference between Desktop Application and Phone Application is disappearing. Everything that a personal computer can do, can also be done with a phone app. At Banana.ch we have ported the whole QT based, accounting software, to Android and IOS. We are still adapting the interface, but basically all the accounting functionalities available on Desktop are there also on the phone or tablet. Delegating all major tasks to the mobile device makes a lot of sense and that why Hedge Computing is becoming relevant. The displacement of the computation from the cloud to the mobile device is also becoming  easier thanks to Webassembly, existing desktop application can be "easily" ported to the browser.

Fitting a software on a phone requires changes to the interface and to the workflow. On mobile devices you cannot rely on the fact that the connection is available and is of a good quality. Connections are improving, but there will always place where it goes down. People want to continue using their apps independently of the fact that the connections is available or not. 

This is the reason why new mobile apps are developed using a the approach called [Service Workers](https://developers.google.com/web/fundamentals/primers/service-workers/). Instead of having just an open and save approches, each software has also a separate componente, the Service Worker, that that is responsible of making the data available of the application and synchronizing with the cloud. The Service Worker takes care of reading the data from the cloud and caching locally. The application will continue to work even if there is no connection. The user can change the data. Once the connection resume, the Service Worker take care of synchronizing local and cloud data. The [Service Worker is also a W3C draft](https://www.w3.org/TR/service-workers-1/) that should be supported by the browser.

Editing a file locally is very similar to the Service Worker approach:
- Data (the file) is read from the cloud. 
- The file content is stored locally (usually on the RAM or in a temporary file). It is therefore possible to works with no connection.
- The App modify the content.
- When the work is saved, the file is put back on the cloud. 

This approach has a big limitation, the previous file is overwritten. Therefore if the file has been already been modified by another user, all the work he has done get lost.  
This is also a **security threat** that should be considered. The Apps that receive the right to access the file in read and writing mode, should be prevented to overwrite a file that has been changed. In a security saving approach, the writing access should be limited to same file version. **The API draft should be completed in order to prevent overwriting a file already modified**.

## Workflow for editing files
Prior to making available the read/write API in the browser we should ask if the traditional read/write file  approach is still the best one or we can find a workflow that is more suitable for the mobile world. The read/write file API assume that the file is local. On mobile device the file usually reside on the cloud (Dropbox) and is copied locally only for the purpose of modifying it, once done is copied back to the cloud. It would be better if the API interact directly with the service that make the file available the file locally and synch it back. While we are editing the file, another device, could have changed the file. When the device is back online and want to save it should notice that the file has already been changed and help handle the situation. 
The workflow for editing a file on a mobile device, described here, is different than the one of a typical Desktop application: 
- The application or the user decide what resource it should have access. 
  - The resource can be a file or a directory content. 
  - The resource could be a local saved one, a network saved one or a cloud saved. 
- In order to edit the resource a local copy is needed. 
  - The API should provide a function to create a local copy on the temporary local user space. 
  - The application will access the local copy.
- The API should allow read/write access to the local copy.
  - The application can read and write the file as necessary. 
  - This allows the Apps to save changes to the local device.
  - In case the App is closed and reopened (the mobile device has been shut down) the work can resume from where it was. 
- The API should make available a commit functions.
  - The local file is copied back to the original place.
  - In case the original file has been changed specific conflict resolution procedure should be started.
- The API should provide an API for conflict resolution.
  There can be two cases:
  - Delegating the conflict resolution to the APP (like in document database).
    - The app prior to writing a file should read the changed file. 
	- The write request should be made with the prior version number.
  - Taking care of the merge.
    - It should offer a cockpit to manage/automate changes. 
   - Security policy may be dependent to the system and other elements, like the app or the web site that is attempting to save the changes.
   - User should be able to decide if and when direct updating of the original file is allowed.
   - The system should offer the possibility to visualize differences between file versions. This will help the user decide how to handle conflict and be sure of what is happening to their files.
	
## API for File-Service-Worker
We thing that the traditional file read/write API is not the best solution. 
We invite browsers or operating system to implement a Service Worker that is specific for accessing files:
The File-Service-Worker should implement an API that allow to: 
- retrieve content from any place
- maintain a local copy
- save the file back 
- be notified of conflicts
- solve possible conflicts  

## Security considerations
Allowing direct access to files within the browser pose a series of security risks. Next to the one that have been highlighted in the proposal, we think that the write of a changed file, should be considered also as a security risk. When the app receive access to a file, it should not be prevented to overwrite it if the file has been changed. 

It is not the aim of this document to focus on the security, but it evident that the  File-Service-Worker API concept allows to handle security a lot better than the proposed API. 
- The API will interact with the operating system and with the file provider that make cloud files available locally. 
- Files changes are written first locally in a confined user space. Browser would only access files in the user space and not all the computer. There are less risks that a bug in a browser could affect the computer security. 
- Virus checks are simpler if the files are first saved to a local user space.
- The only application that is allowed to makes changes is the file provider:  
  - File providers will take care of making the file available and write it back.
  - File provider will decide what access policies they will implement based on their needs.
  - Operating system should provide a local file provider for accessing file that are local to the device. The access policies of the device should apply.
	- Access policies to Dropbox/ICloud/Github/WebDav or other services will be managed by the specific file provider. 
- Apps are prevented to overwrite files that have been changed by someone else. 

## Comments
We available for further information and welcome comments and feedback. 




