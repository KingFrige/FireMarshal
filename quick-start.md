module load riscv-toolchian/chipyard


./marshal -v -d build br-base.json
./marshal -v -d install -t prototype br-base.json
./marshal -v -d launch br-base.json


sudo gdisk /dev/sdc
sudo mkfs.hfsplus -v "PrototypeData" /dev/sdc2

sudo dd if=images/br-base-bin-nodisk-flat of=/dev/sdc1
sudo dd if=images/br-base.img of=/dev/sdc1

screen -S FPGA_UART_CONSOLE /dev/ttyUSB1 115200


./marshal -v -d build example-workloads/coremark/marshal-configs/coremark.json
./marshal -v -d install -t prototype example-workloads/coremark/marshal-configs/coremark.json
./marshal -v -d launch example-workloads/coremark/marshal-configs/coremark.json

sudo dd if=images/coremark-bin-nodisk-flat of=/dev/sdc1




./marshal -v -d build example-workloads/spec2006-workload/marshal-configs/spec06-intspeed.json
./marshal -v -d install -t prototype example-workloads/spec2006-workload/marshal-configs/spec06-intspeed.json
./marshal -v -d launch example-workloads/spec2006-workload/marshal-configs/spec06-intspeed.json

riscv64-unknown-elf-objcopy -S -O binary --change-addresses -0x80000000 spec06-intspeed-bin-nodisk spec06-intspeed-bin-nodisk-flat

guestmount --pid-file guestmount.pid -a images/br-base.img -m /dev/sda disk-mount
guestunmount disk-mount
