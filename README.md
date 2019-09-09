# GetStarted_IM69D130_With_RaspberryPi
Interfacing MEMS Microphone with Raspberry Pi start guide

Steps-
1- Follow the document named "Raspberry Pi with MEMS Microphone" to interface Infineon MEMS Microphone with Raspberry pi and record the sound and play the recorded sound.

Result- After following the document Microphone will be running but with some issues-
###### Less area coverage for recording i.e. sound coming from far cannot be recorded.
###### While playing the audio the volume will be very feeble even if you increase the raspberry pi volume to max then also sound given 	     out by raspberry will be in low volume.

2- To remove the volume issue follow document named "Raspberry Pi with MEMS (Part 2)- Adding volume control" this will boost up the volume and increase the senstivity of microphone.

Result-	
###### Best quality of sound recorded.
###### While playing back volume is boosted.
###### Whenever the raspberry pi will be switched on alsamixer needs to be configured (Volume to be raised after selecting the 		snd_rpi_card) then again record and now volume wil be boosted with increase in senstivity. 
