###What you need to do
To correctly integrate the solution, follow these simple steps:

1.) Paste the following code into the JavaScript portion of your creative. This code is minimized to be as lightweight as possible. You will find a readable form at the end of this document.
```
!function(){var e={y:window.top!==window?window.top:window,t:4,p:function(){try{return window.top.document}catch(t){return!1}}(),w:function(t){"string"==typeof t.data&&"IAB_HOST_LOADED"===t.data&&(window.clearTimeout(e.k),e.x())},k:"",q:"function"==typeof CustomEvent?new CustomEvent("iabSubLoadStart"):document.createEvent("Event").initEvent("iabSubLoadStart",!0,!0),x:function(){return self.dispatchEvent(e.q)}};e.p?"complete"!==e.y.document.readyState?(console.log("SETTING READYSTATE LISTENER"),e.y.document.addEventListener("readystatechange",function(){"complete"===e.y.document.readyState&&e.x()})):e.x():(window.addEventListener("message",e.w.bind(e)),e.k=setTimeout(function(){window.removeEventListener("message",e.w),e.x()},1e3*e.t))}();
``` 


2.) Create a new JavaScript function, which will handle the "subload" phase of your creative (loading further resources, starting animations, and so on). For example, name it "startCreativeSubload".

3.) Add a new EventListener for "iabSubLoadStart" to the window object, reference your new "subload" function as second argument of "addEventListener".

 
 ```
 <script>
 window.addEventListener('iabSubLoadStart', startCreativeSubLoad)
 </script>
 ``` 

4.) Your done! The creative will now start "subload" as soon as it is able to. It is recommended to test your creative thoroughly. 

Important: It must be ensured that the subload is only started once per creative, since the corresponding signals may be passed multiple times.


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
            iabSubLoadLib.startSubLoad();
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
