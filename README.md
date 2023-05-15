# StreamerBotWSClientExtension
This extension allows you to have the tracker act as a Websocket Client, specifically to interact with a Websocket Server hosted in Streamer Bot to have the tracker communicate data changes instantly.

This extension requires steps beyond the typical "copy files and enable the extension in the tracker", some of which will be technically intensive, so please try to follow the steps as closely as possible.
Prerequisites:
- Windows 10 (11 is probably fine too, but haven't tried it)
- Bizhawk 29.9.1 (NOTE: we will be effectively reinstalling Bizhawk so you don't need to already have it)
- Github Desktop (Git also works, Github Desktop is easier to use so intructions will reference this)
- Visual Studio Code (Visual Studio would also work) aka VS Code
- Streamer Bot 0.1.20+

For those of you who are fairly savvy to this kind of thing the basic steps you need to do are:
1) Create the Custom Websocket Server in Streamer Bot and assign appropriate actions
2) Build Bizhawk manually so you can enable its websocket capabilities
3) Install the Tracker Extension files and enable the extension.

Streamer Bot setup:
1) Open Streamer Bot, and navigate to Action Queues -> Queues
2) Right click the empty white space and select Add. You can name this queue what you like, but make sure Blocking is not checked.
3) Navigate to the Actions tab.
4) Right-click the empty space and click Add. You can name this what you like, but know that it will be triggered when the Tracker connects to Streamer Bot. Make sure that Enabled is checked, and select the Action Queue you created in step 2) from the Queues dropdown.
5) Repeat step 4) twice. These new actions will be triggered when Streamer Bot receives a message from the tracker, and when the tracker disconnects from Streamer Bot, respectively.
6) Navigate to the Servers/Clients -> Websocket Servers tab (this is the LAST tab, not the first one which is similarly named)
8) Right-click the empty white space and click Add
9) You can name this whatever you like, but you must provide the following inputs:
  - Address: 127.0.0.1
10) I recommend checking the 'Auto Start on Startup' option, and using port 8080 for simplicity. In the Actions section, assign the 3 Actions you created early for Connected, Disconnected, and Message respectively. Once everything is entered click OK.
11) Right-click the newly create Server and check that it says 'Stop' near the bottom of the list, this means the Server is started. If it instead says 'Start', you'll need to click on 'Start'.

Github Desktop
1) Installing Github Desktop is fairly straightforward. At some point during the installation/setup, you will have the option to select an External editor. You can select VS Code here if you have it installed, makes things a little easier later
2) If you don't yet have VS Code installed at this time, you can open Github Desktop later and go to File-> Options -> Integrations and select it there.

(If you haven't already, install Visual Studio Code/have your IDE ready for the next step)

Bizhawk
1) In Github Desktop, go to File -> Clone Repository. Go to the URL tab and provide the following URL: https://github.com/TASEmulators/BizHawk.git
  - For the local path I recommend choosing somewhere not in OneDrive or on your desktop. Otherwise it should be fine. Once everything is entered click Clone. The process will take a while
2) Once the repository is created go to Repository -> Open in Visual Studio Code
3) In VS Code, open this file: "\src\BizHawk.Client.Common\lua\CommonLibs\CommLuaLibrary.cs" and add this at line number 6 (under all the other using System.xyz lines):
using System.Net.WebSockets;
4) In the VS Code menu, go to Terminal -> New Terminal
5) In the Terminal at the bottom of the window, paste the following command and hit Enter (this may take some time):
dotnet build -p:MachineExtraCompilationFlag=ENABLE_WEBSOCKETS
6) Once done, you should find in the folder where you copied the bizhawk files, a folder called output, which contains the usual files and folders that you'd find in your regular Bizhawk installation.
7) At this point, you should copy any save/save state/settings data from the Bizhawk installation that you typically use into the same locations in this new folder. Otherwise, it will be like a fresh Bizhawk installation with all default settings and keybindings.
8) There should be an Application executable called "EmuHawk" in the output folder, running it will launch Bizhawk.
9) Download the Extension file and move it to your Extensions folder in the tracker folder
10) Open your game, open your tracker, and enable the Extension under Settings->Extensions. And you should be good to go, if the Websocket Server is running then the tracker will have connected it, and you should see that the Action you assigned in the Websocket Server to "Connected" should have run.

From here on out, the world is your oyster, but you'll need to do some programming to set things up. You'll need to edit the extension .lua file directly to control how and what the Tracker communicates to Streamer Bot, and you'll also need to configure the Message Action of the Streamer Bot server to handle varying requests and perform different actions. I'd recommend having the Message action trigger a C# code subaction, and handling the message from there. The message sent from the tracker can be accessed in the C# code via args["data"]
