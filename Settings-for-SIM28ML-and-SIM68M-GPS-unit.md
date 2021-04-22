# Change settings for SIM28ML and SIM68M GPS unit

## Change change UART baud rate and data update period for SIM28ML and SIM68M GPS unit, which base on MT3333.

The SIM28 NMEA and PMTK commands are documented here https://bit.ly/3xezoQx

This GPS device has the following **default settings:**

 - UART baud rate of 9600 of 115200
 - GPS update rate of 1 Hz
 - The following NMEA sentences are output:

      - NMEA_SEN_RMC, // GPRMC interval - Recomended Minimum Specific GNSS Sentence
      - NMEA_SEN_GGA, // GPGGA interval - GPS Fix Data
      - NMEA_SEN_GSA, // GPGSA interval - GNSS DOPS and Active Satellites
      - NMEA_SEN_GSV, // GPGSV interval - GNSS Satellites in View

Below are the settings **I want:**
 - GPS update rate of **10 Hz**
 - The following NMEA sentences are output:

      - NMEA_SEN_RMC, // **GPRMC** interval - Recomended Minimum Specific GNSS Sentence

## NMEA
NMEA sentences are what most GPS devices output, when you turn on the GPS, the output just happens.  
You may be able to use the following command to listen to your GPS over a serial port:
However, if the baud rate is not 9600 (in my case) it does not work well.  

## PMTK
PMTK is language with which you can send commands to the your GPS device, also over the same serial port.  
You can send  a command like `$PMTK314,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0*2D<CR><LF>`
where:
 -  `$PMTK`: is the start of the string,
 -  `314`: is the PMTK command ID, this particular command is the PMTK_API_SET_NMEA_OUTPUT, which sets how frequently to display the NMEA commands;
 -  `,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0`: Is the command argument;
 -  `*2D`: is the checksum; online NMEA Checksum Calculator: https://bit.ly/3nlq1u9
 -  `<CR><LF>`: are sent in that order.

And you should get an acknowledgement like `$PMTK001,314,3*36`.

## Change the baud rate of the GPS
The `251` (PMTK_SET_NMEA_BAUDRATE) command can be used to change the baud rate.  We want to change this to 115200 baud.

`$PMTK251,9600*17			9600`
`$PMTK251,115200*1F 		115200`
No need to restart your device, but you will have to restart your serial port monitoring software.

## Reduce the cruft that gets sent
I am only interested in getting the following information:
 - position (lat, long, alt)
 - velocity and direction of travel

The `GGA` and `RMC` NMEA sentences provide this information.  The PMTK `314` (PMTK_API_SET_NMEA_OUTPUT) command allows you to set the NMEA output.  For the SIM28 GPS module this means I disable (set value to `0`) the other NMEA output and set the others to once every cycle.  This means once a second if you have the NMEA output rate at 1 Hz.

Documentation states that `RMC` is the 2nd option and `GGA` is the 4th.

`$PMTK314,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0*34`

The response should be something like `$PMTK001,314,3*36`.
$PMTK001,314,3*36

To reset this option to default, use the following command:

PMTK314,-1

## Update the rate of NMEA sentences
I would like to have NMEA sentences spat out at 10 Hz (10 times per second). 
This is simple to do, but the documentation is not clear.  
So, this is where my knowledge gets fuzzy, because the documentation is not clear.

The explains that EASY Mode (Embedded Assist System) works only at 1 Hz, it implies it conflicts with higher frequencies.  So we
turn off EASY Mode use PMTK command `869` (PMTK_CMD_EASY_ENABLE).  

**Disable EASY Mode:**

PMTK869,1,0

Once we have done that we can use PMTK command `220` (PMTK_SET_POS_FIX) to set the NMEA update rate.  The value is in
ms.  The documenation states it must be 200 ms or greater, but it seems to work at 100 ms.  The
also state that it should work at 10 Hz (100ms frequency).

**Set to 10 Hz update rate:**

PMTK220,100
You should see the update increase significantly.

Set to default:
PMTK220,1000

### End of file
