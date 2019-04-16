# Initial-Sub-Load

###Intro

The L.E.A.N (**L**ight, **E**ncrpyted, **A**dChoice supported, **N**on-invasive) Ad Principals, released by the IAB as a new standard for serving online advertisements, aims to give ad creators, as well as publishers, a framework in which ads can be created and served as easily and lightweight as possible.

To archives this a creative may have 3 distinct loading phases, each limiting the ad on how many files and request if may process in that phase, so the ad may load as fast as possible and be visible as soon as possible to the website visitor.

* "initial" phase: The creative may load a limited number of resources (like images and scripts), to provide a first but functional presentation of the ad. Furthermore it may load trackings to measure simple technical KPI's like impression and viewability. Most publisher's limit the maximum total file weight (example 100 KB) and  the amount of file request during this phase.  Make sure you are aware of the exact limitations for each publisher. <br><br>OVK members will all support the specification at http://www.werbeformen.de/ovk/ovk-de/werbeformen/spezifikationen/initialsubload.html


* "subload" phase: Only when the publisher website is fully loaded, may the ad load more and heavier resources. These include animations, light video files or other interactive experiences. All of these resources still need to adhere to a maximum file weight for the whole creative (example +200KB), but without a limit to file requests.

* "user initiated" phase: Only when the user interacts with the ad (swipe, click), the ad may load further resources without any limits. This can be full-length video spots or other heavy media files.


For ad creators the "inital" phase is easy to program and accommodate. The loading process of the ad starts, so the ad can do a limited amount of steps before it has to stop and needs to wait for the "subload" phase.

To correctly anticipate the "subload" phase however, might prove more difficult.  Most ads are "locked" inside a SafeFrame (an iframe with no way to access the site), or nested deep inside of iframe-chains with different solutions and scenarios on each website.

To provide an easy solution for ad creators and publisher alike the OVK developed a lightweight and fast solution, which can be inserted into any creative and correctly determines when to start the subload, no matter on which website the creative is served.

Based on JavaScript, the solution can be pasted into the JavaScript code of the creative and exposes a new "Event" called "iabSubLoadStart". Any steps the creative should take during "subload", can be added via an EventListener on the "window" object of the creative.<br><br>
 
