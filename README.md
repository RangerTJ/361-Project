﻿# CS361-Project: String to Image Association
This program has two components - a local client service and a microservice that can respond to a request.  
Local users can input a string and the program will load an image from an image folder that best matches their  
string's content, based on an algorithm analysis (related to keywords stored within the program).  

The user can further customize this by using a settings menu to enter additional keywords to be used as a factor in  
analyzing strings and what images best fit. Any images in the images/ folder relatively local to the script will be  
considered. A "default.png" file is expected as part of minimum functionality, so that some form of image is always  
returned. By appropriately naming images and adding them to the image folder and creating additional keywords, the
user can expand the potential image associations created by the application.  

The user can also execute the early stages of a microservice in this first version. There is a command available to  
manually check a request pipeline JSON file for request content. If content is found, it is returned via a response  
pipeline JSON file, which can be accessed by other applications. Current microservice implementation relies on the  
other application knowing where to submit a request and find a response, and they must wait for a local user to  
manually activate the request/response function.  

The current implementation of this is a console application navigated primarily by numerical option navigation and  
typing words or strings when prompted by the system.  

# Microservice Instructions and Communication Contract
Requires Python 3.10 and installing the zmq module.  

The microservice portion of this project is designed to be a flexible way for client programs to quickly associate one of the images available to their program with a string. The intention is that this can be used to quickly and dynamically generate content where you may have a large library of images available, and you do not want to manually assign an image in an accompanying space every time. To start using the microservice, open a command-line terminal, navigate to the folder containing the microserverice, and run microservice_server.py. It will actively listen for requests and respond to them until the program is stopped (in software like PyCharm you may need to hit the "stop" button twice to fully halt the program) or the terminal window is closed. In the event the socket is left open for some reason (which, on Windows, tends to happen if you try to run the script directly from a window), pull up your computer's task manager and look for a python.exe process to terminate (under task manager "details" tab in Windows 10).  

To achieve this, the service uses socekts via ZeroMQ (https://zeromq.org/get-started/). JSON-formatted infromation is sent form a requesting client to the microservice, which acts as a server, listening in the socket's port for a request. Once the request is recieved, it runs a process to associate the strings in the request to one of the image filepaths in the request and it sends back the association to the requesting client.  

The JSON object sent by the requesting client should be the JSON-encoded form of a dictionary with two specific keys, "strings" and "images". These two keys are expected to have arrays as values, with the former being an array of strings to associate with an image, and the latter being an array of images that the requesting client has available within the client program's files. An example of a valid JSON request would be: {"strings": ["Pizza eating!", "Eat your veggies!"], "images": ["pizza.png", "carrot-veggies.png"]}  

Once the microservice has assigned images into a string:image dictionary, it will encode the dictionary to a JSON object and send it back to the requesting client. This JSON object will need to be decoded back into its non-byte form using the JSON decoding methods available to whatever respective programming language is used by the client. An example of what to expect from a decoded JSON (continuing the above example) would be: {"Pizza eating!": "pizza.png", "Eat your veggies!": "carrot-veggies.png"}  

Once the client-side program stores the results dictionary, this allows the client to directly index into matching image files using a string as a key. For example, this might allow a client to auto-generate an image to match an accompanying line of text, based on appropriate images that are already available in the client's file database.  

Note: If you would only like a single string to be associated, use the service like normal and just submit a JSON where the "strings" key's array only contains the string that you would like to be associated. (Exmaple: You have a user input a line, and as soon as their input is saved, this service is used to fetch a matching image).  

If a request is invalid, the string "format_error" will be sent back instead of an assignment dictionary, so it may be helpful to incorpoate that into client-side program logic in the case that an invalid request is somehow sent (to avoid throwing exceptions and such).  

Example python code for a client request to the Microservice:
![ExampleCall](https://user-images.githubusercontent.com/87739732/216794236-69f83c1f-e7cb-4ce2-9a20-79996fc95b85.JPG)

