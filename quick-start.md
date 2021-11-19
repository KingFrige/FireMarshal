module load riscv-toolchian/chipyard


# base workload
./marshal -v -d build br-base.json
./marshal -v -d install -t prototype br-base.json
./marshal -v -d launch br-base.json


sudo gdisk /dev/sdc
sudo mkfs.hfsplus -v "PrototypeData" /dev/sdc2

sudo dd if=images/br-base-bin-nodisk-flat of=/dev/sdc1

screen -S FPGA_UART_CONSOLE /dev/ttyUSB1 115200

guestmount --pid-file guestmount.pid -a images/br-base.img -m /dev/sda disk-mount
guestunmount disk-mount

# coremark workload
./marshal -v -d build example-workloads/coremark/marshal-configs/coremark.json
./marshal -v -d install -t prototype example-workloads/coremark/marshal-configs/coremark.json
./marshal -v -d launch example-workloads/coremark/marshal-configs/coremark.json

sudo dd if=images/coremark-bin-nodisk-flat of=/dev/sdc1

qemu-system-riscv64 -nographic -bios none -smp 4 -machine virt -m 16384 -kernel /home/stc/riscv/chipyard/software/firemarshal/images/coremark-bin-nodisk -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-device,rng=rng0 -device virtio-net-device,netdev=usernet -netdev user,id=usernet,hostfwd=tcp::35396-:22 -s -S

qemu-system-riscv64 -nographic -bios none -smp 4 -machine virt -m 16384 -kernel /home/stc/riscv/chipyard/software/firemarshal/images/coremark-bin -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-device,rng=rng0 -device virtio-net-device,netdev=usernet -netdev user,id=usernet,hostfwd=tcp::53638-:22 -device virtio-blk-device,drive=hd0 -drive file=/home/stc/riscv/chipyard/software/firemarshal/images/coremark.img,format=raw,id=hd0 


# spec2006 workload
./marshal -v -d build example-workloads/spec2006-workload/marshal-configs/spec06-intspeed.json
./marshal -v -d install -t prototype example-workloads/spec2006-workload/marshal-configs/spec06-intspeed.json
./marshal -v -d launch example-workloads/spec2006-workload/marshal-configs/spec06-intspeed.json

riscv64-unknown-elf-objcopy -S -O binary --change-addresses -0x80000000 spec06-intspeed-bin-nodisk spec06-intspeed-bin-nodisk-flat

qemu-system-riscv64 -nographic -bios none -smp 4 -machine virt -m 16384 -kernel /home/stc/riscv/chipyard/software/firemarshal/images/spec06-intspeed-bin-nodisk -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-device,rng=rng0 -device virtio-net-device,netdev=usernet -netdev user,id=usernet,hostfwd=tcp::40342-:22 -s -S


# run-spec2006 workload
./marshal -v -d build example-workloads/run-spec2006-workload/marshal-configs/run-spec2006.json
./marshal -v -d install -t prototype example-workloads/run-spec2006-workload/marshal-configs/run-spec2006.json

sudo dd if=images/run-spec2006-bin-nodisk-flat of=/dev/sdc1 
