# M480BSP_ISP_HID_20_APROM
 M480BSP_ISP_HID_20_APROM


update @ 2022/01/04

1. Scenario notice:

	- Boot loader project : ISP_HID_20 
	
		- under sct file (hid_iap.sct) , will allocate flash size (target MCU flash size : 512K , use flash_calculate.xlsx to calculate)
		
				LDROM_Bootloader.bin : 0x100000 ~ 0xFFF (default LDROM size : 4K)
			
				APROM_Bootloader.bin : 0x7B000 0x4000 (reserve 16K size , to store extra boot loader code , total boot loader size : 20K)
	
		- when power on , will check power on source (ex : power on reset , nReset , from application code)
	
		- use CRC to calculate Application code checksum (length : 0x7AFFC)
		
		- load Application code checksum , from specific address (at 0x7AFFC)
		
		- if two checksum result are different , will trap in Boot loader , and wait for ISP tool hand-shaking
		
		- if two checksum result are the same , will jump to Application code

		- if reset from application code , will entry timeout counting (5sec) , jump to application code if not receive ISP tool command
	
	- Application code project : AP
	
		- use SRecord , to calculate application code checksum 
		
		- SRecord file : srec_cat.exe , generateChecksum.bat (will execute generateChecksum.cmd , generateCRCbinary.cmd , generateCRChex)
					
		- under generateChecksum.bat will execute generateChecksum.cmd , generateCRCbinary.cmd , generateCRChex.cmd
	
		- generateChecksum.cmd : calculate checksum by load the original binary file , and display on KEIL project
		
		- generateCRCbinary.cmd : calculate checksum by load the original binary file , and fill 0xFF , range up to 0x7AFFC
		
		- generateCRChex.cmd : conver binary file into hex file
		
		- at KEIL output file , file name is APROM_application , under \obj folder , 
	
			which mapping to generateChecksum.cmd , generateCRCbinary.cmd , generateCRChex.cmd
	
			modify the file name in KEIL project , also need to modify the file name in these 3 generate***.cmd		
			
		- check sum calculate will start from 0 to 0x7AFFC , and store in (0x7B000 - 4) address (0x7AFFC)
		
		- after project compile finish , binary size will be 492K (total application code size : 0x7B000)
		
		- under terminal , use keyboard , 'z' , 'Z' , will write specific value in RTC backup register , and return to boot loader
		
		- use ISP tool , to programming Application code project binary (500K) , when under Boot loader flow		
		
		- if plan to use KEIL to programming flash with CRC , refer to below setting
		
![image](https://github.com/released/M480BSP_ISP_HID_20_APROM/blob/main/program_by_KEIL.jpg)
		
		
	- reserve data flash address : 0x7F000
	
2. Flash allocation start address

	- LDROM_Bootloader.bin : 0x100000 ~ 0xFFF
	
	- APROM_Bootloader.bin : 0x7B000 0x4000
	
	- Application code : 0x00000
	
	- Data flash : 0x7F000
	
	- Chcecksum storage : 0x7AFFC

3. Function assignment

	- debug port : UART0 (PB12/PB13) , in Boot loader an Application code project
	
	- enable RTC and CRC , Timer1 module
	
4. Need to use ICP tool , to programm boot loader project file (LDROM_Bootloader.bin , APROM_Bootloader.bin)

below is boot loader APROM setting and program LDROM , APROM , config 

![image](https://github.com/released/M480BSP_ISP_HID_20_APROM/blob/main/LDROM_ICP.jpg)

below is boot loader project , Config setting 

![image](https://github.com/released/M480BSP_ISP_HID_20_APROM/blob/main/Config_Bits.jpg)

in Application project , press 'z' , 'Z' will reset to Boot loader 

![image](https://github.com/released/M480BSP_ISP_HID_20_APROM/blob/main/Under_APROM_z.jpg)

with ISP tool , set connect and wait USB connection from APROM jump to LDROM , and programming application code

![image](https://github.com/released/M480BSP_ISP_HID_20_APROM/blob/main/ISP_connect.jpg)

below is log message , from boot loader , to application code

![image](https://github.com/released/M480BSP_ISP_HID_20_APROM/blob/main/regular_power_on_check_checksum.jpg)

6. if checksum incorret , trap in boot loader 

![image](https://github.com/released/M480BSP_ISP_HID_20_APROM/blob/main/error_checksum_stay_in_boot_loader.jpg)


