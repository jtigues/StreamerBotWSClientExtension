# StreamerBotWSClientExtension
This Extension allows you to have the tracker act as a Websocket Client, specifically to interact with a Websocket Server hosted in Streamer Bot to have the tracker communicate data changes instantly.

This Extension requires steps beyond the typical "copy files and enable the Extension in the tracker", some of which will be technically intensive, so please try to follow the steps as closely as possibleto avoid complicated debugging later.
Prerequisites:
- Windows 10 (11 is probably fine too, but haven't tried it)
- Bizhawk 2.9+ (NOTE: we will be effectively reinstalling Bizhawk so you don't need to already have it, and you probably can't use what you already have anyway)
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
10) I recommend checking the 'Auto Start on Startup' option, and using port 8080 for simplicity. If you have something using port 8080 you'll need to change it to an unused port. In the Actions section, assign the 3 Actions you created early for Connected, Disconnected, and Message respectively. Once everything is entered click OK.
12) Right-click the newly create Server and check that it says 'Stop' near the bottom of the list, this means the Server is started. If it instead says 'Start', you'll need to click on 'Start'.

Github Desktop
1) Installing Github Desktop is fairly straightforward. At some point during the installation/setup, you will have the option to select an External editor. You can select VS Code here if you have it installed, makes things a little easier later
2) If you don't yet have VS Code installed at this time, you can open Github Desktop later and go to File-> Options -> Integrations and select it there.

(If you haven't already, install Visual Studio Code/have your IDE ready for the next step)

Bizhawk
1) In Github Desktop, go to File -> Clone Repository. Go to the URL tab and provide the following URL: https://github.com/TASEmulators/BizHawk.git
  - For the local path I recommend choosing somewhere not in OneDrive or on your desktop. Otherwise it should be fine. Once everything is entered click Clone. The process will take a while
2) Once the repository is created go to Repository -> Open in Visual Studio Code
3) In VS Code we need to make a couple changes, the websocket support has a couple bugs:
  i) open this file: "\src\BizHawk.Client.Common\lua\CommonLibs\CommLuaLibrary.cs" and add this at line number 6 (under all the other using System.xyz lines. If you see this line there already, you can skip this part, maybe they fix it before you see this):  
  using System.Net.WebSockets;  
  ii) In the same file, find this section (lines 298-306 for me):  
    [LuaMethod("ws_close", "Close a websocket connection with a close status")]  
		[LuaMethodExample("local ws_status = comm.ws_close(ws_id, close_status);")]
		public void WebSocketClose(
			string guid,
			WebSocketCloseStatus status,
			string closeMessage)
		{
			if (\_websockets.TryGetValue(Guid.Parse(guid), out var wrapper)) wrapper.Close(status, closeMessage);
		}  
     - and replace it with this:  
    [LuaMethod("ws_close", "Close a websocket connection with a close status")]
    [LuaMethodExample("local ws_status = comm.ws_close(ws_id, close_status);")]
		public void WebSocketClose(
			string guid,
			int status,
			string closeMessage)
		{
			if (\_websockets.TryGetValue(Guid.Parse(guid), out var wrapper)) wrapper.Close((WebSocketCloseStatus)status, closeMessage);
		}    
4) In the VS Code menu, go to Terminal -> New Terminal
5) In the Terminal at the bottom of the window, paste the following command and hit Enter (this may take some time):  
dotnet build -p:MachineExtraCompilationFlag=ENABLE_WEBSOCKETS
6) Once done, you should find in the folder where you copied the bizhawk files, a folder called 'output', which contains the usual files and folders that you'd find in your typical Bizhawk installation.
7) At this point, you should copy any save/save state/config data from your existing Bizhawk installation into the same locations in this new folder. Otherwise, it will be like a fresh Bizhawk installation with all default settings and keybindings.
8) There should be an Application executable called "EmuHawk" in the output folder, running it will launch Bizhawk. Note that a shell window will open up with it; similar to the Randomizer if you close this window, Bizhawk itself will close. But you can just minimize or ignore it.
9) Download the StreamerBotClient.lua file and move it to your Extensions folder in the tracker folder
10) Open your game, open your tracker, and enable the Extension under Settings->Extensions. And you should be good to go, if the Websocket Server is running then the tracker will have connected it, and you should see that the Action you assigned in the Websocket Server to "Connected" should have run.
  - I should mention here that if you try to connect to the Websocket Server and it isn't running, the Tracker will crash, so make sure that side of things is sorted out.
  - If you changed the server port when setting up Streamer Bot, you'll need to find the line in StreamerBotClient.lua which reads 'Port = "8080",' and change the number to whatever port you decided to do this.
12) If you had to change the port value back in step 10 of the Streamer Bot setup, go to the Streamer Bot Client Extension

From here on out, the world is your oyster, but you'll need to do some programming to set things up. You'll need to edit StreamerBotClient.lua file directly to control how and what the Tracker communicates to Streamer Bot. I've set up the basic functions to connect to the bot, send messages, and disconnect from the server, but you'll also need to configure how the Extension detects changes in the game data and sends messages to the Server, as well as configure the Streamer Bot's Message action handle your various requests and perform different actions. I'd recommend having the Message action trigger a C# code subaction, and handling the message from there. The message sent from the tracker can be accessed in the C# code via args["data"]
I've attached a couple sample C# files that work with the start-of-battle announcement message that comes with the extension. WSMessage is attached to the Websocket Server's 'Message' Action, and eventually calls WSsendMessage to send the message to chat.
