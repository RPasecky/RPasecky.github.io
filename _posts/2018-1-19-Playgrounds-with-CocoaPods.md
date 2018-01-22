---
layout: post
title: Playgrounds + CocoaPods (FireStore Edition)
markdown: redcarpet
---

update 8
<img style="float: right;" src="..images/firestore.png">
Playgrounds are an incredible tool for experimenting and and iterating on ideas in Swift. 

In the past I've primarily used them to build self-contained libraries ((See here)[https://github.com/dhmspector/ZeitSatTrack/blob/master/ZeitSatTrackLib/CelestialBody.swift]. However, when it comes to using them on anything that requires dependencies, the path to success can get a bit muddled. 

Recently, however, I've been breaking ground on a new app for iOS using a firebase as a backend. After compiling one too many times while tinkering with queries, I realized that this might be a great opportunity to bring playgrounds into my project. The results we're pretty neat! The path to integrating Playgrounds into your project is surprisingly simple once you've done it once. Though not exactly fun to figure out for yourself. Hopefully this helps out anyone interested in using this powerful tool in your development. 

*Note: This tutorial is much more useful if you have a FireStore account of your own setup before starting. You can learn how to do that here.*


BEFORE 					 			    AFTER
















At a birds Eye view it takes about 4, maybe 5, steps to get things up and running

- You'll create a playground
- You'll create a new target in your project
- You'll tell CocoaPods about the new target
- You'll tell files in your project about the new target (Optional)
- You'll tell your playground about the new target


## Step 1. Create a Playground

- Open **Xcode**
- Go to File &rarr; New &rarr; Playground. 
- Name the playground whatever you want (e.g. 'CoolAppPlayground') and save it in the root directory of your App.
- Finally Drag it into your open XCode Project next to your .xcodeproject file

Once you have a playground in your project you should be able to run interactive code from within your typical project window. Try it out to make sure you didn't run into any unexpected obstacles before moving forward. 

*Note: Xcode 9 removed the ability to create a new playground within your project, this method it still easy enough and gives the same effect.*

## Step 2. Create a new Target.

Next up, you need to create a new Target which will allow your playground to mirror the abilities standard target.  

Do this by opening up your desired project and clicking (**Xcode** &rarr; **File** &rarr; **New** &rarr; **Target** &rarr; **Scroll** to the bottom &rarr; select **Cocoa Touch Framework**

Once you do this you will be brought to a screen prompts you for a bit more information. Here, you'll just need to:
- Create a name for your framework (i.e. 'CoolAppPlaygroundFramework') 
- Select an organization name for your framework
- Make sure to select your default target under embed in Application

One final task: 

- Select your project file to access the general settings for the project.
- Switch targets on the top left corner to the target you just created. 
- Select a deployment target for your new Target to match that of the current target.

Great, step two done. Now it's time to tell Cocoapods about what you did. 





## Step 3. Edit your Podfile

The target you just created is going to provide a way for Playgrounds to import all of the installed Pods in your project with just one line. Before that can happen however, you are going to need to edit your Podfile and reinstall your pods. 

Let's get started by editing the example below: 

```ruby
platform :ios, '11.0'

target 'CoolApp' do
  use_frameworks!

  # Pods for CoolApp
  pod 'Firebase/Core'
  pod 'Firebase/Firestore'
end
```

Copy and paste the text from target &rarr; end, and replace the existing target with the name of the Target you just created. Once that is done, you will end up with a  as shown below

```ruby
	platform :ios, '11.0'

	target 'CoolApp' do
		  use_frameworks!

	  pod 'Firebase/Core'
	  pod 'Firebase/Firestore'
	end

	target 'CoolAppPlaygroundSupport' do
	  use_frameworks!

	  pod 'Firebase/Core'
	  pod 'Firebase/Firestore'
	end
```
Finally you're going to want to close your project and run 'pod install'. Installation should complete error free. If you do run into problems, the most likely cause is a discrepancy between Deployment targets between the new target ('CoolAppPlaygroundSupport') and the standard target ('CoolApp') for your app.





## Step 4.  Setting up the Playground.

With your pods installed, open up your project workspace. Your final step is to build against the target you created: 

- On the top left side of the screen, ensure that the new target you created is selected. 
- Select any iOS device simulater to build with. (I stuck with iPhone 6)
- If you are using firebase, you NEED to make sure that your Google-plist file is added to your playground target, as shown in the image below.
- Press Command + B to build the new target. 

The build should be successful, if not check out the debugging section below for some troubleshooting help.

Finally, the moment you've been waiting for. Select your playground and import the Target you've created as shown below: 

```swift
import CoolAppPlaygroundSupport
import PlaygroundSupport 
import FirebaseFirestore

PlaygroundPage.current.needsIndefiniteExecution = true // Important when doing anything async

FirebaseApp.Init()

let ref = FireStore.firestore()
```

If all went well, you should see some comforting confirmations that the code ran successfully. 

*Note: Again, checkout the troubleshooting section below if you run into issues.*

*Note: I've used Firebase in this case, but I believe this process could be followed with really any CocoaPod that can successfully run on a device simulator.*


I've gone ahead and provided two funtions that will write data and decode it once you're all set up. You should be all set to play around, have fun and enjoy a much easier way of experimenting with database design.  

```swift	
//Keep in mind that these need to be run continuously, comment out a function call to keep it from running	
var allowUpload = true
var allowDownload = true

postNotes() //You should only run this once unless you plan to add more notes
getNotes() 

func postNotes() {

	guard allowUpload == true else {
		print("Blocked Upload request -- Request previously run")
		return
	} // prevents continuously overwriting data


	let pastDate = Date().addingTimeInterval(-100000)
	let futureDate = Date().addingTimeInterval(100000) 
	let currentDate  = Date()

	let note1 = ["Title": "Falcon 9 Launch Friday",
				 "Text" : "Remember that the launch is on friday, Leave by 4pm to get there on time"
				 "Date" : pastDate]


	let note2 = ["Title": "Pizza Party Saturday",
				 "Text" : "Remember to eat some pizza"
				 "Date" : currentDate]


	let note2 = ["Title": "Sleep on Sunday",
				 "Text" : "Remember to Sleep a lot"
				 "Date" : futureDate]

	let notes = [note1, note2, note3]

	for note in notes {
		ref.upload(notes){ completion: {finished in allowUpload = false }}
	}
}

//Should print out the title of all the notes that have a date in the future
func getNotes() {

	guard allowDownload == true else {    // prevents continuously downloading data
		print("Blocked Download request -- Request Previously Run")
		return
	} 
	ref.getDocument .where("Date" > now) {
		for document in queryResults.getData() {

		} completion: { (finished) in allowDownload = false }
	}
}
```



Debugging Tools: 

Playgrounds are trying to do a lot at once and can get hung up in some areas that normal, compiled code tested on a device would not. I've tried to list common issues and solutions below. If you followed all the steps in this project and you are not getting it to work, try the steps below. They're a good checklist for cleaning out the crud that can accumulate in Xcode and interfere with your Playground. If you run into anything else, submit a pull request and I'll look into adding it. 


Issue: Cannot import the framework I created. There can be a whole number of reasons for this.

	- Make sure your Playground is running (Press Play Button)
	- Make sure you have built against the target you created using a simulated device


Issue: Cannot find a file that should be included in your target

- Build the target (Command + B) to ensure any new files that you've included in the target available to the playground

Issue: Cannot import a pod that should be included in the CocoaPod.

- Build the target (Command + B) to ensure any new files that you've included in the target available to the playground
- Ensure you have run 'pod install' on the modified pod file after you added your new target. 

Issue: Weird Playground error involving stuff. Try the following in some combination: 

- Cleaning the project and rebuilding
- Clearing your derived data 
(XCode => Preferences => Locations => Click on arrow next to derived data path => Drag all folders into trash)
- Restarting XCode
- Restarting your computer 

Issue: Authorization not working due to keychain access restrictions. 

- I haven't figured that one out just yet, though my project doesn't require auth so I haven't put too much work into this just yet. If you find an answer, submit a pull request!



