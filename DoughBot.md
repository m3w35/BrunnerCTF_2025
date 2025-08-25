---
tags:
  - ctfs/challenges
ctf: "[[BrunnerCTF 2025]]"
areas:
  - "[[Forensics]]"
solved: true
---

> [!info] Prompt
> **Difficulty:** Beginner  
**Author:** rvsmvs
> 
> Our state-of-the-art smart mixer, DoughBot, crashed during a routine kneading cycle. Luckily, a technician was monitoring the device over UART and captured the memory output just before the reboot.
> 
> Analyze the captured dump and see what the DoughBot was trying to say before it rebooted.

# Walk Through

## Initial setup

Download the `forensics_doughbot.zip` file.

## List the contents

	❯ ls
	forensics_doughbot.zip
	❯ unzip -l forensics_doughbot.zip
	Archive:  forensics_doughbot.zip
	Length      Date    Time    Name
	---------  ---------- -----   ----
		0  2025-08-21 21:24   forensics_doughbot/
	  725  2025-08-21 06:32   forensics_doughbot/doughbot_dump.bin
	---------                     -------
	  725                     2 files

## Extract the contents

	❯ unzip forensics_doughbot.zip
	Archive:  forensics_doughbot.zip
	   creating: forensics_doughbot/
	  inflating: forensics_doughbot/doughbot_dump.bin  

## Poke around

	❯ file doughbot_dump.bin
	doughbot_dump.bin: Windows SYSTEM.INI

### Inspect the contents of the file

	❯ cat doughbot_dump.bin
	[BOOT] DoughBot 1.2.4
	[INFO] Initializing sensor calibration...
	[INFO] Sensor calibration complete.
	[INFO] Establishing Wi-Fi connection...
	[WARN] Wi-Fi unstable, retrying...
	[INFO] Uploading diagnostics...
	[DEBUG] Loading configuration from EEPROM...
	[EEPROM CONFIG DUMP @ 0x2000]
	    device_name     = "DoughBot"
	    firmware_ver    = 1.2.4
	    knead_duration  = 780
	    mix_speed       = AUTO
	    safety_timeout  = 300
	    temp_unit       = "C"
	    debug_enabled   = true
	    log_mode        = FULL
	
	// dev.note: bootlog_flag=YnJ1bm5lcnttMXgzZF9zMWduYWxzXzRfc3VyZX0=
	
	[CRASH] Unexpected interrupt.
	[REBOOT] Attempting recovery boot...
	[WARN] ??>!!%0^ [RECV_ERR]  3499$
	[WARN] 0x00ff @@@ERROR@:~:~
	[BOOT] Safe Mode Active.

Ooh, we found a comment describing what appears to be a variable called `bootlog_flag` with a `base64` encoded value. Let's try to decode the value.

## Decode the `base64` value

	❯ echo 'YnJ1bm5lcnttMXgzZF9zMWduYWxzXzRfc3VyZX0=' | base64 -d
	brunner{m1x3d_s1gnals_4_sure}%        

## Flag found

**brunner{m1x3d_s1gnals_4_sure}**
