# build_custom_board_PYNQ_image
Brief explanation for PYNQ image build procedure


## .1.1 brief steps to make sd image

# 4.1 RRAR board PYNQ build

## 4.1.1 brief steps to make sd image

1. get PYNQ
    1. git clone https://github.com/Xilinx/PYNQ.git --depth 1
    2. ICS25 image use PYNQ v3.0.1
        1. For Pynq v3.0 , Compatible Tool version : Petalinux/Vitis 2022.1 
        2. git clone --branch image_v3.0.1 --depth 1 https://github.com/Xilinx/PYNQ.git 
2. Make a custom board folder (let say it ‘RRAR’) below ‘boards’
    1. cd boards
    2. mkdir RRAR (ICS25)
3. Delete unnecessary board folder under ‘boards’
    1. copy PYNQ/boards/ZCU104.spec to RRAR(ICS25) folder
    2. copy PYNQ/boards/ZCU104/base, petalinux_bsp folder to RRAR(ICS25)
    3. rm -rf Pynq-Z1 Pynq-Z2 ZCU104
    
4. bitstream, xsa, spec file
    1. Copy <my bitstream file> to ‘RRAR/base’
    2. Copy <my XSA file> to ‘boards/RRAR/petalinux_bsp/hardware_project’
        1. create 'hardware_project' folder below petalinux_bsp
        2. copy xsa file to 'hardware_project' folder
    3. Make a RRAR.spec file & modification
        1. location :  RRAR.spec file ‘boards/RRAR’ folder
        2. copy ZCU104.spec file and modifed
        3. Left the BSP_XXX section empty. Removed boot_leds and sensorconf from the STAGE4_PACKAGES_XXX section.
    
    ```jsx
    ARCH_RRAR := aarch64
    BSP_RRAR :=
    BITSTREAM_RRAR := base/my_bitstream.bit
    FPGA_MANAGER_RRAR := 1
    STAGE4_PACKAGES_RRAR := xrt pynq ethernet pynq_peripherals
    ```

5. Prepare system-user.dtsi file for RRAR board & modification for SD card on mmcblk1 
    1. PYNQ default use SD card on mmcblk0 , but RRAR board SD is on mmcblk1
    2. Copy ZCU104  boards/ZCU104/petalinux_bsp to RRAR
        1. For ICS25, the system_user.dtsi provided after installing PYNQ was replaced with the one from the rrar folder. Updated the gem3 section to gem2.
        2. Copy uboot-device-tree directory as it is not present by default. Keep system-user.dtsi in uboot-device-tree identical to device-tree/files
        - PYNQ/boards/RRAR/petalinux_bsp/meta-user/recipes-bsp/device-tree/files
        - PYNQ/boards/RRAR/petalinux_bsp/meta-user/recipes-bsp/uboot-device-tree/files
        
        ### system-user.dtsi updates
        
        - ‘gem3’ node is needed to fix ZYNQ GEM error in U-boot

        
<img width="863" height="313" alt="image1" src="https://github.com/user-attachments/assets/be576a9b-fe99-499e-9332-4320ca6a60d4" />

        - ‘sdhci1’ node is needed to boot on SD card rootfs
        - add spi0,1 node to use linux kernel spidev driver(see below 4.1.2 system-user.dtsi file)
        
        ### system-conf.dtsi in buidling RRAR
        
        ```jsx
        /include/ "system-conf.dtsi"
        / {
        chosen{ 
        		bootargs = "earlycon clk_ignore_unused console=ttyPS0,115200 root=/dev/mmcblk1p2 rw rootfstype=ext4 rootwait"; 
        	};
        };
        &gem3 { 
        	status = "okay";
        	phy-handle = <&phy0>;
        	phy-mode = "rgmii-id";
        
        	phy0: ethernet-phy@c {
        		#phy-cells = <1>;
        		reg = <0xc>;
        		device_type="ethernet-phy";
        		ti,rx-internal-delay = <0x8>;
        		ti,tx-internal-delay = <0xa>;
        		ti,fifo-depth = <0x1>;
        		ti,dp83867-rxctrl-strap-quirk;
        	};
        };
        /* SD1 with level shifter */
        &sdhci1 {    
        	disable-wp;
        	no-1-8-v;
        	max-frequency=<20000000>;
        };
        
        /* PS SPI0 */
        &spi0 {
            is-decoded-cs = <0>;
            num-cs = <1>;
            status = "okay";
            spidev@0 {
                compatible="rohm,dh2228fv";
                reg =<0>;
                spi-max-frequency = <10000000>;
            };
        };
        
        /* PS SPI1 */
        &spi1 {
            is-decoded-cs = <0>;
            num-cs = <1>;
            status = "okay";
            spidev@0 {
                compatible="rohm,dh2228fv";
                reg =<0>;
                spi-max-frequency = <10000000>;
            };
        };
        
        ```
        
6. Change Rootfs location to ‘mmcblk1p2’ in PYNQ files
    - “/dev/mmcblk0p2” ⇒ “/dev/mmcblk1p2” in sdbuild/Makefile
    
    ```jsx
    echo 'CONFIG_SUBSYSTEM_SDROOT_DEV="/dev/mmcblk1p2"' >> $$(PL_CONFIG_$1)
    ```
    
<img width="858" height="270" alt="image2" src="https://github.com/user-attachments/assets/3fb8b0e2-7769-4e5c-8c55-e24c9ebd43c6" />

    
    - /dev/mmcblk0p2 ⇒ /dev/mmcblk1p2  in <PYNQ>/sdbuild/boot/meta-pynq/recipes-bsp/device-tree/files/pynq_bootargs.dtsi
    
    ```jsx
    chosen {
    bootargs = "root=/dev/mmcblk1p2 rw earlyprintk rootfstype=ext4 rootwait devtmpfs.mount=1 uio_pdrv_genirq.of_id=\"generic-uio\" clk_ignore_unused";
    pynq_board = "Unknown";
    };
    ```
    
7. Download prebuilt rootfs and sdist file and copy them to sdbuild/prebuilt folder
    
    cp -rf /home/download/prebuilt/* sdbuild/prebuilt
    
8. Tool seutp
    
    source ~/petalinux/2022.1/setting.sh
    
    source ~/Xilinx/Vitis/2022.1/setting64.sh
    
9. cd sdbuild/
    
    make clean
    
    make
    

[[Custom board: Rootfs on /dev/mmcblk1p2 not found - Support - PYNQ](https://discuss.pynq.io/t/custom-board-rootfs-on-dev-mmcblk1p2-not-found/1248)](https://discuss.pynq.io/t/custom-board-rootfs-on-dev-mmcblk1p2-not-found/1248)

## 4.1.2 system-user.dtsi

To add PS_SPI0,1 

<img width="1175" height="950" alt="image3" src="https://github.com/user-attachments/assets/d21fa9b4-63db-46bb-b61e-bfedae4196cc" />

Note : Modification of system-user.dtsi to add spidev in peta-linux 

<img width="672" height="829" alt="image4" src="https://github.com/user-attachments/assets/06c30b8f-3d0b-498d-84a9-27d9a49de598" />
