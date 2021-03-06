sno+ monitoring web app design notes
====================================
Data
----
### Dispatch
* PMT data
* MTC data
* CAEN data
* Run/event headers
* CMOS rates
* HV
* LV

### Slow control
* Rack HV
* Rack LV
* Compensation coil status
* XL3 resets
* Interlocks

### Calibration
* SMELLIE laser configuration
* ...

### (New) SNOOP
* Low-level analysis data: rates, nu flux, ...
* Data quality

### CMA
* Alarms

### Webcams
* webcams...

### ORCADB
* Detector configuration
* Run configuration: calibration sources, etc.
* Run headers

### Data flow
* Data rates (crates, deck, beyond)
* Queue status (FEC FIFO, ORCA, builder)
* DB status
* Orca and builder log tail?

Server
------
* python socketio + gevent wsgi
* orcaroot, etc. -> unpack -> queue or event -> socketio namespaces --subfilter--> client
* server side is a stack of fixed-length queues + most recent sample for each queue
  * queue elements represent a time slice
  * incoming data is binned in time and appended to queue
  * one queue for every piece of data coming in (tens of thousands)

* does not replace logging, but keeps a shallow buffer of time-binned historical data where relevant
  * queue data: actual queue, most recent value, length, time per bin, hold timeout (keep constant/drop to zero)

* some data does not need to be buffered: pmt data comes in, emitted to namespaces as event, and they send it or drop it depending on the client event display filter settings.

example: there is a queue for cmos rate of each channel. when a cmos rate record comes off the data stream, it gets put into the current queue element.

Client
------
* There are many pages, each of which is a special view of a detector subsystem. Some visual elements (a crate, the set of crates, ...) may be shared and used to visualize different data.
* socketio-client js "subscribes" to certain queues at page load. these structures are populated with the same server queue, and after setup the server just sends new points.
* backbone sees changes in the client-side queues and updates dom elements accordingly
* logins with different priveleges: e.g. admin can reset xl3s with slow controls, users view only.

Pages:
  * Overview: detector status at a glance, shift start
  * Alarms: CMA, SNOOP, and other alarms
  * CMOS Rates: detector overview (good/bad per crate), click for crate details with numbers, click channel for stripchart. user set alarm level in terms of screamer threshold and count
  * Base currents: same as cmos
  * Event display (websnoed): 3D detector, histograms, crates, caen traces, control (sum, clear, stop, type filter)
  * Data flow: "reactor" flowchart from crates > orca > builder > dispatch and dflow, showing rates, buffer status, and log tails
  * Detector configuration: static, current detector config from orca

* Persistent run status indicator and toaster alarms across all pages

