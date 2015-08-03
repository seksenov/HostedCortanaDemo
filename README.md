# HostedCortanaDemo
Integrating with Cortana on Windows 10 from a remote website.

## In Progress
This project is in progress. It currently supports the Cortana command "Cortana Demo send message" followed by any spoken phrase. The html is not complete to show the text being passed back from Cortana to the site but it can be seen in the console by hitting F12 when the Hosted App is running.

## Steps Required to Integrate Cortana on any Hosted Web App

### Create a Voice Command Definition (VCD) file

Create an xml VCD file defining your app name, the set of commands your app will handle and how it Cortana will respond to users. The schema definition can be found [here](https://msdn.microsoft.com/en-us/library/windows/apps/dn706593.aspx). Once you've created the VCD file you need to place it on your server where it's accessible from the web. 

Here is the one used in the sample project:

```xml

<?xml version="1.0" encoding="utf-8"?>
<VoiceCommands xmlns="http://schemas.microsoft.com/voicecommands/1.2">
  <CommandSet xml:lang="en-us" Name="CortanaDemo">
    <CommandPrefix>Cortana Demo</CommandPrefix>
    <Example>Cortana demo send this is a test</Example>
    <Command Name="send">
      <Example>send {message} using cortana demo</Example>
      <ListenFor RequireAppName="BeforeOrAfterPhrase">send message {messageSubject}</ListenFor>
      <Feedback>sending {messageSubject} to Cortana Demo</Feedback>
      <Navigate Target="/sendMessage.htm"/>
    </Command>
    <PhraseTopic Label="messageSubject" Scenario="Dictation"></PhraseTopic>
  </CommandSet>
</VoiceCommands>

```

### Add the meta tag pointing to the location of your VCD

Next you'll need to add the Cortana meta tag to you html start page. When your app is launched the App Host will scan the html content for a Cortana specific meta tag that points to the location of you VCD file. The App Host will automatically download and register the VCD file with Windows. After this Cortana will recognize the commands uttered by users and activate your app.

Here is the meta tag that is added for this project:

```html

<meta name="msapplication-cortanavcd" content="http://seksenov.github.io/HostedCortanaDemo/vcd.xml"/>

```

### Handle the Cortana activation

To complete Cortana integration you will need to handle the activation event from Cortana in your app. Your app will be need to handle the activation using Windows APIs as show below and extract the speech recognition from the arguments of the event.

Here is the handling in this project:

```js

if (typeof Windows !== 'undefined') {
  console.log('Windows namespace is available');
  // Subscribe to the Windows Activation Event
  Windows.UI.WebUI.WebUIApplication.addEventListener('activated', function (args) {
    var activation = Windows.ApplicationModel.Activation;
    // Check to see if the app was activated by a voice command
    if (args.kind === activation.ActivationKind.voiceCommand) {

      var speechRecognitionResult = args.result;
      var textSpoken = speechRecognitionResult.text;
      var command = speechRecognitionResult.rulePath[0];

      console.log('The command is: ' + command);
      document.getElementById('command').innerHTML = command;
      document.getElementById('speechReco').innerHTML = speechRecognitionResult;
      document.getElementById('text').innerHTML = textSpoken;

      // Determine the command type {play} defined in vcd
      if (command === 'send') {
        // Determine the stream name specified
        console.log('The speech reco result is: ' + speechRecognitionResult);
        console.log('The text spoken is: ' + textSpoken);
      }
      else { 
        // No valid command specified
        console.log('No valid command specified');
      }
    }
  });
} else {
  console.log('Windows namespace is unavaiable');
}

```

## Setup:
1. Clone or fork the repo
2. npm install
3. gulp appx:dev

