---
title: "Project Setup"
date: "2014-09-09"
---

### Download the latest brainCloud SDK

TODO INSERT LINK HERE

 

### Include SDK in Xcode Project

- Extract the SDK zip file "brainCloudClient\_x.x.x.zip" to a reasonable location within your project folder
- Right click on the project and select Add Files To "Your Project Name"… (bunch of pictures)

![](images/AddFilesToProject.jpg)

 

- Select the extracted zip folder and ensure that **Create groups for any added folders** is checked and **Add to targets** is checked for your appropriate target

 

![](images/AddFilesToProject2.jpg)

 

 

This will create two folders:

- include — this contains all of the headers for the brainCloud SDK
- libs — this contains all of the libs files for the brainCloud SDK

 

### Add the brainCloud Helper class

Included on the FTP site is a "brainCloudHelper\_x.x.x.zip" file which includes helper class files to get you started. We recommend using these interfaces to speak with brainCloud. Benefits are simplified error handling and an ability to pass in JSON objects directly.

- Extract the SDK zip file "brainCloudHelper\_x.x.x.zip" to a reasonable location within your project folder
- Right click on the project and select Add Files To "Your Project Name"…
- Select the extracted zip folder and ensure that Create groups for any added folders is checked and Add to targets is checked for your appropriate target

Note that this class depends on the JSON library described below so make sure to add it to your project as well!

 

### Include a JSON parsing library

brainCloud uses the jsoncpp library, version 0.6.0-rc2. Follow the steps below to include this library:

- Download the library from here [http://sourceforge.net/projects/jsoncpp/files/jsoncpp/0.6.0-rc2/](http://sourceforge.net/projects/jsoncpp/files/jsoncpp/0.6.0-rc2/)
- Extract the JSON library to a reasonable location within your project folder
- Right click on the project and select Add Files To "Your Project Name"…
- Select the "src/lib\_json" folder and ensure that Create groups for any added folders is checked and Add to targets is checked for your appropriate target
- Right click on the project and select Add Files To "Your Project Name"…
- Select the "include" folder and ensure that Create groups for any added folders is checked and Add to targets is checked for your appropriate target
