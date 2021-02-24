# An overview on the data logging approach.
___
### File storage architecture:  
 /media/$USER/USB/
 
	/controller
	
	
        /error_<timestamp>_c.log


        /syslog_<timestamp>_c.log


	/processor


		/error_<timestamp>_p.log
		
		/syslog_<timestamp>_p.log


### Log file types:

1.  **error-log**: 
- Filename: error_<timestamp>.log
- Frequency: As per occurence 
- Source of data: events
- Format: 

```sh
[time], [version], [mission instance],  [error-code],[error description],  [event occurence or source]
```
2. **sys-log**:
- Filename: syslog_<timestamp>.log
- Source of data: Software/firmware generated
- Format:
 ```sh
[time], [version], [software_module number], [generated information]
```
 eg: **File: */media/yantra-deploy/USB/error_2021-01-08T12:22:11.log***

```sh 
Jan 8 11:00:38, 2.1.0, 0x032A, 0x0C41 , Failed to locate map, nav_msgs/GetPlan PID: 672 
```
Message breakdown: 
| Field | Description |
 ------ | ------ |
| Version | Software version number |
| Mission instance| 16-bit hex, system generated from the mission initiator(initiates file operation).[1] 
|Error-Code|  16-bit hex value that can be looked-up in the database. [2]

[1]: The instance number has the following metadata: 
 - Home co-ordinates (start)
 - Destination co-ordinates (end)
-  POST result (Power On Self Test)
- Battery %

[2]: Error-Code: Error Code is a 16-bit hex value that can be looked-up in the database. Each error-code has a separate error description stored in that database. 


## Push frequency and Conditions: 
___
###	Frequency: 
The data will be uploaded to the server after every mission so that each mission instance is logged. 
 ### Conditions (state identification)
 State 1: Once succesfully Docked, the CPU usage is minimum and hence is a suitable time to push the data remotely. (most favorable for Compression also)

State 2: Spontaneous update of the data (?) Maybe when storage is getting filled and data cannot be pushed during docking. 

State 3: Busy state so all non-time critical applications are paused. (Unfavorable for compression or update of the data.)

### Method: 
	
[FTPS](https://en.wikipedia.org/wiki/FTPS) or [rsync](https://linux.die.net/man/1/rsync) server is configured on the remote end. 
From the processor (client) , the data is pushed as per the frequency and conditions met. 
eg: 
```sh
ftps://[user[:password]@]host[:port]/url-path/
```
	

### Data removal/clean-up:
___
### Frequency/condition: 
The data is removed once storaged is filled above 80%. 
### Method: 
Data sorted in queue format: FIFO(First In First Out) and deleted sequentially. 


