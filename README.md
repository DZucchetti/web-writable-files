# web-writable-files

An early draft propose an [API to allow web/browser based applications to access the file system](https://github.com/WICG/writable-files), for reading and writing files.
The maintainer is seeking comments, especially related to the security model. Giving browser applications the ability to directly access, both in read and write, the file system has the potential to make the browser experience less secure.

Browser based app are getting more powerful and this trend will become stronger with webassembly. It is therefore becoming a necessity for apps to have access to local files and directories, read and write content.

We have currently at [banana.ch](https://www.banana.ch/) an internal research project, together with a SUPSI, a local university, that is aimed at allowing concurrently accessing and modifying accounting files in off line mode. People using their mobile device should be able to do change to the accounting file even if they are not connected to the internet. When they connect again the changes should be automatically synchronized. In the meantime, someone else may have made changes, so there is a necessity to merge changes and resolve conflicts. 
The work has made us aware that the traditional, desktop way, to programmatically access files is not suited for mobile applications that change files. The same reasoning apply to browser based apps. Therefore, I will elaborate on that and comment the proposed API.

## offline web app
When people are working on mobile applications the connection is not always reliable. If you are editing a file and you lose the connection for the fact that the train is entering a tunnel, you should be able to continue working. Modern apps are encouraged to use a [Service Workers](https://developers.google.com/web/fundamentals/primers/service-workers/) to cache data locally. When the connection is lost, the user can continue working. When the connection resumes the changes are automatically synchronized.

The problem arises also on desktop applications that are constantly connected. A user is editing a text file from a local Dropbox directory. At the same time someone else is editing the same file. When the user is done editing and he push the save button. the file is completely overwritten. If the other user has already saved their work, all he has done get lost. 
This situation is more and more becoming common, not just for multi-user concurrent working, but also with the same user working on different devices. 

Allowing web apps to directly write file is therefore not a very suitable solution, for security reasons, but also for the fact that the "desktop-api" does not accommodate working on concurrent working offline. The API that allow webapps to access file locally should consider the new technology environment and be more similar to a "service-worker" than to a file system API as proposed in the draft. 

## Possible approach
The research project has made us aware of the problem, but we never attempted to imagine a new API to save files. The publication of the API is the occasion to get a step further by trying to image an alternative way to approach the problem. 

When editing documents or images locally it is wise for the software to create a temporary copy on the device. If the connection goes down or is not of good quality the application has continuous access to the data. Software should also provide a way to temporarily save the changes. If the app crash or it is closed, what have been done should be retrieved.  
The easiest way to achieve that is by creating a local copy of the original file and use this. Changes are saved directly to the file. When the user push the "save/commit" button the changes are copied back and the original file is overwritten by the new copy.  
Due to the fact that the original file may reside on the cloud, in the meantime someone else could have changed it. If the original content has been changed, the app, should be able to to know and takes appropiate steps. It should provide a way to merge the changes or else, in case that a merge cannot be performed, inform the user witch file version to keep. 

Each application that allows to work offline should provide this functionality. But a better solution, also for security reason, would be to have the operating system or the web browser act as a "file-service-worker". 
The File-Service-Worker should implementa and API that allow to: 
- retrieve content from any place
- be notified of changes 
- save the file back 
- be notified of conflict
- see differences between version
- resolve possible conflicts.  

The apps interact with a File-Service-Worker that makes available the file and take care to save the data back to the appropriate place and in case of concurrent changes delegate offer a chance to the app to solve the conflict or let the user decide what version of the file to keep. 

Instead of a simple API to access a local file, a complete service should be made available:  
- The File-Service-Worker should be provided by the browser or by the operating system.
- The webapp should not have direct access to the file.
- The same API should allow to access any kind of resource (local file, network file, Dropbox file) 
- When the App request access to a file, the API create a local copy in the user space. 
- The app can have full access to the file.
- The file also serves as a place to store temporary changes.
- It should be possible to specify an updating policy. The system will save the file back to the original position, based on the specific choice:
  - when the user commits the changes.
  - when a continuous automatic synchronization has been specified. 
- Like in a modern document database, When saving a file back the hash of the orginal file should be provided.
- The system will only overwrite the original file with the new file if the hash of the original file  version is the same. 
- It is up to the app to solve the conflict. 
  - The app read the new version of the file.  
  - It then can tries to do an automatic merge or it ask the user what is appropriate to do. 
- The system that takes care of making available the file locally and save back to the original position, should also be responsible to implement the security policies. 
   - It should offer a cockpit to manage/automate changes. 
   - Security policy may be dependent to the system and other elements, like the app or the web site that is attempting to save the changes.
   - User should be able to decide if and when direct updating of the original file is allowed.
   - The system should offer the possibility to visualize differences between file versions. This will help the user decide how to handle conflict and be sure of what is happening to their files.
   
   


