# AES Accelerator IoT Camera Demo

This project is built upon two other repos.

1. App & Modules [aes128_driver](https://github.com/happyx94/aes128_driver)

2. HDL [AXIS_AES128](https://github.com/happyx94/AXIS_AES128)


# \*Under Construction...

---
Create HDF and Generate Bitstream
---

#### Prequsites:
	a. Have Vivado 2017.4 and git installed on your machine 
	b. Have the minized board definition file (BDF) in <your-vivado-install_path>/data/boards
		(This can be downloaded from Avnet website. Tutorials are also available there.

1. Download the minized_petalinux project 

	cd ~/Vivadoprojects
	git clone git://github.com/Avnet/hdl.git

2. Open Vivado. In the TCL prompt window, type

cd ~/Vivadoprojects/hdl/Scripts
source ./make_minized_petalinux.tcl

3. Open the project /hdl/Projects/minized_petalinux/




1. Clone the AXIS_AES128 project. 

2. Open IP locations -> AXIS_AES128

3. Add the IPs, axis_aes128, axilite_aes_cntl, AXI Direct Memory Access, and two instances of AXIS_DATA_FIFO to the block design.

4. Connect the signals accordingly. 

M_AXIS_MM2S of the axi_dma 	to 	S_AXIS of axis_data_fifo_0 

M_AXIS of axis_data_fifo_0 	to 	S00_AXIS of axis_aes128 

M00_AXIS of axis_aes128 	to 	S_AXIS of axis_data_fifo_1

M_AXIS of axis_data_fifo_1	to	S_AXIS_S2MM of axi_dma

aes_key of axilite_aes_cntl	to	cipher_key of axis_aes128

set_IV of axilite_aes_cntl	to	set_IV of axis_aes128

5. Let auto-connection tool handle the rest of the connections. 

6. Double click on axi_dma. 

Disable Scatter Gather Engine

Set Width of Buffer Length Register to 20 bits.

Set the Memory Map Data Width and Stream Data Width to 128 bits.

7. Double click on the axi_data_fifo (do this step for both data fifos)

Set TDATA width to 16 bytes

8. Double clock on the processing_system (PS)

Clock Configuration -> PL Fabric Clocks -> 

Set FCLK_CLK_0 to 71 MHz 

Set FCLK_CLK_1 to 35 MHz 

9. Double click on bluetooth_uart

Set External CLK Freqency to 35 MHz

10. On the toolbar, click Validate Block Design. (And pray for no errors :) )

11. Click Regenerate Layout on the toolbar. You should have something similar to the following schematic. Save the block design.

12. On top of the toolbar of block design window, click on the Address Editor tab.

Record the Offset Address of
processing_system7_0 
-> Data
  -> axi_dma_0
  -> axilite_aes_cntl_0

12. Go to Tools -> Settings -> Project Settings -> Implementation 

Change the Strategy under Options to Performance_ExtraTimingOpt

13. Run Synthesis. Run Implementation. Generate Bitstream.

14. File -> Export -> Export Hardware (check Include Bitstream)


---
Create and Program the Binary to Flash
---

#### Prequsites:
	a. Running Ubuntu 16.04
	b. Installed PetaLinux Tools 2017.3 (Link Follow the user guide to so do)
	c. Have the minized_petalinux.hdf and minized_petalinux.bit files from the previous section.


1. Download minized_qspi.bsp from minized.org

2. Source $(PETALINUX)/settings.sh to start PetaLinux enviornment if you haven't

3. Create a folder for PetaLinux projects if you don't have one yet 
(i.e. mkdir ~/projects). cd to the folder.

4. Create a petalinux project by typing the following.

petalinux-create -t project -n minized_qspi -s <path-to-the-minized_qspi.bsp>


5. Replace minized_qspi/hardware/MINIZED/minized_petalinux.sdk/minized_petalinux_hw.hdf
with the minized_petalinux_wrapper.hdf under your Vivado project directory (where you have exported the hardware to).


Replace minized_qspi/hardware/MINIZED/minized_petalinux.sdk/minized_petalinux_hw/minized_petalinux_hw.bit
with the bitstream (*.bit) file under your Vivado project directory.

6.

cd to minized_qspi

petalinux-config --get-hw-description=./hardware/MINIZED/minized_petalinux.sdk/

The config screen should pop up. Just save and exit.

petalinux-build

7. Copy boot_gen.sh, bootgen.bif, and zynq_fsbl.elf from the minized_qspi BSP zip file to ~/minized_qspi
./boot_gen.sh
Rename the resulting boot.bin file to flash_fallback_7007S.bin

8. Start xsct in sudo mode by typing

sudo <your-vivado-install_path>/Xilinx/SDK/2017.4/bin/xsct

9. Connect minized to your computer

10. 

exec program_flash -f BOOT.fsbl -bin zynq_fsbl.elf -flash_type qspi_single


---
Create PetaLinux Project
---

Prequsites:
	a. Running Ubuntu 16.04
	b. Installed PetaLinux Tools 2017.3 (Link Follow the user guide to so do)
	c. You have created the minized_qspi project and program the flash accordingly
	d. A USB stick


1. Download minized.bsp from minized.org

petalinux-create -t project -n minized -s <path-to-the-minized.bsp>

2. Source $(PETALINUX)/settings.sh to start PetaLinux enviornment if you haven't

3. cd <your-project-folder>.

4. Create a new petalinux project by typing the following.

petalinux-create -t project -n minized_qspi -s <path-to-the-minized_qspi.bsp>

5. Replace minized_qspi/hardware/MINIZED/minized_petalinux.sdk/minized_petalinux_hw.hdf
with the minized_petalinux_wrapper.hdf under your Vivado project directory (where you have exported the hardware to).

6. cd to minized

petalinux-config --get-hw-description=./hardware/MINIZED/minized_petalinux.sdk/

Optional: When the settings screen pop up, change the rootfs type from initram to initrd if you think your image.ub will exceed 64MB.

7. cd ./project-spec/meta-user/recipes-core/images

8. 
	petalinux-config -c kernel
	-- turn off DMA drivers.
	-- turn on UVC

9. Follow the steps in 

10. Copy ./images/linux/image.up to your USB stick. Copy the tool scripts as well as the wifi config files to the usb stick

11. Boot your minized. Interrupt the autobooting. In uboot shell, type

	run boot_qspi

12. Plug-in an extra power cable and the USB stick. Mount the device to /mnt/usb if it is not mounted automatically. 

13. 
	cd /mnt/usb	
	cp image.ub ../emmc/
	mkdir ../emmc/demo
	cp <all-demo-files-and-wifi-conf-file> ../emmc/demo
	sync
	reboot


---
Run the Demo
---
1. Setup the receiver side on your computer.
Record the ip address of your computer. Run

ifconf

to see you IP address.

2. Run ./receiver 5000 8192

3. Boot minized. On the shell, run
	
	wifi.sh
	/mnt/emmc/demo/easy_demo.sh <your-computers-ip>






  

