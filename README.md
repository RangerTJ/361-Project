﻿# CS361-Project: Smart Launcher
This project has two components - a client-side application (smart_launcher.py) and a service (smart_selector.py). The former is a command-line program that allows the launching of files from a specified launch directory based on either relevant text inputs, or random selection. By default, the launch directory is initailized to a folder called 'Launch-Files' that will be kept local to wherever smart_launcher.py is located. Both programs should work just fine for launching any files with traditional .XYZ style 3-letter extension. As such, they are compatible with launching or selecting images, sound files, program shortcuts, and pretty much anything else that fits that naming convention. Files will be launched with whatever application is currently set as the default to handle their respective file type.  These files can be launched by typing in text, voice commands, or random selection.

UI Navigaiton is performed by typing in a number or text prompt that is listed as corresponding to the menu option, and hitting 'Enter'. At any screen, you can type "HELP" to access help text or "QUIT" to close the application. Both of these text-based commands are case-insensitive.  

Requires locally-operated microservices on ports 5555 (smart_selector.py) and 5556 (chooseRandom.py from https://github.com/fitellieburger/CS361). Both of these must be running in the background to respond to requests for text-file association and random choice selection from the main application (smart_launcher.py). The location of these files doesn't matter, as long as the scripts are running in the background while smart_launcher.py is active, and ports 5555 and 5556 are both kept free for both of these services' socket connections. You will also need to make sure that you have the zmq and speech_recognition python modules installed.

# Smart Selector - Microservice Instructions and Communication Contract
Requires Python 3.10 and installing the zmq module.  

The microservice portion of this project is designed to be a flexible way for client programs to quickly associate one of the files available to their program with a string. The intention is that this can be used to quickly and dynamically generate content where you may have a large library of files available, and you do not want to manually assign an file in an accompanying space every time. To start using the microservice, open a command-line terminal, navigate to the folder containing the microserverice, and run smart_selector.py. It will actively listen for requests and respond to them until the program is stopped (in software like PyCharm you may need to hit the "stop" button twice to fully halt the program) or the terminal window is closed. In the event the socket is left open for some reason (which, on Windows, tends to happen if you try to run the script directly from a window), pull up your computer's task manager and look for a python.exe process to terminate (under task manager "details" tab in Windows 10).  

To achieve this, the service uses socekts via ZeroMQ (https://zeromq.org/get-started/). JSON-formatted infromation is sent form a requesting client to the microservice, which acts as a server, listening in the socket's port for a request. Once the request is recieved, it runs a process to associate the strings in the request to one of the filepaths in the request and it sends back the association to the requesting client via port 5555.  

The JSON object sent by the requesting client should be the JSON-encoded form of a dictionary with two specific keys, "strings" and "files". These two keys are expected to have arrays as values, with the former being an array of strings to associate with an file, and the latter being an array of files that the requesting client has available within the client program's files. An example of a valid JSON request would be: {"strings": ["Pizza eating!", "Eat your veggies!"], "files": ["pizza.png", "carrot-veggies.png"]}  

Once the microservice has assigned files into a string:file dictionary, it will encode the dictionary to a JSON object and send it back to the requesting client. This JSON object will need to be decoded back into its non-byte form using the JSON decoding methods available to whatever respective programming language is used by the client. An example of what to expect from a decoded JSON (continuing the above example) would be: {"Pizza eating!": "pizza.png", "Eat your veggies!": "carrot-veggies.png"}  

Once the client-side program stores the results dictionary, this allows the client to directly index into matching files using a string as a key. For example, this might allow a client to auto-assign a file to match an accompanying line of text, based on appropriate files that are already available in the client's file database. In a case where more than one possible match is found for a string, one of the matching files will be randomly selected as the return. In a case where no match is found for the string, it will be matched with the string ".defaultChoice", which can be used by the client program to determine what logic to execute when no match was found. It is up to the programmer of the client to determine how they would like to process the ".defaultChoice" response to implement default behavior.  

Note: If you would only like a single string to be associated, use the service like normal and just submit a JSON where the "strings" key's array only contains the string that you would like to be associated. (Exmaple: You have a user input a line, and as soon as their input is saved, this service is used to fetch a matching file). This service also relies on the assumption that all files have 3 letter extensions. Using it in directories of files of a different extnsion length may cause incorrect substring generation. This could also be used to launch any file, not just images, so be mindful of how the service is being used and what is being passed to it.  

If a request is invalid (wrong request format or there are no requested strings), the string "format_error" will be sent back instead of an assignment dictionary, so it may be helpful to incorpoate that into client-side program logic in the case that an invalid request is somehow sent (to avoid throwing exceptions and such).  

**Example Call: Python code for a client request to the Microservice**  
![Client_example](https://user-images.githubusercontent.com/87739732/218598540-661d682c-24f1-4fa8-8d1b-ea57fa041b98.JPG)  
 
 Ouput possibilities from this request, based on random selection of files that match the string:
![Client_example_output](https://user-images.githubusercontent.com/87739732/218598629-0a099459-4bcd-4b52-aee7-f18d88e08a46.JPG)  
![Client_example_output_alt](https://user-images.githubusercontent.com/87739732/218598636-1697d6bc-71f5-4adb-9c86-92bc04a80d24.JPG)  
  
**Diagram of Client-Microservice Interaction**  
![microUMLNewNewName](https://user-images.githubusercontent.com/87739732/221490157-c0f3c5f3-6f25-4667-a291-106e46ed0f86.png)



