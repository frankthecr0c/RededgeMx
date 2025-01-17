﻿<article class="markdown-body">
         <h1 id="rededge-serial-api">RedEdge Serial API</h1>
         <p>This document describes the protocol which can be be used by software to control the RedEdge multi-spectral camera. The
         camera can be controlled using serial via the host connector. For more detailed information the physical connection to the
         camera, see the RedEdge Integration Guide on the Downloads page of <a href="https://atlas.micasense.com/">ATLAS</a>.</p>
         <h1 id="table-of-contents">Table of Contents</h1>
         <ul>
            <li>
               <a href="#rededge-serial-api">RedEdge Serial API</a>
            </li>
            <li>
               <a href="#table-of-contents">Table of Contents</a>
            </li>
            <li>
               <a href="#general-protocol-overview">General Protocol Overview</a>
               <ul>
                  <li>
                     <a href="#communicating">Communicating</a>
                  </li>
                  <li>
                     <a href="#gps">GPS</a>
                  </li>
                  <li>
                     <a href="#commanding-captures">Commanding Captures</a>
                  </li>
               </ul>
            </li>
            <li>
               <a href="#parameter_microservice">Parameter Microservice</a>
               <ul>
                  <li>
                     <a href="#supported_parameters">Supported Parameters</a>
                  </li>
               </ul>
            </li>
            <li>
               <a href="#messages">Messages</a>
               <ul>
                  <li>
                     <a href="#heartbeat">HEARTBEAT</a>
                  </li>
                  <li>
                     <a href="#ping">PING</a>
                  </li>
                  <li>
                     <a href="#gps_raw_int">GPS_RAW_INT</a>
                  </li>
                  <li>
                     <a href="#system_time">SYSTEM_TIME</a>
                  </li>
                  <li>
                     <a href="#attitude">ATTITUDE</a>
                  </li>
                  <li>
                     <a href="#mount_status">MOUNT_STATUS</a>
                  </li>
                  <li>
                     <a href="#global_position_int">GLOBAL_POSITION_INT</a>
                  </li>
                  <li>
                     <a href="#gps_status">GPS_STATUS</a>
                  </li>
                  <li>
                     <a href="#digicam_configure">DIGICAM_CONFIGURE</a>
                  </li>
                  <li>
                     <a href="#digicam_control">DIGICAM_CONTROL</a>
                  </li>
                  <li>
                     <a href="#camera_status">CAMERA_STATUS</a>
                  </li>
                  <li>
                     <a href="#camera_feedback">CAMERA_FEEDBACK</a>
                  </li>
                  <li>
                     <a href="#param_request_read">PARAM_REQUEST_READ</a>
                  </li>
                  <li>
                     <a href="#param_request_list">PARAM_REQUEST_LIST</a>
                  </li>
                  <li>
                     <a href="#param_value">PARAM_VALUE</a>
                  </li>
                  <li>
                     <a href="#param_set">PARAM_SET</a>
                  </li>
               </ul>
            </li>
         </ul>
         <h1 id="general-protocol-overview">General Protocol Overview</h1>
         <p>The API is accessed via serial messages in the Mavlink format (see 
         <a href="http://en.wikipedia.org/wiki/MAVLink">http://en.wikipedia.org/wiki/MAVLink</a> and 
         <a href="https://mavlink.io/en/about/overview.html">https://mavlink.io/en/about/overview.html</a>). Mavlink provides an
         open data format for interaction as well as a suite of tools to assist the programmer in developing and testing the
         interface. RedEdge uses Mavlink v1.0 messages and communicates with the host at 57600 baud.</p>
         <h2 id="communicating">Communicating</h2>
         <p>The <a href="#heartbeat">HEARTBEAT</a> message can ensure that the RedEdge is booted and ready to receive MAVLink packets.</p>
         <p>The <a href="#ping">PING</a> message can be used to ensure round-trip serial connectivity with the RedEdge.</p>
         <p>MAVLink messages include a sender <em>System ID</em> (sysid) parameter. The RedEdge will adopt the System ID
         of any message it receives, and defaults to 255 otherwise. Because of this behavior, it also ignores the
         Target System field, and instead assumes that any target system filtering is handled by the sender.</p>
         <p>Many MAVLink messages include a <em>Target Component</em> parameter. The RedEdge will ignore any command that does not
         specify MAV_COMP_ID_CAMERA or MAV_COMP_ID_ALL as the Target Component.</p>
         
  <h2 id="gps">GPS</h2>
         <p>Receipt of serial state messages will override any RedEdge internal or external sensor data, such as a connected GPS,
         for 5 seconds since the last state message was received. For aircraft state and mount status, messages should be sent as
         often as possible to ensure the latest state information is available to the camera when a capture is taken.</p>
         <p>There are two supported ways to provide GPS data to the RedEdge. The simplest method is to send the <a href="#gps_raw_int">
         GPS_RAW_INT</a> message, as this single message contains both the time and position information. The other supported way is to
         send both <a href="#system_time">SYSTEM_TIME</a> and <a href="#global_position_int">GLOBAL_POSITION_INT</a> together. Sending
         GPS in both styles at the same time is not recommended.</p>
         
   <h2 id="commanding-captures">Commanding Captures</h2>
         <p>The <a href="#digicam_configure">DIGICAM_CONFIGURE</a> message can be used to set up the camera imagers</p>
         <p>The <a href="#digicam_control">DIGICAM_CONTROL</a> message will command a capture, and the camera will send a
         <a href="#camera_status">CAMERA_STATUS</a> and a <a href="#camera_feedback">CAMERA_FEEDBACK</a> message in response</p>
         
   <h1 id="parameter_microservice">Parameter Microservice</h1>
         <p>The parameter microservice implements the <a href="https://mavlink.io/en/services/parameter.html">MAVLink
            Parameter Protocol</a> to read or change camera configuration settings.</p>
         <p>There are 4 messages associated with the parameter microservice, which are detailed in the Messages section below:
            <a href="#param_request_read">PARAM_REQUEST_READ</a>,
            <a href="#param_request_list">PARAM_REQUEST_LIST</a>,
            <a href="#param_value">PARAM_VALUE</a>,
            <a href="#param_set">PARAM_SET</a></p>
         
   <p id="mav_param_type">The enumeration and description of each MAV_PARAM_TYPE is listed below</p>
         <table>
            <tbody><tr>
               <th>Value</th>
               <th>Field Name</th>
               <th>Description</th>
            </tr>
            <tr>
               <td>1</td>
               <td>MAV_PARAM_TYPE_UINT8</td>
               <td>8-bit unsigned integer</td>
            </tr>
            <tr>
               <td>2</td>
               <td>MAV_PARAM_TYPE_INT8</td>
               <td>8-bit signed integer</td>
            </tr>
            <tr>
               <td>3</td>
               <td>MAV_PARAM_TYPE_UINT16</td>
               <td>16-bit unsigned integer</td>
            </tr>
            <tr>
               <td>4</td>
               <td>MAV_PARAM_TYPE_INT16</td>
               <td>16-bit signed integer</td>
            </tr>
            <tr>
               <td>5</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>32-bit unsigned integer</td>
            </tr>
            <tr>
               <td>6</td>
               <td>MAV_PARAM_TYPE_INT32</td>
               <td>32-bit signed integer</td>
            </tr>
            <tr>
               <td>7</td>
               <td>MAV_PARAM_TYPE_UINT64</td>
               <td>64-bit unsigned integer</td>
            </tr>
            <tr>
               <td>8</td>
               <td>MAV_PARAM_TYPE_INT64</td>
               <td>64-bit signed integer</td>
            </tr>
            <tr>
               <td>9</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>32-bit floating point</td>
            </tr>
            <tr>
               <td>10</td>
               <td>MAV_PARAM_TYPE_REAL64</td>
               <td>64-bit floating point</td>
            </tr>
         </tbody></table>
         <h2 id="supported_parameters">Supported Parameters</h2>
         <table>
            <tbody><tr>
               <th>Param ID</th>
               <th>Param Type</th>
               <th>Description</th>
               <th>Versions</th>
            </tr>
            <tr>
               <td>wifi</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Set to 1.0 to turn WiFi module on</li>
                     <li>Set to 0.0 to turn WiFi module off</li>
                  </ul>
               </td>
               <td>v1.5.30-</td>
            </tr>
            <tr>
               <td>rawFormat</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Selects what raw output format the camera saves as</li>
                     <ul>
                        <li>Set to 0 for TIFF</li>
                        <li>Set to 1 for DNG</li>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>enabledBandsRaw</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Bit mask of bands that are enabled for raw capture storage</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>enabledBandsJpeg</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Bit mask of bands that are enabled for jpeg capture and storage</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>autoCapMode</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Auto capture mode</li>
                     <ul>
                        <li>Set to 0 for Disabled</li>
                        <li>Set to 1 for Timer</li>
                        <li>Set to 2 for Overlap</li>
                        <li>Set to 3 for External Trigger</li>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>timerPeriod</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Period between timer captures (seconds)</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>overlapAlongTrk</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Desired along track overlap percentage (0-100)</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>overlapCrossTrk</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Desired cross track overlap percentage (0-100)</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>pwmTriggerThresh</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>The long/short pulse width threshold for PWM input, in milliseconds</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>extTriggerMode</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>When set to 1, top of frame pulses will be generated on a camera-specific pin.</li>
                     <ul>
                        <li>Note that activating this feature may cause damage to the camera if the pin is also driven by an external device</li>
                        </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>extTrgrOutPulseH</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>When set to 1.0, top of frame pulses will be generated by resting low, with the rising edge of the pulse indicating the top of frame of a capture</li>
                     <li>When set to 0.0, the top of frame pulse will be generated by resting high, with the falling edge of the pulse indicating the top of frame of a capture</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>agcMinimumMean</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Sets the minimum normalized mean value for each image during Automatic Gain Control (AGC)</li>
                     <li>Valid values are from 0.0 to 1.0, inclusive</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>gainExpCrossover</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Exposure time (in seconds) at which the gain should be increased in order to properly expose rather than increasing the exposure time further</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>operatingAlt</td>
               <td>MAV_PARAM_TYPE_INT32</td>
               <td>
                  <ul>
                     <li>Aircraft operating altitude above ground (meters)</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>opAltTolerance</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Distance, in meters, below the operating altitude (operatingAlt) to enable automatic capture modes</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>audioEnable</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>When set to 1.0, the camera is allowed to produce sounds</li>
                     <ul>
                        <li>Note that not all camera models have a method of producing sound, in which case this field is not a functional parameter</li>
                     </ul>
                     <li>When set to 0.0, the camera will not make any sounds</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>audioSelBitfield</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Bitfield that masks individual audio features. Setting a bit to 0 will disable the individual associated audio feature
                        <br>
                        <table>
                          <thead>
                             <tr>
                                <th>Bit</th>
                                <th>Feature</th>
                             </tr>
                          </thead>
                          <tbody>
                             <tr>
                                <td>0</td>
                                <td>Tone when a normal capture is taken (anti_sat=false)</td>
                             </tr>
                             <tr>
                                <td>1</td>
                                <td>Tone when a panel capture is taken (anti_sat=true)</td>
                             </tr>
                             <tr>
                                <td>2</td>
                                <td>Tone when a panel detection is active</td>
                             </tr>
                          </tbody>
                        </table>
                     </li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>injectedGpsDelay</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Sets the fixed expected delay in seconds between the timestamp contained in the GPS source 
                     (whether that be from a directly connected GPS, DLS, or injected via one of the APIs), and the time the message is received</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>ipAddress</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Sets the Ethernet IP address of the camera</li>
                     <li>The input is divided into four 8-bit sections (part1.part2.part3.part4) and interpreted as follows:</li>
                     <ul>
                        <li>part1 = (ipAddress &gt;&gt; 24) &amp; 0xFF</li>
                        <li>part2 = (ipAddress &gt;&gt; 16) &amp; 0xFF</li>
                        <li>part3 = (ipAddress &gt;&gt; 8) &amp; 0xFF</li>
                        <li>part4 = (ipAddress &gt;&gt; 0) &amp; 0xFF</li>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>pinModes</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Sets the pin modes of the camera</li>
                     <li>The input is divided into two 16-bit sections and interpreted as follows:</li>
                     <ul>
                        <li>pin number = (pinModes &gt;&gt; 16) &amp; 0xFFFF</li>
                        <li>pin mode = (pinModes &gt;&gt; 0) &amp; 0xFFFF</li>
                     </ul>
                     <li>Each time this value is read, it will cycle through the pins,
                        i.e., if there are 3 configurable pins, 3 separate calls will need to be made to see all pin information</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>networkMode</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Selects whether the camera will act as a main camera or auxiliary camera in a MicaSense network</li>
                     <ul>
                        <li>Set to 0 for Main</li>
                        <li>Set to 1 for Auxiliary</li>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>thermalNucTime</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Set to 1.0 to command a NUC</li>
                     <ul>
                        <li>This will cause automatic NUCing to become disabled and all further NUCs will need to be commanded until the camera has been power cycled</li>
                     </ul>
                     <li>Returns the number of seconds since the last NUC</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>thermalNucTemp</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Set to 1.0 to command a NUC</li>
                     <ul>
                        <li>This will cause automatic NUCing to become disabled and all further NUCs will need to be commanded until the camera has been power cycled</li>
                     </ul>
                     <li>Returns the change in temperature (measured in K) since the last NUC</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
         </tbody></table>
         <p><b>WARNING: </b>The parameters below, all prefixed with a "~", are depreciated
            and may not be supported in future camera software versions. They are read-only, and any
            PARAM_SET calls will have no effect on the returned value or operation of the camera.</p>
         <table>
            <tbody><tr>
               <th>Param ID</th>
               <th>Param Type</th>
               <th>Description</th>
               <th>Versions</th>
            </tr>
            <tr>
               <td>~sdGbFree</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>The amount of free space on the SD card (GB)</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~sdGbTotal</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>The total amount of storage on the SD card (GB)</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~sdWarn</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>When the value returned is 1.0, indicates the low SD space warning threshold is met (Under 10GB remaining) or using unrecommended SD filesystem type</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~sdStatus</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Status of the SD card</li>
                     <ul>
                        <li>Returns 0 when SD card is not present</li>
                        <li>Returns 1 when SD card is full</li>
                        <li>Returns 2 when SD card is detected and has free space</li>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~busVolts</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Returns the measured supply voltage</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~autoCapActive</td>
               <td>MAV_PARAM_TYPE_REAL32</td>
               <td>
                  <ul>
                     <li>Returns 1.0 when the camera's automatic capture mode is activated</li>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~dlsStatus</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>Returns the current status of the DLS:</li>
                     <ul>
                        <li>Returns 0 when DLS status is 'Not Present'</li>
                        <li>Returns 1 when DLS status is 'OK'</li>
                        <li>Returns 2 when DLS status is 'Programming'</li>
                        <li>Returns 3 when DLS status is 'Error'</li>
                        <li>Returns 4 when DLS status is 'Initializing'</li>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~timeSource</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>What type of time source is being used by the camera:</li>
                     <ul>
                        <li>Returns 0 for None</li>
                        <li>Returns 1 for GPS</li>
                        <li>Returns 2 for PPS</li>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~serialFirstFour</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>The first four values of the camera's serial number</li>
                     <ul>
                        <li>The serial number can be interpreted from the returned value by evaluating each 8-bit section as an individual ASCII character.</li>
                        <li>EX: If the serial is 'AL03'</li>
                        <ul>
                           <li>(serialfirstFour &gt;&gt; 24) &amp; 0xFF = A</li>
                           <li>(serialfirstFour &gt;&gt; 16) &amp; 0xFF = L</li>
                           <li>(serialfirstFour &gt;&gt; 8) &amp; 0xFF = 0</li>
                           <li>(serialfirstFour &gt;&gt; 0) &amp; 0xFF = 3</li>
                        </ul>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
            <tr>
               <td>~swVersion</td>
               <td>MAV_PARAM_TYPE_UINT32</td>
               <td>
                  <ul>
                     <li>The camera's software version</li>
                     <ul>
                        <li>The software version can be interpreted from the returned value by evaluating each 8-bit section as an individual uint8 value</li>
                        <li>EX: If the software version is 'v1.2.3'</li>
                        <ul>
                           <li>(swVersion &gt;&gt; 0) &amp; 0xFF = 1</li>
                           <li>(swVersion &gt;&gt; 8) &amp; 0xFF = 2</li>
                           <li>(swVersion &gt;&gt; 16) &amp; 0xFF = 3</li>
                        </ul>
                     </ul>
                  </ul>
               </td>
               <td>v7.1.0-</td>
            </tr>
         </tbody></table>

 <h1 id="messages">Messages</h1>
         
   <h2 id="heartbeat">HEARTBEAT (0)</h2>
         <p><strong>Versions</strong>: v1.3.2-</p>
         <p>The RedEdge will send a heartbeat message approximately once per second using the component ID MAV_COMP_ID_CAMERA.
         While this message can be safely ignored, it can be used to verify the RedEdge is properly powered on.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#HEARTBEAT">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Type</td>
               <td>uint8_t</td>
               <td>Type of the MAV (quadrotor, helicopter, etc., up to 15 types, defined in MAV_TYPE ENUM)</td>
               <td>Sent as 0 (ignored)</td>
            </tr>
            <tr>
               <td>Autopilot</td>
               <td>uint8_t</td>
               <td>
                  <ol>
                     <li value="0">MAV_AUTOPILOT_GENERIC - Generic autopilot, full support for everything</li>
                     <li>MAV_AUTOPILOT_RESERVED - Reserved for future use.</li>
                     <li>MAV_AUTOPILOT_SLUGS - SLUGS autopilot, http://slugsuav.soe.ucsc.edu</li>
                     <li>MAV_AUTOPILOT_ARDUPILOTMEGA - ArduPilotMega / ArduCopter, http://diydrones.com</li>
                     <li>MAV_AUTOPILOT_OPENPILOT - OpenPilot, http://openpilot.org</li>
                     <li>MAV_AUTOPILOT_GENERIC_WAYPOINTS_ONLY - Generic autopilot only supporting simple waypoints</li>
                     <li>MAV_AUTOPILOT_GENERIC_WAYPOINTS_AND_SIMPLE_NAVIGATION_ONLY - Generic autopilot supporting waypoints and other simple navigation commands</li>
                     <li>MAV_AUTOPILOT_GENERIC_MISSION_FULL - Generic autopilot supporting the full mission command set</li>
                     <li>MAV_AUTOPILOT_INVALID - No valid autopilot, e.g. a GCS or other MAVLink component</li>
                     <li>MAV_AUTOPILOT_PPZ - PPZ UAV - http://nongnu.org/paparazzi</li>
                     <li>MAV_AUTOPILOT_UDB - UAV Dev Board</li>
                     <li>MAV_AUTOPILOT_FP - FlexiPilot</li>
                     <li>MAV_AUTOPILOT_PX4 - PX4 Autopilot - http://pixhawk.ethz.ch/px4/</li>
                     <li>MAV_AUTOPILOT_SMACCMPILOT - SMACCMPilot - http://smaccmpilot.org</li>
                     <li>MAV_AUTOPILOT_AUTOQUAD - AutoQuad -- http://autoquad.org</li>
                     <li>MAV_AUTOPILOT_ARMAZILA - Armazila -- http://armazila.com</li>
                     <li>MAV_AUTOPILOT_AEROB - Aerob -- http://aerob.ru</li>
                     <li>MAV_AUTOPILOT_ASLUAV - ASLUAV autopilot -- http://www.asl.ethz.ch</li>
                  </ol>
               </td>
               <td>Sent as 1 (ignored)</td>
            </tr>
            <tr>
               <td>Base Mode</td>
               <td>uint8_t</td>
               <td>
                  <ol>
                     <li value="128">MAV_MODE_FLAG_SAFETY_ARMED - 0b10000000 MAV safety set to armed. Motors are enabled / running / can start. Ready to fly.</li>
                     <li value="64">MAV_MODE_FLAG_MANUAL_INPUT_ENABLED - 0b01000000 remote control input is enabled.</li>
                     <li value="32">MAV_MODE_FLAG_HIL_ENABLED - 0b00100000 hardware in the loop simulation. All motors / actuators are blocked, but internal software is full operational.</li>
                     <li value="16">MAV_MODE_FLAG_STABILIZE_ENABLED - 0b00010000 system stabilizes electronically its attitude (and optionally position). It needs however further control inputs to move around.</li>
                     <li value="8">MAV_MODE_FLAG_GUIDED_ENABLED - 0b00001000 guided mode enabled, system flies MISSIONs / mission items.</li>
                     <li value="4">MAV_MODE_FLAG_AUTO_ENABLED - 0b00000100 autonomous mode enabled, system finds its own goal positions. Guided flag can be set or not, depends on the actual implementation.</li>
                     <li value="2">MAV_MODE_FLAG_TEST_ENABLED - 0b00000010 system has a test mode enabled. This flag is intended for temporary system tests and should not be used for stable implementations.</li>
                     <li value="1">MAV_MODE_FLAG_CUSTOM_MODE_ENABLED - 0b00000001 Reserved for future use.</li>
                  </ol>
               </td>
               <td>Sent as 2 (ignored)</td>
            </tr>
            <tr>
               <td>Custom Mode</td>
               <td>uint32_t</td>
               <td>A bitfield for use for autopilot-specific flags.</td>
               <td>Sent as 3 (ignored)</td>
            </tr>
            <tr>
               <td>System Status</td>
               <td>uint8_t</td>
               <td>
                  <ol>
                     <li value="0">MAV_STATE_UNINIT - Uninitialized system, state is unknown.</li>
                     <li>MAV_STATE_BOOT - System is booting up.</li>
                     <li>MAV_STATE_CALIBRATING - System is calibrating and not flight-ready.</li>
                     <li>MAV_STATE_STANDBY - System is grounded and on standby. It can be launched any time.</li>
                     <li>MAV_STATE_ACTIVE - System is active and might be already airborne. Motors are engaged.</li>
                     <li>MAV_STATE_CRITICAL - System is in a non-normal flight mode. It can however still navigate.</li>
                     <li>MAV_STATE_EMERGENCY - System is in a non-normal flight mode. It lost control over parts or over the whole airframe. It is in mayday and going down.</li>
                     <li>MAV_STATE_POWEROFF - System just initialized its power-down sequence, will shut down now.</li>
                  </ol>
               </td>
               <td>Sent as 4 (ignored)</td>
            </tr>
            <tr>
               <td>MAVLink Version</td>
               <td>uint8_t</td>
               <td>MAVLink version, not writable by user</td>
               <td>Same as official</td>
            </tr>
         </tbody></table>
         
   <h2 id="ping">PING (4)</h2>
         <p><strong>Versions</strong>: v1.3.2-</p>
         <p>Send the PING message with system_id = 0 and component_id = 0, and with a new sequence number each time, to ensure
         round-trip communications with the RedEdge. The PING message will be responded to with the same sequence number and the
         system_id and component_id of the sender (which are present in the message header).</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#PING">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Time Unix</td>
               <td>uint64_t</td>
               <td>Timestamp (microseconds since UNIX epoch or microseconds since system boot)</td>
               <td>Ignored</td>
            </tr>
            <tr>
               <td>Sequence</td>
               <td>uint32_t</td>
               <td>PING sequence</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>System ID</td>
               <td>uint8_t</td>
               <td>0: request ping from all receiving systems, if greater than 0: message is a ping response and number is the system id of the requesting system</td>
               <td>See description above</td>
            </tr>
            <tr>
               <td>Component ID</td>
               <td>uint8_t</td>
               <td>0: request ping from all receiving components, if greater than 0: message is a ping response and number is the system id of the requesting system</td>
               <td>See description above</td>
            </tr>
         </tbody></table>
         
  <h2 id="gps_raw_int">GPS_RAW_INT (24)</h2>
         <p><strong>Versions</strong>: v1.3.2-</p>
         <p>Use this message to send the raw GPS information to the RedEdge as received by the host vehicle. This is the main
         message providing GPS information to the RedEdge. Forwarding GPS data to the camera as soon as it is received by the host
         will ensure the latest information is present in a new capture. Use the 64-bit time_usec field to set the camera's UTC
         time.</p>
         <p>UNIX time is separate from GPS time by a fixed base epoch offset and a changing number of leap seconds (a value that
         most GPS receivers provide). The base offset is +315964800 seconds between UNIX time and GPS time (UNIX is referenced to
         1970-Jan-1 while GPS is referenced to 1980-Jan-6).</p>
         <p>Most GPS receivers provide a UTC time message, but it may or may not be microseconds referenced to the Unix epoch,
         which is what we want. However, most of them provide GPS week number (WN) plus time of week (TOW), plus number of leap
         seconds (LS), which makes the math easier for embedded processors.</p>
         <p>With those GPS time inputs, UNIX epoch UTC times can be calculated by:</p>
<pre>Base offset + (GPS Week Number * Seconds in week) + GPS Time of Week - Leap Seconds.
  315964800 + (      WN        *      604800    ) +       TOW        -      LS</pre>
         <p>As of 2019 May 05, LS equals 18.</p>
         <p>Depending on the receiver/formatting/etc. it may be required to also add in a nanoseconds term to get sub-second resolution.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#GPS_RAW_INT">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Time Unix</td>
               <td>uint64_t</td>
               <td>Timestamp (microseconds since UNIX epoch or microseconds since system boot)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Fix Type</td>
               <td>uint8_t</td>
               <td>
                  <ol>
                     <li value="0">No fix</li>
                     <li>No fix</li>
                     <li>2D fix</li>
                     <li>3D fix</li>
                     <li>DGPS</li>
                     <li>RTK</li>
                  </ol>
               </td>
               <td>
                  &lt;= 2: no fix
                  <br>&gt; 2: 3D fix
               </td>
            </tr>
            <tr>
               <td>Latitude</td>
               <td>int32_t</td>
               <td>Latitude (WGS84), in degrees * 1E7</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Longitude</td>
               <td>int32_t</td>
               <td>Longitude (WGS84), in degrees * 1E7</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Altitude (MSL)</td>
               <td>int32_t</td>
               <td>Altitude (AMSL, NOT WGS84), in meters * 1000 (positive for up).</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Estimated Position Horizontal</td>
               <td>uint16_t</td>
               <td>GPS HDOP horizontal dilution of position in cm (m*100). If unknown, set to: UINT16_MAX</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Estimated Position Vertical</td>
               <td>uint16_t</td>
               <td>GPS VDOP vertical dilution of position in cm (m*100). If unknown, set to: UINT16_MAX</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Speed</td>
               <td>uint16_t</td>
               <td>GPS ground speed (m/s * 100). If unknown, set to: UINT16_MAX</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Course Over Ground</td>
               <td>uint16_t</td>
               <td>Course over ground (NOT heading, but direction of movement) in degrees * 100, 0.0..359.99 degrees.
                     If unknown, set to: UINT16_MAX</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Satellites Visible</td>
               <td>uint8_t</td>
               <td>Number of satellites visible. If unknown, set to 255</td>
               <td>Same as official</td>
            </tr>
         </tbody></table>
         
 <h2 id="system_time">SYSTEM_TIME (2)</h2>
         <p><strong>Versions</strong>: v1.3.2-</p>
         <p>This message is required if using GLOBAL_POSITION_INT, as that message does not provide a UTC timestamp. It is used to
         update UTC time to provide accurate timestamping of images. Provide a 64-bit unix timestamp referenced to UTC time and the
         UNIX (not GPS) epoch. Most GPS receivers provide this information in a UTC time message.</p>
         <p>Note: if using GPS_RAW_INT, the 64bit timestamp should be set to UTC time prior to sending.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#SYSTEM_TIME">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Time Unix</td>
               <td>uint64_t</td>
               <td>Timestamp of the master clock in microseconds since UNIX epoch.</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Time Boot</td>
               <td>uint32_t</td>
               <td>Timestamp of the component clock since boot time in milliseconds.</td>
               <td>Ignored</td>
            </tr>
         </tbody></table>
         
<h2 id="attitude">ATTITUDE (30)</h2>
         <p><strong>Versions</strong>: v1.3.2-</p>
         <p>Vehicle state (roll, pitch, yaw) and vehicle rates can be provided via the ATTITUDE message. Note that this is the
         state of the host vehicle. The offset of the camera from the host reference frame, whether fixed or on a gimbals, should
         be provided via the MOUNT_STATUS message defined below.</p>
         <p>The aircraft orientation is specified in the standard aeronautical coordinate frame, where the X axis points to the
         front of the vehicle, the Y axis points to the right, and the Z axis points down. Rotation order is Yaw, then Pitch, then
         Roll, about the Z, Y, then X axes accordingly.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#ATTITUDE">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Time Boot</td>
               <td>uint32_t</td>
               <td>Timestamp (milliseconds since system boot)</td>
               <td>Ignored</td>
            </tr>
            <tr>
               <td>Roll</td>
               <td>single precision float</td>
               <td>Roll angle (rad, -pi..+pi)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Pitch</td>
               <td>single precision float</td>
               <td>Pitch angle (rad, -pi..+pi)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Yaw</td>
               <td>single precision float</td>
               <td>Yaw angle (rad, -pi..+pi)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Roll Rate</td>
               <td>single precision float</td>
               <td>Roll angular velocity (rad/s)</td>
               <td>Ignored</td>
            </tr>
            <tr>
               <td>Pitch Rate</td>
               <td>single precision float</td>
               <td>Pitch angular velocity (rad/s)</td>
               <td>Ignored</td>
            </tr>
            <tr>
               <td>Yaw Rate</td>
               <td>single precision float</td>
               <td>Yaw angular velocity (rad/s)</td>
               <td>Ignored</td>
            </tr>
         </tbody></table>
         
<h2 id="mount_status">MOUNT_STATUS (158)</h2>
         <p><strong>Versions</strong>: v1.3.2-</p>
         <p>This message can be used to provide the orientation of the camera measured by an external source by sending the
         MOUNT_STATUS message along with the ATTITUDE message. Anytime this value has been written in the preceding 5 seconds,
         it will be considered valied and will be written to image metadata.</p>
         <p>Using these two messages, two rotations can be specified: An aircraft orientation and a camera orientation. The
         aircraft orientation gives the orientation of the aircraft relative to the earth frame, and the camera orientation gives
         the orentiation of the fixed or gimbaled camera relative to the aircraft. If the camera is fixed mounted, the camera
         orientation can be set to the appropriate static value and only the aircraft orientation needs to be updated.</p>
         <p>The camera orientation represents the orientation of the camera relative to the aircraft. The default orientation, when
         the camera angles are all zero, is defined such that the camera is pointed straight down relative to the aircraft
         (aircraft nadir) with the top of the camera towards the "front" of the aircraft. The rotation order is again yaw, then
         pitch, then roll, about the camera Z, Y, and X axes accordingly. The camera axes are defined such that at zero rotation
         angle, the X axis points down (along the camera focal axis) the Z axis points towards the rear of the vehicle, and the Y
         axis follows right-hand rule and points out the right side of the vehicle.</p>
         <p>The default value for these angles is zero. For a fixed camera pointed straight down relative to the aircraft, and with
         the top of the camera toward the front of the aircraft, this default value can be used (pitch = 0.0, roll = 0.0, yaw =
         0.0).</p>
         <p>The combination of the aircraft attitude sent in the attitude message and the rotations in the mount status message
         will be used to derive an earth-fixed camera orientation. It is recommended that when these messages are built on the host
         aircraft, the values for both the aircraft orientation and the gimbals orientation are latched into memory, then they are
         sent via these messages. This will ensure that the aircraft orientation and gimbals orientations were acquired at the same
         moment.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://github.com/mavlink/c_library/blob/master/ardupilotmega/mavlink_msg_mount_status.h">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>pointing_a</td>
               <td>int32_t</td>
               <td>Pitch (deg*100)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>pointing_b</td>
               <td>int32_t</td>
               <td>Roll (deg*100)</td>
               <td>Same as official</td>
            </tr><tr>
               <td>pointing_c</td>
               <td>int32_t</td>
               <td>Yaw (deg*100)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Target System</td>
               <td>uint8_t</td>
               <td>System ID</td>
               <td>Ignored</td>
            </tr>
            <tr>
               <td>Target Component</td>
               <td>uint8_t</td>
               <td>Component ID</td>
               <td>Same as official</td>
            </tr>
         </tbody></table>
         
<h2 id="global_position_int">GLOBAL_POSITION_INT (33)</h2>
         <p><strong>Versions</strong>: v1.3.2-</p>
         <p>Used to update the position from a host state estimate using more than raw GPS, such as a GPS-aided INS system. This
         may not be the position provided by the GPS receiver, but may be more accurate and/or updated more frequently. Since this
         message does not have a fix status field, sending it will imply a 3D GPS fix.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#GLOBAL_POSITION_INT">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Time Boot</td>
               <td>uint32_t</td>
               <td>Timestamp (milliseconds since system boot)</td>
               <td>Ignored</td>
            </tr>
            <tr>
               <td>Latitude</td>
               <td>int32_t</td>
               <td>Latitude, expressed as * 1E7</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Longitude</td>
               <td>int32_t</td>
               <td>Longitude, expressed as * 1E7</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Altitude (MSL)</td>
               <td>int32_t</td>
               <td>Altitude in meters, expressed as * 1000 (millimeters), AMSL (not WGS84 - note that virtually all GPS
                     modules provide the AMSL as well)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Altitude (Relative)</td>
               <td>int32_t</td>
               <td>Altitude above ground in meters, expressed as * 1000 (millimeters)</td>
               <td>Ignored</td>
            </tr>
            <tr>
               <td>Velocity X</td>
               <td>int16_t</td>
               <td>Ground X Speed (Latitude), expressed as m/s * 100</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Velocity Y</td>
               <td>int16_t</td>
               <td>Ground Y Speed (Longitude), expressed as m/s * 100</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Velocity Z</td>
               <td>int16_t</td>
               <td>Ground Z Speed (Altitude), expressed as m/s * 100</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Heading</td>
               <td>uint16_t</td>
               <td>Compass heading in degrees * 100, 0.0..359.99 degrees. If unknown, set to: UINT16_MAX</td>
               <td>Ignored</td>
            </tr>
         </tbody></table>
         
<h2 id="gps_status">GPS_STATUS (25)</h2>
         <p><strong>Versions</strong>: v1.3.2-</p>
         <p>GPS satellite information which is updated on the RedEdge WiFi user interface. This message is not required, but if sent
         the GPS status bars on the RedEdge WiFi interface will be updated.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#GPS_STATUS">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Satellites Visible</td>
               <td>uint8_t</td>
               <td>Number of satellites visible</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Satellite PRN</td>
               <td>uint8_t[20]</td>
               <td>Global satellite ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Satellite Used</td>
               <td>uint8_t[20]</td>
               <td>
                  <ol>
                     <li value="0">Satellite not used</li>
                     <li>Used for localization</li>
                  </ol>
               </td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Satellite Elevation</td>
               <td>uint8_t[20]</td>
               <td>Elevation of satellite, degrees. 0-90, where 0 is directly above the receiver, 90 is on the horizon</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Satellite Azimuth</td>
               <td>uint8_t[20]</td>
               <td>Direction of satellite, with 0-360 degrees scaled to the range 0-255</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Satellite SNR</td>
               <td>uint8_t[20]</td>
               <td>Signal to noise ratio of the satellite</td>
               <td>Same as official</td>
            </tr>
         </tbody></table>
         
<h2 id="digicam_configure">DIGICAM_CONFIGURE (154)</h2>
         <p><strong>Versions</strong>: v1.5.5-</p>
         <p>NOTE: At some point, MAVLink.io decided to add a
            <a href="https://mavlink.io/en/messages/common.html#MAV_CMD_DO_DIGICAM_CONFIGURE">MAV_CMD_DO_DIGICAM_CONFIGURE</a>,
            which, confusingly, does NOT have the same definition or 154 designation as the DIGICAM_CONFIGURE message.</p>
         <p>Manual and Automatic mode are global switches and impact all cameras. For this reason, if commanding each imager to
         different manual settings, ensure that no messages are sent with auto mode enabled or all imagers will switch to auto mode.</p>
         <p>The camera can set all of the imagers to manual mode in a single DIGICAM_CONFIGURE message by using the Extra Command
         value of 255. Alternatively, one DIGICAM_CONFIGURE message can be sent for each imager (numbers 0-4 for a stock camera) with an
         individual gain and exposure setting for each.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/ardupilotmega.html#DIGICAM_CONFIGURE">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Mode</td>
               <td>uint8_t</td>
               <td>
                  <ol>
                     <li>ProgramAuto</li>
                     <li>Aperture Priority</li>
                     <li>Shutter Priority</li>
                     <li>Manual</li>
                     <li>IntelligentAuto</li>
                     <li>SuperiorAuto</li>
                  </ol>
               </td>
               <td>
                  <ol>
                     <li value="1">Auto gain/exposure</li>
                     <li value="4">Manual</li>
                  </ol>
                  All other values have no effect
               </td>
            </tr>
            <tr>
               <td>Shutter Speed</td>
               <td>uint16_t</td>
               <td>Shutter speed (seconds divisor). For example, to specify 1/60sec, send 60</td>
               <td>As described in the official description<br>Ignored unless Mode set to Manual</td>
            </tr>
            <tr>
               <td>Aperture</td>
               <td>uint8_t</td>
               <td>Aperture: F stop number</td>
               <td>Set to 0 (ignored)</td>
            </tr>
            <tr>
               <td>ISO</td>
               <td>uint8_t</td>
               <td>Enumeration of ISO number e.g. 80, 100, 200, etc.</td>
               <td>
                  Gain:
                  <ol>
                     <li value="1">ISO100</li>
                     <li value="2">ISO200</li>
                     <li value="4">ISO400</li>
                     <li value="8">ISO800</li>
                  </ol>
                  Ignored unless Mode is set to Manual
               </td>
            </tr>
            <tr>
               <td>Exposure Mode</td>
               <td>uint8_t</td>
               <td>Exposure type enumerator</td>
               <td>Set to 0 (ignored)</td>
            </tr>
            <tr>
               <td>Command ID</td>
               <td>uint8_t</td>
               <td>Command Identity</td>
               <td>Set to 0 (ignored)</td>
            </tr>
            <tr>
               <td>Engine Cut-Off</td>
               <td>uint8_t</td>
               <td>Main engine cut-off time before camera trigger in seconds/10 (0 means no cut-off)</td>
               <td>This is for the aircraft, should be set to the value recommended by the manufacturer (ignored)</td>
            </tr>
            <tr>
               <td>Extra Command</td>
               <td>uint8_t</td>
               <td>Extra Command</td>
               <td>Sensor number (0-4) of which sensor to set manual settings. Set to 255 for all.</td>
            </tr>
            <tr>
               <td>Extra Value</td>
               <td>single precision float</td>
               <td>Extra Value</td>
               <td>Set to 0 (ignored)</td>
            </tr>
         </tbody></table>
         
<h2 id="digicam_control">DIGICAM_CONTROL (155)</h2>
         <p><strong>Versions</strong>: v1.5.5-</p>
         <p>NOTE: At some point, MAVLink.io decided to add a
            <a href="https://mavlink.io/en/messages/common.html#MAV_CMD_DO_DIGICAM_CONTROL">MAV_CMD_DO_DIGICAM_CONTROL</a>,
            which, confusingly, does NOT have the same definition or 155 designation as the DIGICAM_CONTROL message.</p>
         <p>The DIGICAM_CONTROL message is used to command captures. Only the "Shutter Command" field is currently used,
         but default values are provided for the other parameters for future API compatibility.</p>
         <p>All sensors configured to capture (e.g. via the WiFi /config route) will be captured.</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/ardupilotmega.html#DIGICAM_CONTROL">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>On/Off</td>
               <td>uint8_t</td>
               <td>Session control (on/off or show/hide lens):
                  <ol>
                     <li value="0">Turn off the camera / hide the lens</li>
                     <li>Turn on the camera /Show the lens</li>
                  </ol>
               </td>
               <td>Set to 1 (ignored)</td>
            </tr>
            <tr>
               <td>Zoom Position</td>
               <td>uint8_t</td>
               <td>Zoom's absolute position. 2x, 3x, 10x, etc.</td>
               <td>Set to 1 (ignored)</td>
            </tr>
            <tr>
               <td>Zoom Step</td>
               <td>int8_t</td>
               <td>Zooming step value to offset zoom from the current position</td>
               <td>Set to 0 (ignored)</td>
            </tr>
            <tr>
               <td>Focus Lock</td>
               <td>uint8_t</td>
               <td>Focus Locking, Unlocking or Re-locking:
                  <ol>
                     <li value="0">Ignore</li>
                     <li>Unlock</li>
                     <li>Lock</li>
                  </ol>
               </td>
               <td>Set to 0 (ignored)</td>
            </tr>
            <tr>
               <td>Shutter Command</td>
               <td>uint8_t</td>
               <td>Shooting command. Any non-zero value triggers the camera</td>
               <td>
                  <ol>
                     <li value="0">(or any number not defined below) Don't trigger</li>
                     <li>Trigger a flight capture (anti-sat mode disabled)</li>
                     <li>Trigger a panel capture (anti-sat mode enabled)</li>
                  </ol>
               </td>
            </tr>
            <tr>
               <td>CommandID</td>
               <td>uint8_t</td>
               <td>Command Identity</td>
               <td>Set to 0 (ignored)</td>
            </tr>
            <tr>
               <td>Extra Command</td>
               <td>uint8_t</td>
               <td>Extra Command</td>
               <td>Set to 0 (ignored)</td>
            </tr>
            <tr>
               <td>Extra Value</td>
               <td>single precision float</td>
               <td>Extra Value</td>
               <td>Set to 0 (ignored)</td>
            </tr>
         </tbody></table>
         
<h2 id="camera_status">CAMERA_STATUS (179)</h2>
         <p><strong>Versions</strong>: v1.5.5-</p>
         <p>NOTE: At some point, MAVLink.io decided to assign
            <a href="https://mavlink.io/en/messages/common.html#MAV_CMD_DO_SET_HOME">MAV_CMD_DO_SET_HOME</a> to 179, conflicting
            with the CAMERA_STATUS message's designation. As of v6.0.0 we do not recognize the MAV_CMD_DO_SET_HOME message,
            and still use 179 for this message.</p>
         <p>If a capture is commanded via MAVLink, a CAMERA_STATUS message with event type "TRIGGER" will be sent back on success</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/ardupilotmega.html#CAMERA_STATUS">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Time</td>
               <td>uint64_t</td>
               <td>Image timestamp (microseconds since UNIX epoch, according to camera clock)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Target System</td>
               <td>uint8_t</td>
               <td>System ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Camera Index</td>
               <td>uint8_t</td>
               <td>Camera ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Image Index</td>
               <td>uint16_t</td>
               <td>Image Index</td>
               <td>Capture number commanded by MAVLink (ignores all other captures, such as WiFi, button, and external trigger)</td>
            </tr>
            <tr>
               <td>Event ID</td>
               <td>uint8_t</td>
               <td>
                  <ol>
                     <li value="0">CAMERA_STATUS_TYPE_HEARTBEAT - Camera heartbeat, announce camera component ID at 1Hz</li>
                     <li>CAMERA_STATUS_TYPE_TRIGGER - Camera image triggered</li>
                     <li>CAMERA_STATUS_TYPE_DISCONNECT - Camera connection lost</li>
                     <li>CAMERA_STATUS_TYPE_ERROR - Camera unknown error</li>
                     <li>CAMERA_STATUS_TYPE_LOWBATT - Camera battery low. Parameter p1 shows reported voltage</li>
                     <li>CAMERA_STATUS_TYPE_LOWSTORE - Camera storage low. Parameter p1 shows reported shots remaining</li>
                     <li>CAMERA_STATUS_TYPE_LOWSTOREV - Camera storage low. Parameter p1 shows reported video minutes remaining</li>
                  </ol>
               </td>
               <td>
                  <ol>
                     <li value="1">CAMERA_STATUS_TYPE_TRIGGER - Camera image triggered</li>
                  </ol>
               </td>
            </tr>
            <tr>
               <td>P1</td>
               <td>single precision float</td>
               <td>Parameter 1</td>
               <td>Sent as 0</td>
            </tr>
            <tr>
               <td>P2</td>
               <td>single precision float</td>
               <td>Parameter 2</td>
               <td>Sent as 0</td>
            </tr>
            <tr>
               <td>P3</td>
               <td>single precision float</td>
               <td>Parameter 3</td>
               <td>Sent as 0</td>
            </tr>
            <tr>
               <td>P4</td>
               <td>single precision float</td>
               <td>Parameter 4</td>
               <td>Sent as 0</td>
            </tr>
         </tbody></table>
         
<h2 id="camera_feedback">CAMERA_FEEDBACK (180)</h2>
         <p><strong>Versions</strong>: v1.5.5-</p>
         <p>NOTE: At some point, MAVLink.io decided to assign
            <a href="https://mavlink.io/en/messages/common.html#MAV_CMD_DO_SET_PARAMETER">MAV_CMD_DO_SET_PARAMETER</a> to 180,
            conflicting with the CAMERA_FEEDBACK message's designation. As of v6.0.0 we do not recognize the
            MAV_CMD_DO_SET_PARAMETER message, and still use 180 for this message.</p>
         <p>If a capture is commanded via MAVLink, a CAMERA_FEEDBACK message with "Flags" type "CAMERA_FEEDBACK_CLOSEDLOOP" will be
         sent back on success</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/ardupilotmega.html#CAMERA_FEEDBACK">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Time</td>
               <td>uint64_t</td>
               <td>Image timestamp (microseconds since UNIX epoch), as passed in by CAMERA_STATUS message (or autopilot if no CCB)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Target System</td>
               <td>uint8_t</td>
               <td>System ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Camera Index</td>
               <td>uint8_t</td>
               <td>Camera ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Image Index</td>
               <td>uint16_t</td>
               <td>Image Index</td>
               <td>Capture number commanded by MAVLink (ignores all other captures, such as WiFi, button, and external trigger)</td>
            </tr>
            <tr>
               <td>Latitude</td>
               <td>int32_t</td>
               <td>Latitude in (deg * 1E7)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Longitude</td>
               <td>int32_t</td>
               <td>Longitude in (deg * 1E7)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Altitude (MSL)</td>
               <td>single precision float</td>
               <td>Altitude Absolute (meters AMSL)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Altitude Relative</td>
               <td>single precision float</td>
               <td>Altitude Relative (meters above HOME location)</td>
               <td>Altitude (in meters) above estimated ground level</td>
            </tr>
            <tr>
               <td>Roll</td>
               <td>single precision float</td>
               <td>Camera Roll angle (earth frame, degrees, +-180)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Pitch</td>
               <td>single precision float</td>
               <td>Camera Pitch angle (earth frame, degrees, +-180)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Yaw</td>
               <td>single precision float</td>
               <td>Camera Yaw (earth frame, degrees, 0-360, true)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Focal Length</td>
               <td>single precision float</td>
               <td>Focal Length (mm)</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Flags</td>
               <td>uint8_t</td>
               <td>
                  <ol>
                     <li value="0">CAMERA_FEEDBACK_PHOTO - Shooting photos, not video</li>
                     <li>CAMERA_FEEDBACK_VIDEO - Shooting video, not stills</li>
                     <li>CAMERA_FEEDBACK_BADEXPOSURE - Unable to achieve requested exposure (e.g. shutter speed too low)</li>
                     <li>CAMERA_FEEDBACK_CLOSEDLOOP - Closed loop feedback from camera, we know for sure it has successfully taken a picture</li>
                     <li>CAMERA_FEEDBACK_OPENLOOP - Open loop camera, an image trigger has been requested but we can't know for sure it has successfully taken a picture</li>
                  </ol>
               </td>
               <td>
                  <ol>
                     <li value="3">CAMERA_FEEDBACK_CLOSEDLOOP - camera has successfully taken a picture</li>
                  </ol>
               </td>
            </tr>
         </tbody></table>
         <h2 id="param_request_read">PARAM_REQUEST_READ (20)</h2>
         <p><strong>Versions</strong>: v7.1.0-</p>
         <p>Allows for reading of camera configuration settings</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#PARAM_REQUEST_READ">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Target System</td>
               <td>uint8_t</td>
               <td>System ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Target Component</td>
               <td>uint8_t</td>
               <td>Component ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Param ID</td>
               <td>char[16]</td>
               <td>Onboard parameter id, terminated by NULL if the length is less than 16 human-readable
                     chars and WITHOUT null termination (NULL) byte if the length is exactly 16 chars -
                     applications have to provide 16+1 bytes storage if the ID is stored as string</td>
               <td>Same as official. See the list of <a href="#supported_parameters">Supported Parameters</a></td>
            </tr>
            <tr>
               <td>Param Index</td>
               <td>int_16</td>
               <td>Parameter index. Send -1 to use the param ID field as identifier (else the param id will be ignored)</td>
               <td>Same as official.<br>
                  We do not guarantee indices for a specific parameter will be the same for all software versions,
                  so we recommend using the Param ID and setting the Param Index to -1 for the first access
                  of each parameter after power on. The returned index can be safely used on subsequent calls.</td>
            </tr>
         </tbody></table>
         <h2 id="param_request_list">PARAM_REQUEST_LIST (21)</h2>
         <p><strong>Versions</strong>: v7.1.0-</p>
         <p>Requests that the camera send the current value for every supported parameter</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#PARAM_REQUEST_LIST">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Target System</td>
               <td>uint8_t</td>
               <td>System ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Target Component</td>
               <td>uint8_t</td>
               <td>Component ID</td>
               <td>Same as official</td>
            </tr>
         </tbody></table>
         <h2 id="param_value">PARAM_VALUE (22)</h2>
         <p><strong>Versions</strong>: v7.1.0-</p>
         <p>The current value of a parameter, broadcast in response to a request to get one or more parameters 
         (<a href="#param_request_read">PARAM_REQUEST_READ</a>, <a href="#param_request_list">PARAM_REQUEST_LIST</a>) 
         or whenever a parameter is set (<a href="#param_set">PARAM_SET</a>).</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#PARAM_VALUE">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Param ID</td>
               <td>char[16]</td>
               <td>Onboard parameter id, terminated by NULL if the length is less than 16 human-readable
                     chars and WITHOUT null termination (NULL) byte if the length is exactly 16 chars -
                     applications have to provide 16+1 bytes storage if the ID is stored as string</td>
               <td>Same as official. See the list of <a href="#supported_parameters">Supported Parameters</a></td>
            </tr>
            <tr>
               <td>Param Value</td>
               <td>4 bytes of data</td>
               <td>Onboard parameter value</td>
               <td>Implemeted as a union. Interpreted as the <a href="#mav_param_type">MAV_PARAM_TYPE</a> 
               listed in the Param Type field under <a href="#supported_parameters">Supported Parameters</a></td>
            </tr>
            <tr>
               <td>Param Type</td>
               <td>uint8_t</td>
               <td>Onboard parameter type: see the <a href="#mav_param_type">MAV_PARAM_TYPE</a> enum for supported data types</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Param Count</td>
               <td>uint16</td>
               <td>Total number of onboard parameters</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Param Index</td>
               <td>uint_16</td>
               <td>Index of this onboard parameter</td>
               <td>Same as official</td>
            </tr>
         </tbody></table>
         <h2 id="param_set">PARAM_SET (23)</h2>
         <p><strong>Versions</strong>: v1.5.30-</p>
         <p>Allows for configuraiton of miscellaneous settings, not already captured by another MAVLink message</p>
         <table>
            <tbody><tr>
               <th>Field Name</th>
               <th>Type</th>
               <th><a href="https://mavlink.io/en/messages/common.html#PARAM_SET">Official Description</a></th>
               <th>RedEdge Implementation</th>
            </tr>
            <tr>
               <td>Target System</td>
               <td>uint8_t</td>
               <td>System ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Target Component</td>
               <td>uint8_t</td>
               <td>Component ID</td>
               <td>Same as official</td>
            </tr>
            <tr>
               <td>Param ID</td>
               <td>char[16]</td>
               <td>Onboard parameter id, terminated by NULL if the length is less than 16 human-readable
                     chars and WITHOUT null termination (NULL) byte if the length is exactly 16 chars -
                     applications have to provide 16+1 bytes storage if the ID is stored as string</td>
               <td>Same as official. See the list of <a href="#supported_parameters">Supported Parameters</a></td>
            </tr>
            <tr>
               <td>Param Value</td>
               <td>4 bytes of data</td>
               <td>Onboard parameter value</td>
               <td>Implemeted as a union. Interpreted as the <a href="#mav_param_type">MAV_PARAM_TYPE</a> listed in the Param Type field under <a href="#supported_parameters">Supported Parameters</a></td>
            </tr>
            <tr>
               <td>Param Type</td>
               <td>uint8_t</td>
               <td>Onboard parameter type: see the <a href="#mav_param_type">MAV_PARAM_TYPE</a> enum for supported data types</td>
               <td>Same as official</td>
            </tr>
         </tbody></table>
      </article>
