### A simple manifesto: let's make the web more accessible
My mum, like many others, has MS. The only movement she has from her neck down is in her right forefinger. These days she doesn't even have the strength to use a track pad, and in the next few months she probably won't be able to press the next page button on her Kindle.

It's particularly frustrating as she's very tech savvy: she emails, browses the web, shops too much, stalks me and my siblings on Facebook and is savvy enough to know Twitter is overrated. The challenge is to help her, any many thousands of others to continue to use the web without motor skills.

### OS X voice recognition: simple, reliable
She's tried out quite a lot of advanced kit, including eye tracking kit hooked in to a tablet. Eye control isn't quite mature enough, and the software supporting is pretty fragile. I'm sure we'll tackle that next, but in the mean time let's exploit a fantastic feature in Mac OS X - voice recognition.

It's incredibly accurate, more so than other packages she's used. What's particularly appealing is that it's exposed via an API, meaning it's possible to build an app to allow disabled users to browse the web using their voice.

**Which leads to the aim of this project: building a browser plugin and supporting tools to allow navigation of the web using just voice commands.**

<hr>

### Architecture

#### Browser plugin / extension
I'm currently working on a browser plugin. It scans the DOM of the web page, and extracts context-sensitive options for the user. For example, to navigate to links, complete forms, go back etc. The plugin's role will be to extract the list of candidate commands, and act on the user's voice command. 

For security reasons, browser plugins can't access the OS's API - for example to start voice recognition. Therefore, the browser plugin will communicate with a desktop helper app using a WebSockets interface. The desktop app will take care of voice recognition and return the results back to the plugin.

#### Desktop app (OS X)
The desktop app will run in the background - perhaps with a status icon in the menu bar. It will listen for WebSocket sessions where it will be given a list of voice commands to listen for (probably as a JSON string). The app will then initiate voice recognition via [NSSpeechRecognizer](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ApplicationKit/Classes/NSSpeechRecognizer_Class/index.html#//apple_ref/occ/cl/NSSpeechRecognizer). When the OS recognises a command it will notify the desktop app, which will then notify the browser plugin. When then executes the command.

I'm not a OS X developer, but I hacked this together and it shows the basic principle in action:

```objectivec
class ViewController: NSViewController, NSSpeechRecognizerDelegate {
    
    
    var speech = NSSpeechRecognizer.init()
    
    
    override func viewDidLoad() {
        super.viewDidLoad()
        speech.commands = ["Follow link","Search","Fill form", "Go back"]
        speech.listensInForegroundOnly = false
        speech.startListening()
        speech.delegate = self
    }

    override var representedObject: AnyObject? {
        didSet {
        // Update the view, if already loaded.
        }
    }

    func speechRecognizer(sender: NSSpeechRecognizer, didRecognizeCommand command: AnyObject?) {
        
        var c:String = command as String!
        if c == "Fill form"
        {
            speech.commands  = ["Form 1", "Form 2"]
        } else if c == "Follow link"
        {
            speech.commands = ["Home","About Us","Contact"]
            
        } else {
						// etc etc etc
            speech.commands = ["Follow link","Search","Fill form", "Go back"]

        }
        
    }

}
```

#### Protocol spec (WIP)

##### Commands
* listenFor (string array of commands)
* stopListening
* resumeListening
* getStatus 
* startDictation (start OS dictation, like pressing fn-fn in a focused input)

##### Return
* response: listening, waiting, error
* command: (command spoken by user)


#### Challenges
A few things that will need to be considered:
* Tabbed / multi window browsing - ensuring dictation is in sync with current window
* SSL - hopefully sandboxing of browser extensions will allow for non-SSL access when the page is SSL - otherwise this approach won't work on SSL sites


