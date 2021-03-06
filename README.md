# microservice-demo-robot-tests
## Environment install

### Mac

#### Download and install Oracle Virtualbox
https://www.virtualbox.org/wiki/Downloads

#### Download and install Vagrant
https://www.vagrantup.com/downloads.html

#### Get bento/ubuntu-16.04 vagrantbox and start it
https://app.vagrantup.com/bento/boxes/ubuntu-16.04
- create directory in PC: <home_dir>/vagrant-ubuntu1604 and cd to it
- vagrant init bento/ubuntu-16.04  #result is you have Vagrantfile in above mentioned directory
- edit Vagrantfile, add following lines after line "config.vm.box = "bento/ubuntu-16.04":
  ```
  config.vm.network "private_network", ip: "192.168.50.6"
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 9001, host: 9001
  config.vm.network "forwarded_port", guest: 9002, host: 9002
  config.vm.provider "virtualbox" do |v|
   v.name = "ubuntu1604-robot-demo"
   v.memory = 4028
   v.cpus = 2
  end
  ```
- vagrant up  #result is virtualbox vm up and running in Oracle Virtualbox ui
- vagrant ssh #you get ssh session to new Virtualbox vm

### Windows using Docker toolbox

#### Download and install Docker Toolbox
https://docs.docker.com/toolbox/toolbox_install_windows/

#### Launch the Docker Quickstart Terminal 

#### Download and install Vagrant
https://www.vagrantup.com/downloads.html

#### Get bento/ubuntu-16.04 vagrantbox and start it
https://app.vagrantup.com/bento/boxes/ubuntu-16.04
- create directory in PC: <home_dir>/vagrant-ubuntu1604 and cd to it
- vagrant init bento/ubuntu-16.04  #result is you have Vagrantfile in above mentioned directory
- edit Vagrantfile, add following lines after line "config.vm.box = "bento/ubuntu-16.04":
  ```
  config.vm.network "private_network", ip: "192.168.50.6"
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 9001, host: 9001
  config.vm.network "forwarded_port", guest: 9002, host: 9002
  config.vm.provider "virtualbox" do |v|
   v.name = "ubuntu1604-robot-demo"
   v.memory = 4028
   v.cpus = 2
  end
  ```
### Windows using Docker for Windows and Subsystem(Windows 10 with Creators update or later)

#### Download and install Docker for Windows(Disables Virtualboxes)
https://hub.docker.com/editions/community/docker-ce-desktop-windows

#### Change Docker settings to expose daemon
<img src="https://nickjanetakis.com/assets/blog/docker-for-windows-expose-daemon-without-tls-5118c5ffd844dd8dbb9e6b2935c108035bd0fc6b06565774db3aed18f138acc3.jpg" width="450">

#### Download and install Windows Subsystem(Ubuntu)
https://docs.microsoft.com/en-us/windows/wsl/install-win10

### Do following tasks inside new Virtualbox vm/Windows subsystem:

#### Install Java
 - sudo add-apt-repository ppa:openjdk-r/ppa -y
 - sudo apt-get -y update
 - sudo apt-get install openjdk-8-jre openjdk-8-jre-headless openjdk-8-jdk -y

#### Install Maven
 - sudo curl -O https://archive.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
 - sudo tar zxvf apache-maven-3.5.2-bin.tar.gz -C /opt/
 - sudo ln -s /opt/apache-maven-3.5.2/bin/mvn /usr/bin/mvn

#### Install Docker
 - sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
 - curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 - sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
 - sudo apt-get -y update
 - sudo apt-get install -y docker-ce

Note: After running following commands exit and log in to VirtualBox VM/Subsystem again:
 - sudo usermod -aG docker $USER 
 - sudo chmod g+rw /var/run/docker.sock

#### Install docker-compose
 - COMPOSE_VERSION=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d\\" -f4)
 - sudo curl -L "https://github.com/docker/compose/releases/download/$COMPOSE_VERSION/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
- sudo chmod +x /usr/local/bin/docker-compose

if you are using Windows subsystem do these steps after docker-compose installation
- echo "export DOCKER_HOST=tcp://localhost:2375" >> ~/.bashrc && source ~/.bashrc

Create and modify the new WSL configuration file inside Ubuntu:
- sudo nano /etc/wsl.conf

Add following to newly created file:
```
[automount]
root = /
options = "metadata"
```
Save file, exit terminal and log in

Verify succesfull connection to Daemon:
- docker ps
- docker info

#### Git clone Robot repo
 - make directory under home dir (e.g. /home/$USER/git) and cd to it
 - git clone https://github.com/jaheikki/microservice-demo-robot-tests.git

#### Clone and build projects with maven
 - cd /home/$USER/git/microservice-demo-robot-tests
 - ./clone_and_build_ms_demo_apps.sh

#### Start demo application
 - cd /home/$USER/git/microservice-demo-robot-tests/src/test/resources/
 - docker-compose -f docker-compose-dev.yml up -d  #result is demo application containers started, see 'docker ps'
 - wait 5 minutes to applications getting up
 - Note: Demo app (Orders tab) is kind of unstable, so get fresh start, stop app: docker-compose -f docker-compose-dev.yml down -v

### Test demo application UI in your PC
 - http://localhost:8080
 - Test that all 3 tabs (Products, Customers, Orders) are working

#### Git clone Robot report in your PC
 - make e.g. 'git' directory under e.g. home dir and cd to it
 - git clone https://github.com/jaheikki/microservice-demo-robot-tests.git

#### Start IntelliJ IDEA in your PC
https://www.jetbrains.com/idea/download
 - Install 'Robot Framework support' plugin to intelliJ IDEA (Preferences/Plugins)
 - Open Robot project: File/Open + navigate to git clone dir + select pom.xml + 'Open'  #in new window
 - Open robot file: src/main/resources/acceptance-tests/MicroserviceAcceptanceTests.robot  #now editing tests is possible

#### Run Robot tests in your PC (Notice that chromedriver must exist in path)
 - mvn -Probot clean install -Drobot.tag="Order product"  #single test by tag
 - mvn -Probot clean install  #run all tests

#### See the Robot test report in browser in our PC  #as outputted to console after test
 - e.g. file:///<home dir>/git/microservice-demo-robot-tests/target/robotframework-reports/log.html#s1-s1-s1-t1
