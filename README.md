# Devops
Set up an OSRM server on AWS using Kubernetes and Docker and make it live for external access.


Steps:
1. Create account on awx : https://eu-north-1.console.aws.amazon.com/
2. create EC2 instance --> ubuntu 22.04
3. Install docker on EC2 instance
   cmd --> i. sudo apt update
           ii. sudo apt install apt-transport-https ca-certificates curl software-properties-common
           iii. curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
           iv. echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
           v. sudo apt update
           vi. apt-cache policy docker-ce
          vii. sudo apt install docker-ce
          viii. sudo systemctl status docker
4.change docker permission
  cmd :
    sudo docker ps
    sudo chown $USER /var/run/docker.sock
    sudo usermod -a -G docker $USER
5.  osrm deployment:
   cmd :
     i. To keep the OS updated.
        sudo apt-get update
        sudo apt-get upgrade
        sudo apt-get install -y unattended-upgrades osmosis

      ii 
Create a dedicated directory and move into to keep everything tidy and clean

mkdir osrm
cd osrm
Get OSRM from the official GitHub repo

git clone https://github.com/Project-OSRM/osrm-backend.git
Compile and install OSRM binaries

cd osrm-backend
mkdir -p build
cd build
cmake ..
cmake --build .
sudo cmake --build . --target install
The map pre-processing is quite memory intensive. For this reason, OSRM uses a library called STXXL to map its internal operations on the hard disk. STXXL relies on a configuration file called .stxxl, which lives in the same directory where you are running your software, to determine how much space is dedicated to the STXXL data structures.

Configure STXXL with an .stxxl file in your osrm folder.

cd ~/osrm
vi .stxxl
Write in it

disk=/tmp/stxxl,10G,syscall
Save and close it.

Because the speed profile script might depend on some Lua functions defined in the profiles library, we also create a symbolic link to it in the same directory by running the following two commands.

ln -s osrm-backend/profiles/car.lua profile.lua
ln -s osrm-backend/profiles/lib
Get openstreetmap data

wget https://download.geofabrik.de/asia/india/central-zone-latest.osm.bz2
bzip2 -d central-zone-latest.osm.bz2



osrm-extract central-zone-latest.osm -p ./osrm-backend/profiles/car.lua

Build the contraction hierarchy

osrm-contract central-zone-latest.osrm

tmux
Start a routing engine HTTP server on port 5000 inside the tmux session

osrm-routed central-zone-latest.osrm
