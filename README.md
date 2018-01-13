# Chrome Popunder

This was just a reverse engineering project in JavaScript. I was given a link to a demo showing the popunder and wanted to know how it worked. This write up is from a few months after this project so it might be slightly off.

## Procedure
I started by trying to get this to show on my own machine. I used python to host a web server and then changed the url's that I could find and the page would load. However there was a license file that tried to prevent you from running the demo so even with the page loaded there was no popunder.

I attempted to reverse the license file but it was just turning into a rabbit hole and I thought I would have better luck with the script file.

Through some brute force commenting I was able to find a section of code that could be removed and allowed the demo to run on my machine.

With the demo running I was finally able to pop open the Chrome developer tools and use the performance module to find what functions were being called. 

From there it was just the matter of setting some breakpoints in those functions and stepping through them in order to deobfuscate them.

## Overview
This popunder works by having an onclick event that will create a new tab. 

'''HTTP
<h1 onmousedown="openPopunder()">Test Popunder</h1>
'''

That new tab has a "mouseup" event listener that creates a new window, hides it by shrinking and moving it off screen, and adds a focus event listener to the tab.

'''javascript
blank_tab.addEventListener("mouseup", function i() 
...
    new_window.resizeTo(1,0);
    new_window.moveTo(9e5,9e5);
    blank_tab.addEventListener("focus", function() 
'''

That new window that was created should now be the focus of the web browser. However, the tab that was opened tries to download a pdf which causes an alert box and brings that tab back to the front.

'''javascript
blank_tab.document.body.innerHTML = '\
    ...
        <object data="data:application/pdf;base64, ...">
'''

With the focus back on the tab the event from before triggers. This event causes the tab to close and the hidden window to resize in the background.

'''javascript
blank_tab.addEventListener("focus", function()  
    ...
    blank_tab.close();
    new_window.moveTo(0,0);
    new_window.resizeTo(1920, 1080);
    ...
'''
