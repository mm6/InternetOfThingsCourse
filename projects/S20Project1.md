# 95-733 Internet of Things Tuesday, May 19, 2020

## Project 1  Due: 11:59 PM, Tuesday, June 2, 2020


### Programming with Servlets, AJAX, JSON, Web Sockets, Particle Photon


This project is intended as an introduction to web programming
and the Particle Photon micro controller.


This project has several objectives. First, you will be exposed
to building Java servlets and server side programming. Second,
you will learn to use Javascript and HTML for client side
programming. Third, you will learn about cookies and
session handling and how several concurrent visitors may maintain
state on the server. Fourth, you will work with a single page
web application that uses AJAX for asynchronous calls. Fifth,
you will be introduced to websockets. And finally, you will build
a web application that reports on the health of a Particle Photon.

0) [Read the article by Philip McCarthy on Ajax](http://www.ibm.com/developerworks/library/j-ajax1/)
   The source code is available in a zip file at the bottom
   of the article. The web.xml file is important and needs
   to be included under Web Pages/WEBINF in IntelliJ. Also,
   be sure to turn off Oracle if it is using port 8080.

   There are notes to help you install IntelliJ and TomEE Plus. [See IntelliJ_Installs.](https://www.andrew.cmu.edu/user/mm6/95-733/IntelliJ_Installs.pdf)

1) 5 Points. Build a working Ajax shopping cart using the code from the article.
   So that the TA's have an easy time grading the assignments, IntelliJ
   and TomEE Plus will be used as demonstrated in class.

   Name your project ACoolAJAXShoppingCart.

   [IntelliJ Ultimate may be downloaded from here](https://www.jetbrains.com/idea/)

   Grading: The grader will test this and will be looking for solid documentation.
   That is, you must provide comments throughout the code. These comments will
   server to convince the grader that you have studied the code and understand
   it and that you are able to clearly and succinctly explain what it does.

   You are being asked to document code that you did not write.

2) 15 Points. Improve the application by adding a button to each item
   displayed so that the user may delete items from the shopping
   cart. Your delete button will behave like the add button, but
   will count down rather than up. If an item is not currently in
   the cart then a delete request will be ignored. If a delete
   request causes a quantity to go to zero then the
   deleted item will be removed from the cart and no longer
   displayed.

   Name your project ACoolAJAXShoppingCartImproved.

   Finally, add a new item to the shopping card catalog. This item will
   be of your own choosing.

   Grading. The documentation from the first part will be found
   within this code as well. In addition, be sure to document the
   modifications that you made. The application will be tested.

3) 25 Points. Modify your solution to part 2 so that it uses
   JSON rather than XML. Note that your servlet will generate a
   content type of application/json and your XMLHttpRequest
   object will return JSON via the responseText property
   rather than the responseXML property.

   This part requires that you design a JSON string that
   corresponds to the XML shopping cart.

   Note there is a full example [provided here](JSONHelloWorld.md) showing how JSON may be generated on the server and read by the client.

   [For a careful description of JSON see here](http://www.json.org/)

   Note that there is a bug in this web application. Enter a few
   items into the shopping cart and then refresh the page. Only after
   an additional item is selected will the contents of the cart reappear.
   Make a simple modification to the index.jsp page to repair this
   bug.

   Name your project ACoolAJAXShoppingCartImprovedWithJSON

   Grading. The documentation from the second part will be found
   within this code as well. In addition, be sure to document the
   modifications that you made. How were you able to fix the bug?
   This should be clearly specified in the documentation.

   You may choose to use a library to generate the JSON or you may
   choose to generate the JSON by hand coding - as the XML is hand
   coded in McCarthy's article.

4) 30 Points. Deploy the Whiteboard application that Oracle uses
   to introduce websockets to TomEE Plus using IntelliJ Ultimate.

   [See whiteboard instructions](Whiteboard_Instructions.md)

   Study the code in the Whiteboard application and use it as a guide
   to building a collaborative shopping cart. That is, build a shopping
   cart that allows two or more users to shop together - seeing the same
   cart from separate machines. Modify the code from the IBM article by
   McCarthy. When an item is added or deleted, everyone should be notified
   without polling the server.

   No AJAX is allowed in this part of Project 1. It is based on websockets.

   Name your project WhiteboardAppProject.

5) 20 Points. Use this [firmware provided](SimpleHTTPClient.md) for your Particle Photon.

   Write a web application that allows a user to monitor the heartbeats
   generated by the Photon. You are required to 1) support updates
   via a full page request and 2) support updates via AJAX calls.
   The IntelliJ project will be named PhotonTracker with a servlet
   named PhotonServlet.

   When the web application is visited by a browser, the browser will
   display the ID of the Photon along with the time of the last heart
   beat.

   If the Photon has been disconnected for 20 seconds or more, the
   web application should report a suspected failure.

   If, after a disconnection, the Photon is connected again, the web application
   will show its heart beat timestamps as before.

   We are not using web sockets in this part of Project 1.

   Name your project PhotonTracker.

Submission Requirements 5 Points
================================

Place all of your work into a single directory named Project1.

This directory will contain the five IntelliJ projects as named above. Zip this directory into a file named <yourAndrewID>Project1.zip.

For example, mine would be named mm6Project1.zip.

Submit your zip file to Canvas.
