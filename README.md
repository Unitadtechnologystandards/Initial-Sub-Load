# Initial-Sub-Load

###Intro

The L.E.A.N (**L**ight, **E**ncrpyted, **A**dChoice supported, **N**on-invasive) Ad Principals, released by the IAB as a new standard for serving online advertisements, aims to give ad creators, as well as publishers, a framework in which ads can be created and served as easily and lightweight as possible.

To archives this a creative may have 3 distinct loading phases, each limiting the ad on how many files and request if may process in that phase, so the ad may load as fast as possible and be visible as soon as possible to the website visitor.

* "initial" phase: The creative may load a limited number of resources (like images and scripts), to provide a first but functional presentation of the ad. Furthermore it may load trackings to measure simple technical KPI's like impression and viewability. Most publisher's limit the maximum total file weight (example 100 KB) and  the amount of file request during this phase.  Make sure you are aware of the exact limitations for each publisher. <br><br>OVK members will all support the specification at http://www.werbeformen.de/ovk/ovk-de/werbeformen/spezifikationen/allgemeine-spezifikationen/initialsubload.html


* "subload" phase: Only when the publisher website is fully loaded, may the ad load more and heavier resources. These include animations, light video files or other interactive experiences. All of these resources still need to adhere to a maximum file weight for the whole creative (example +200KB), but without a limit to file requests.

* "user initiated" phase: Only when the user interacts with the ad (swipe, click), the ad may load further resources without any limits. This can be full-length video spots or other heavy media files.


For ad creators the "inital" phase is easy to program and accommodate. The loading process of the ad starts, so the ad can do a limited amount of steps before it has to stop and needs to wait for the "subload" phase.

To correctly anticipate the "subload" phase however, might prove more difficult.  Most ads are "locked" inside a SafeFrame (an iframe with no way to access the site), or nested deep inside of iframe-chains with different solutions and scenarios on each website.

To provide an easy solution for ad creators and publisher alike the OVK developed a lightweight and fast solution, which can be inserted into any creative and correctly determines when to start the subload, no matter on which website the creative is served.

Based on JavaScript, the solution can be pasted into the JavaScript code of the creative and exposes a new "Event" called "iabSubLoadStart". Any steps the creative should take during "subload", can be added via an EventListener on the "window" object of the creative.<br><br>


###What you need to do
To correctly integrate the solution, follow these simple steps:

1.) Paste the following code into the JavaScript portion of your creative. This code is minimized to be as lightweight as possible. You will find a readable form at the end of this document.
```
!function(){var e={y:window.top!==window?window.top:window,t:4,p:function(){try{return window.top.document}catch(t){return!1}}(),w:function(t){"string"==typeof t.data&&"IAB_HOST_LOADED"===t.data&&(window.clearTimeout(e.k),e.x())},k:"",q:"function"==typeof CustomEvent?new CustomEvent("iabSubLoadStart"):document.createEvent("Event").initEvent("iabSubLoadStart",!0,!0),x:function(){return self.dispatchEvent(e.q)}};e.p?"complete"!==e.y.document.readyState?(console.log("SETTING READYSTATE LISTENER"),e.y.document.addEventListener("readystatechange",function(){"complete"===e.y.document.readyState&&e.x()})):this.x():(window.addEventListener("message",e.w.bind(e)),e.k=setTimeout(function(){window.removeEventListener("message",e.w),e.x()},1e3*e.t))}();
``` 


2.) Create a new JavaScript function, which will handle the "subload" phase of your creative (loading further resources, starting animations, and so on). For example, name it "startCreativeSubload".

3.) Add a new EventListener for "iabSubLoadStart" to the window object, reference your new "subload" function as second argument of "addEventListener".
 
 
 ```
 <script>
 window.addEventListener('iabSubLoadStart', startCreativeSubLoad)
 </script>
 ``` 

4.) Your done! The creative will now start "subload" as soon as it is able to. It is recommended to test your creative thoroughly. 


### How it works
The solution will check for 2 possible states/points in time to start "subload":
* The website can be reached, and it's load status can be read by the creative
* The website can not be reached, it's load status is unknown or can only be determined, if the website supports inter-iframe communications.


For the first state, the creative will try to contact the website and monitor its load status. If it is already loaded, the "subload" phase will start at once. Otherwise the creative will wait until the website finishes loading.

For the second state, the creative will wait a minimum of 4 seconds for the site to contact the creative and communicate its load status. If the site does not support inter-iframe communication, "subload" will start after 4 seconds. If the website contacts the creative before the 4 seconds, the "subload" phase will start as soon as the connection is established.


``` 
/**
		 * Trigger Script  (unminified)
		 *
		 * Tries to determine how the publisher delivers the creative to the website.
		 * There are 4 possibilities:
		 *  - Directly on the website via a Javascript. We can read the "host" site and wait for a complete load
		 *  - Via an iframe, separating the creative from the website, but in a friendly Iframe. We still can read when the "host" is completely loaded.
		 *  - Via an unfriendly iframe, completely blocking access to the website. In this case we just assume the "host" is completely loaded after X seconds.
		 *  - Via an unfriendly iframe, but the websites sends the message "IAB_HOST_LOADED" when it has finished loading.
		 * As soon as we think the website is loaded, the creative can initiate the subload by listening for the event "iabSubLoadStart".
		 **/



(function () {
    var iabSubLoadLib = {
        mainWindow: (window.top !== window) ? window.top : window,
        waitTimer: 4,
        friendlyFrame: (function () {
            try {
                return (window.top.document);
            } catch (e) {
                return false;
            }
        })(),
        postMessageHandler: function (event) {
            if (typeof event.data === 'string' && event.data === 'IAB_HOST_LOADED') {
                window.clearTimeout(iabSubLoadLib.loadTimer);
                iabSubLoadLib.startSubLoad();
            }
        },
        loadTimer: '',
        subLoadEvent: (typeof CustomEvent === 'function') ? new CustomEvent('iabSubLoadStart') : (document.createEvent('Event')).initEvent('iabSubLoadStart', true, true),
        startSubLoad: function () {
            return self.dispatchEvent(iabSubLoadLib.subLoadEvent);
        }
    };
    if (iabSubLoadLib.friendlyFrame) {
        //we know we are in a friendly iframe or directly on the site, check readystate
        if (iabSubLoadLib.mainWindow.document.readyState !== 'complete') {
            //site is still loading, use event
            iabSubLoadLib.mainWindow.document.addEventListener('readystatechange', function () {
                if (iabSubLoadLib.mainWindow.document.readyState === 'complete') {
                    iabSubLoadLib.startSubLoad();
                }
            });
        } else {
            //site is already loaded, start subload
            this.startSubLoad();
        }
    } else {
        //we are in an unfriendly iframe, wait a bit before subload starts or the site sends a message that it is loaded
        window.addEventListener('message', iabSubLoadLib.postMessageHandler.bind(iabSubLoadLib));
        iabSubLoadLib.loadTimer = setTimeout(function () {
            window.removeEventListener('message', iabSubLoadLib.postMessageHandler);
            iabSubLoadLib.startSubLoad();
        }, iabSubLoadLib.waitTimer * 1000);
    }
}());
		
``` 





 
 