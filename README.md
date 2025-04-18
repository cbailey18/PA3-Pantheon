# PA3-Pantheon
Code and data for Computer Networks PA3


# Installation
## 1. Install Oracle VirtualBox. Make sure to also download VBox GuestAdditions https://www.oracle.com/virtualization/technologies/vm/downloads/virtualbox-downloads.html
## 2. Download Ubuntu 18.04.6 https://releases.ubuntu.com/18.04/
## 3. Set up a virtual machine (VM) in VirtualBox:
1. Open VirtualBox --> New
- Type: Linux Version: Ubuntu (64-bit)
- Memory: ≥2048MB
- HDD: ≥25GB (VDI, Dynamically allocated)
2. Attach ISO: Settings --> Storage --> Optical Drive (Click CD icon) --> Chose a Disk File --> Chose Ubuntu 18.04.6 download
3. Install Ubuntu:
- Start VM → Install Ubuntu (don't chose Try Ubuntu)
- Check "Install Guest Additions" during installation
- Complete normal installation
- Reboot when prompted
4. Enable copy/paste between VM and host
- Power off VM completely
- Configure Settings:
- Settings → General → Advanced: Shared Clipboard: Bidirectional, Drag'n'Drop: Bidirectional
- Settings → System → Processor: Enable PAE/NX (if available)
- Settings → Display → Screen: Video Memory: ≥128MB, Enable 3D Acceleration
- Insert Guest Additions CD:Devices → Insert Guest Additions CD Image
- Reboot VM and run inside ubuntu terminal:
```
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
sudo mount /dev/cdrom /mnt
cd /mnt
sudo ./VBoxLinuxAdditions.run
sudo usermod -aG vboxsf $USER
sudo reboot
```
## 4. Install dependencies 
- Python 2.7 required for Pantheon
```
sudo apt update
sudo apt install -y python2.7 python2.7-dev python-pip
```
- Mahimahi dependencies can be found here: http://mahimahi.mit.edu/#getting

## 5. Install Pantheon and dependencies
Clone the Pantheon repository as instructed here: https://github.com/StanfordSNR/pantheon
```
git clone https://github.com/StanfordSNR/pantheon.git
git submodule update --init --recursive  # or tools/fetch_submodules.sh
```
Do NOT proceed to using tools/install_deps.sh. Instead, install mahimahi from source:
```
git clone https://github.com/ravinet/mahimahi
cd mahimahi
./autogen.sh
./configure
make
sudo make install
```
After installing mahimahi, do the following
```
cd pantheon/third_party/pantheon-tunnel
./autogen.sh
./configure
```
Now, proceed with downloading congestion control protocol specific dependencies and completing setup. For this assignment, I only utilized cubic, vegas, and pcc. For instructions on installing other dependencies, see the main Pantheon 
```
src/experiments/setup.py --install-deps -- schemes "cubic vegas pcc"
src/experiments/setup.py --setup -- schemes "cubic vegas pcc"
```
## 6. Test to ensure all programs have been installed correctly
Run a simple test to ensure everything is working properly by running a local test with a single congestion control protocol. Here, I use cubic.
```
src/experiments/test.py local --schemes "cubic"
```
If this runs successfully, you are good to move to the next step!

## 7. Experiment with different latency and bandwidth schemes
We will test the congestion control schemes in 2 different environments
1. Low latency (50mbps), high bandwidth (10ms RTT)
2. High latency (1mbps), constrained bandwith (200ms RTT)

First, we must set up the trace files for these conditions. Use the following code: 
```
cd pantheon/tests
echo "50" > 50mbps.trace 
echo "1" > 1mbps.trace 
```
To run our 3 congestion control algorithms in a low latency, high bandwidth environment, use the following code:
```
src/experiments/test.py local --schemes "cubic vegas pcc" -t 60 --uplink-trace tests/50mbps.trace --downlink-trace tests/50mbps.trace --append-mm-cmds "mm-delay 10" --data-dir insert_your_output_dir
```

For the high latency, constrained bandwidth environment, use the following: 
```
src/experiments/test.py local --schemes "cubic vegas pcc" -t 60 --uplink-trace tests/1mbps.trace --downlink-trace tests/1mbps.trace --append-mm-cmds "mm-delay 10" --data-dir insert_your_output_dir
```

Finally, analze the outputs
```
src/experiments/analyze.py --data-dir insert_your_output_dir
```
