﻿# CS361-Project: String to File Association
This program has two components - a local client service and a microservice that can respond to a request. Local users can input a string and the program will load a file from a folder that best matches their string's content, based on an algorithm analysis (related to keywords stored within the program). By default, this is initailized to an images folder local to this application with a default.png gile. While it works for images (and images were the initial idea), it should work with any file type as well, as long as it has a .XYZ style 3-letter extension. So this could also be feasibly used to launch sound files, programs, or shortcuts.

This is a command-line application and navigated primarily with numerical inputs corresponding to options at each menu screen. At any screen, you can also type "HELP" to access help text, "MAIN" to return to the main menu, or "QUIT" to close the application (case-insensitive). The three current main functions of the application are a manual service to launch a file that most closely matches a string the user is prompted to input, a service to launch a random file from those available, and an option to list all the files availablef or association in the current "target" file directory. Files are launched with whatever program is assigned to be the default application to handle that type of file by your operating system. 

Down the road, I am looking to add support for changing the "target" file directory of available files for association while within the program, and adding a feature that allows you to save the new location of the new target directory so that it persists between sessions (along with a restore to default option).

Requires locally-operated microservices on ports 5555 (microservice_server.py) and 5556 (randomizer python script - details to follow later).  

# Microservice Instructions and Communication Contract
Requires Python 3.10 and installing the zmq module.  

The microservice portion of this project is designed to be a flexible way for client programs to quickly associate one of the files available to their program with a string. The intention is that this can be used to quickly and dynamically generate content where you may have a large library of files available, and you do not want to manually assign an file in an accompanying space every time. To start using the microservice, open a command-line terminal, navigate to the folder containing the microserverice, and run microservice_server.py. It will actively listen for requests and respond to them until the program is stopped (in software like PyCharm you may need to hit the "stop" button twice to fully halt the program) or the terminal window is closed. In the event the socket is left open for some reason (which, on Windows, tends to happen if you try to run the script directly from a window), pull up your computer's task manager and look for a python.exe process to terminate (under task manager "details" tab in Windows 10).  

To achieve this, the service uses socekts via ZeroMQ (https://zeromq.org/get-started/). JSON-formatted infromation is sent form a requesting client to the microservice, which acts as a server, listening in the socket's port for a request. Once the request is recieved, it runs a process to associate the strings in the request to one of the filepaths in the request and it sends back the association to the requesting client via port 5555.  

The JSON object sent by the requesting client should be the JSON-encoded form of a dictionary with two specific keys, "strings" and "files". These two keys are expected to have arrays as values, with the former being an array of strings to associate with an file, and the latter being an array of files that the requesting client has available within the client program's files. An example of a valid JSON request would be: {"strings": ["Pizza eating!", "Eat your veggies!"], "files": ["pizza.png", "carrot-veggies.png"]}  

Once the microservice has assigned files into a string:file dictionary, it will encode the dictionary to a JSON object and send it back to the requesting client. This JSON object will need to be decoded back into its non-byte form using the JSON decoding methods available to whatever respective programming language is used by the client. An example of what to expect from a decoded JSON (continuing the above example) would be: {"Pizza eating!": "pizza.png", "Eat your veggies!": "carrot-veggies.png"}  

Once the client-side program stores the results dictionary, this allows the client to directly index into matching files using a string as a key. For example, this might allow a client to auto-assign a file to match an accompanying line of text, based on appropriate files that are already available in the client's file database.  

Note: If you would only like a single string to be associated, use the service like normal and just submit a JSON where the "strings" key's array only contains the string that you would like to be associated. (Exmaple: You have a user input a line, and as soon as their input is saved, this service is used to fetch a matching file). This service also relies on the assumption that all files have 3 letter extensions. Using it in directories of files of a different extnsion length may cause incorrect substring generation. This could also be used to launch any file, not just images, so be mindful of how the service is being used and what is being passed to it.  

If a request is invalid, the string "format_error" will be sent back instead of an assignment dictionary, so it may be helpful to incorpoate that into client-side program logic in the case that an invalid request is somehow sent (to avoid throwing exceptions and such).  

**Example python code for a client request to the Microservice**  
This also has examples of the kinds of ZMQ socket commands used to send and recieve the JSON files via a client.
![ExampleCall](https://user-images.githubusercontent.com/87739732/217414488-e9f8bb1f-676a-4a03-985c-2f7329e7d165.JPG)

**Diagram of Client-Microservice Interaction**
![microUMLNewName](https://user-images.githubusercontent.com/87739732/217414999-c40c4227-807c-47ef-a3c4-c5b43f6efeb1.JPG)


