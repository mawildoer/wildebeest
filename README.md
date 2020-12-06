# Wildebeest

The high-level project goals are:
 - Develop a machine controller focussed on pre-planned tightly-synchronised multi-axis motion
 - Focus on high-power servo motors suited for CNC machines, but also applicable to high-performance 3D printers (and perhaps the next generation of super-light direct-drive extruders)
 - Create a control and driver board providing a similar level of "plug-n-play" functionality as a smoothieboard or duet do for 3D printers
 - Make the system expandable and future-proof, while maintaining a low barrier to entry for other developers to launch into

In more tangible terms and for illustration's sake, I intend to slap 6 (5 axes + spindle) servo controllers and a raspberry pi onto a board, glue it all together with CAN and use it as a CNC machine controller and keep the cost below what you'd normally pay for two "low-cost" servo drivers, without all the added fuss, looming and know-how required to string a normal control cabinet together. 
If this interests you, welcome aboard! :)

## Intended control scheme

A big key to this project is to remove the central real-time controller. I was partially inspired into this thought by the [Klipper project](https://github.com/KevinOConnor/klipper) around the start of 2019. I have not yet decided whether I will recycle some of it's main components.

 - Path planning using a more powerful machine and high-level language (python) and not in real-time, which means it can be done under linux
 - These paths (represented as polynomials) will be transmitted to the CAN network using UAVCAN
 - Each axes controller collects it's instructions fromt the bus and follows them closely as possible with it's motor. 
 - Time will be synchronised across devices using [UAVCAN's existing scheme](https://github.com/UAVCAN/public_regulated_data_types/tree/master/uavcan/time)
 - Devices are assigned a node ID based on their position on the board; for example by using a divider on an analog pin which acts as startup configuration. This means the same software can be deployed to all axes
 - Other functions (GPIO, temperature etc...) is all transmitted to the same CAN bus.
   - Configuration describes what each function ("stop") should be bound to ("limit switch x+"). 
    - There's no limit to how many functions can be bound to a piece of data; a probe input should be bound to "stop" on all motion axes. 
	- There's only a practical limit (e.g. array size) to how many data bindings can be made for a given function; "limit switch x+", "probe" and "control panel stop" can all be bound to each servo's "stop" function
	- These bindings are interpereted by a pre-processor, using intelligble names for the data bindings and functions, but boiled down to discrete bounded configuration along the lines of "axis 1, stop when: limit switch x is high, probe is high or stop input is high. axis 1, reset position to 0 when: limit switch x+ is high" ect...
	- The above node assignment means that devices can listen for the configuration relevant to them
	
## Hardware

### Development Hardware

I am starting work focussed on the servo driver firmware. For this work I am using a pair of [B-G431B-ESC1 development kit](https://www.st.com/en/evaluation-tools/b-g431b-esc1.html) from ST micro. To control them, I am using a [SeeedStudio 2 Channel CAN BUS FD Shield for Raspberry Pi](https://wiki.seeedstudio.com/2-Channel-CAN-BUS-FD-Shield-for-Raspberry-Pi/) and a raspberry pi 4. If you wish to come aboard, I recommend this hardware, so we begin on common ground. We can collectively transition to alternate hardware once we've designed it. 

### Current state of integrated hardware

In my opinion, there are two aspects precluding the wider use of servos in low-mid-range CNC machines, 3D printers, plasma cutter etc...; their cost and their barrier to entry. That means, a critical element to the success of this project must be a low cost platform. For that reason, I have listed the large and exensive components anticipated for use on this controller along with the current components of interest below. 

First however, the hardware design decisions:
 - ST microcontrollers will be used for the servo drivers as they provide a low barrier to entry motor control suite and libraries. I do not intend to develop and maintain this firmware for multiple platforms without good reason. 
 - A raspberry pi 4 will be the primary computer. Again, this choice is because they're ubiquitous, low barrier to entry, low cost, well supported and consistently high quality.
 - Salvaged ex-industrial servo motors should be compatible with this controller. They are high-performance, reliable, low cost and easy to procure. This means the power electronics of this system should be suitible for at least 400v (339v of rectified Aussie single phase, plus some overhead). This also comes with the not-to-be-overlooked benifit that it removes the nessecity for an expensive high-current power supply unit, with the downside that it's kinda dangerous. For me, this is great, but this specific design choice really doesn't lock us in too much, as only the output stage of the board will need effort for an update to a lower voltage system.
 
 
Major components + prices (euro) at 1k unit:
 - Raspberry Pi: €30
 - Power module (400w): €4 x 5 = 20 (example IM231-L6T2B)
 - High power module (6.4kW): €8 x 1 = 8 (example IFCM20U65GDXKMA1)
 - Servo controller + GPIO module: €2.50 x 7 = 17.50 (example STM32G431CBU6)
 - CAN Transceiver: €0.50 x 8 = 4 (MCP2558FD)
 - Total = €79.50
 
There's a lot missing above (the capacitors for example will add up quickly), but even if we double this cost to €180, it remains well within my budget expectations for a machine build.

