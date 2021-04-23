# Change settings for SIM28ML and SIM68M GPS units

## How to change UART baud rate and data update period for SIM28ML and SIM68M GPS units, that are based on MT3333.

## NMEA
NMEA sentences are what most GPS devices output when you turn on the GPS, the output just happens.  
You may be able to use commands described below to set your GPS over a serial port.
The SIM28 NMEA and PMTK commands are documented here https://bit.ly/3xezoQx

## PMTK
PMTK protocol is used to send commands to your GPS over the same serial port.

##1.Set connection
**For serial port this GPS device has the following default settings:**
	- `baud rate: 		9600 or 115200`
	- `data bits: 		8`
	- `stop bits: 		1`
	- `parity: 			None.`
	- `flow control:	None.`
	
##2.Test connection
To check the connection use the following command:
`$PMTK000*32`

In response, we must receive confirmation:
`$PMTK001,0,3*30`
 
##3.Below are the settings **I want:**
 - GPS update rate **10 Hz**
 - The following NMEA output sentence:
      - NMEA_SEN_RMC, // **GPRMC** interval - Recommended Minimum Specific GNSS Sentence

**The following NMEA sentences are the output:**
      - NMEA_SEN_RMC, // GPRMC interval - Recommended Minimum Specific GNSS Sentence
      - NMEA_SEN_GGA, // GPGGA interval - GPS Fix Data
      - NMEA_SEN_GSA, // GPGSA interval - GNSS DOPS and Active Satellites
      - NMEA_SEN_GSV, // GPGSV interval - GNSS Satellites in View
	  - .....
	  
**GPS update rate is 1 Hz**

##4. Change the baud rate of the GPS
First we have to upper the baud rate.
Because at a speed of 9600 and a update rate of 10 Hz, the NMEA sentences will not have time to be transmitted.

The `251` (PMTK_SET_NMEA_BAUDRATE) command can be used to change the baud rate. 
Example: if we want to change to 115200 baud, enter the following command:
`$PMTK251,115200*1F`
To change to 9600 use `$PMTK251,9600*17`
No need to restart your device, but you will have to restart your serial port monitoring software.
*Be careful: confirmation is absent.*

## How to choose relevant sentences
I am only interested in getting the following information:
 - position (lat, long, alt)
 - velocity and direction of travel
 
The `GGA` and `RMC` NMEA sentences provide this information.  
You can use PMTK `314` (PMTK_API_SET_NMEA_OUTPUT) command.
It command allows you to set the NMEA output.  
`$PMTK314,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0*2D<CR><LF>`
where:
 -  `$PMTK` is the start of the string;
 -  `314,` is the PMTK command ID, this particular command is the PMTK_API_SET_NMEA_OUTPUT, which sets how frequently to display the NMEA commands;
 -  `1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0` is the command argument;
 -  `*2D` is the checksum; online NMEA Checksum Calculator: https://bit.ly/3nlq1u9;
 -  `<CR><LF>` are sent in this exact order.
And we should get the following confirmation: 
`$PMTK001,314,3*36`

Documentation states that `RMC` is the 2nd option and `GGA` is the 4th.
`$PMTK314,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0*34`

To reset this option to default, use the following command:
`$PMTK314,-1*04`

## Update the rate of NMEA sentences
I would like to receive NMEA sentences at 10 Hz (10 times per second). 
This is simple to do, but the documentation is not clear.  
It is possible that we have to turn off the EASY mode.

**Disable EASY Mode:**
use PMTK command `869` (PMTK_CMD_EASY_ENABLE)
`$PMTK869,1,0*34`

**Set to 10 Hz update rate:**
Now we can use PMTK command `220` (PMTK_SET_POS_FIX) to set the NMEA update rate.  
To receive the update frequency at 10Hz, we will need to set the period to 100ms.
To do that, use the following command:
`$PMTK220,100*2F`
And we should get the following confirmation: `$PMTK001,220,3*30`.

If you want to reset to default, use the following command:
`$PMTK220,1000*1F`

##Summary
1) set RMC only:	`$PMTK314,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0*35`
					 answer: `$PMTK001,314,3*36`
2) set 10 Hz:		`$PMTK220,100*2F`
					 answer: `$PMTK001,220,3*30`

*End of file*