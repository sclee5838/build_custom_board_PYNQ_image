# build_custom_board_PYNQ_image
Brief explanation for PYNQ image build procedure


## .1.1 brief steps to make sd image

1. get PYNQ
    1. git clone https://github.com/Xilinx/PYNQ.git --depth 1
    2. ICS25 img 생성시 , v3.0.1 가져옴 
        1. 버전 Pynq v3.0 - Petalinux/Vitis 2022.1 사용해야 함.
        2. git clone --branch image_v3.0.1 --depth 1 https://github.com/Xilinx/PYNQ.git 
2. Make a custom board folder (let say it ‘RRAR’) below ‘boards’
    1. cd boards
    2. mkdir RRAR (ICS25)
3. Delete unnecessary board folder under ‘boards’
    1. ZCU104 폴더의 ZCU104.spec 파일은 부분 수정을 위해서 RRAR(ICS25) 폴더로 이동
    2. ZCU104 폴더 아래의 base, petalinux_bsp 폴더는 RRAR(ICS25) 폴더 아래로 이동
    3. rm -rf Pynq-Z1 Pynq-Z2 ZCU104
