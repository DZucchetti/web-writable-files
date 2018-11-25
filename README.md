# web-writable-files

An early draft propose an [API to allow web/browser based applications to access the file system](https://github.com/WICG/writable-files), for reading and writing files.
The maintainer is seeking comments, especially related to the security model. Giving browser  applications the ability to directly access, both in read and write, the file system has the potential to make the browser experience less secure.

Browser based app are getting more powerful and this trend will become more strong with webassembly. It is therefore becoming a necessity for apps to have access to local files and directories, read and write content.

We have currently at [banana.ch](https://www.banana.ch/) an internal research project, together with a SUPSI, a local university, that is aimed at allowing concurrently accessing and modifying accounting files in off line mode. People using their mobile device should be able to do change a the accounting file even if they are not connected to the internet. When they connect again the changes should be automatically synchronized. In the meantime, someone else may have made changes, so there is a necessity to merge changes and resolve conflicts. 
The work has made us aware that the traditional, desktop way, to programmatically access files is not suited for mobile applications that change files. The same reasoning apply to browser based apps. Therefore I will elaborate on that and comment the proposed API.

## offline web app
When people are working on mobile applications the connection is not always reliable. If you are editing a file and you lose the connection for the fact that the train is entering a tunnel, you should be able to continue working. Modern apps are encouraged to use a [Service Workers](https://developers.google.com/web/fundamentals/primers/service-workers/) to cache data locally. When the connection is lost, the user can continue working. When the connection resumes the changes are automatically synchronized.

The problem arise also on desktop applications that are constantly connected. A user is editing a text file from a local Dropbox directory. At the same time some one else is editing the same file. When the user is done editing and he push the save button. the  file is completely overwritten. If the other user has already saved their work, all he has done get lost. 
This situation is more and more becoming common, not just for multi-user concurrent working, but also with the same user working on different devices. 

Allowing web apps to directly write file is therefore not a very suitable solution, for security reasons, but also for the fact that the "desktop-api" does not accomodate working on concurrent working offline. The API that allow webapps to access file locally should consider the new technology environment and be more similar to a "service-worker" than to a file system API as proposed in the draft. 

## Possible approach
The research project has made us aware of the problem, but we never attemped to imagine a new API to save file. The publication of the API has been the occasion to get a step further and imaging a possible alternative way to see and solve the problem. 



