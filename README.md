# zRam
ZRam is a kernel module that provides a compressed block device in RAM. This block device can be mounted as a swap device. When the system RAM starts to fill the kernel dynamicly compresses some of the information and moves it into the block device. This is much faster than a traditional swap as it all happens in ram. There is a small cpu hit. For systems with limited ram and storage or slow storage such as an old PC, Laptop or raspberry pi. With lz4 compression there is about a 2.5 compression ratio. However some space needs to be left in RAM for decompression of memory.

[zRam documentation on kernel.org](https://www.kernel.org/doc/html/latest/admin-guide/blockdev/zram.html)

[zRam for impoving performance at wiki.archlinux.org](https://wiki.archlinux.org/title/improving_performance#zram_or_zswap)


# Try out zRam
## This assumes there is no current swap setup.

### Load the zram kernel module.
`sudo modprobe zram`

### Set compression algorithm to lz4.
`sudo bash -c "echo 'lz4' > /sys/block/zram0/comp_algorithm"`

### Set the size of the zram device.
The size should not exceed 2.5 the size of the actual RAM. To find the systems RAM size in kb try this command.\
`grep MemTotal /proc/meminfo`

For double that size in MegaBytes instead of KiloBites try this.\
`echo $(($( grep MemTotal /proc/meminfo | grep -o '[0-9]*' ) / 500))M`

If that gave you a number that looks good. Use this command to set it.\
`sudo bash -c "echo $(($( grep MemTotal /proc/meminfo | grep -o '[0-9]*' ) / 500))M > /sys/block/zram0/disksize"`

If the above commands didn't work for you. You can choose a number manually in M or G Megabytes or Gigabytes. With a command like this.\
`sudo bash -c "echo '2G' > /sys/block/zram0/disksize"`

### Create a swap device.
`sudo mkswap --label zram0 /dev/zram0`

### Set the priority of the swap device.
`sudo swapon --priority 100 /dev/zram0`

### Check if swap is working.
If htop is installed it will show the active RAM/Swap size.\
`htop`


# Disable zRam.
## This isn't needed if zram is working. zRam will not be recreated on next boot. See systemd section.
### Disabling zram swap.
`sudo swapoff /dev/zram0`

### Unloading zRam kernel module.
`sudo rmmod zram`


# Auto create zRam with systemd service.
## A simple zram.service file using the above commands can be used to make a systemd service.
### Download zram.service
`wget https://raw.githubusercontent.com/JeremiahCheatham/zRam/main/zram.service`\
or\
`curl -LO https://raw.githubusercontent.com/JeremiahCheatham/zRam/main/zram.service`
### Copy zram.service file into place.
`sudo cp zram.service /etc/systemd/system`
### Enable zram.service.
`sudo systemctl enable zram`
### Start zram.service.
`sudo systemctl start zram`

# Disbale or remove zram.service.
### Stop zram.service.
`sudo systemctl stop zram`
### Disable zram.service.
`sudo systemctl disable zram`
### Remove zram.service file. First stop and disable zram.service.
`sudo rm /etc/systemd/system/zram.service`
