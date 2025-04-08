# Integrating Network Intrusion Detection Systems and Neural Network with Tensoflow

In this project, I will attempt to link a Network Intrsuion Detecion System and a Neural Network using Tensorflow to classify network activity

The aim of this project, is to link these two systems together, to develop an autonomous system where network traffic is captured using a Security Information and Event Manager system and this network traffic is then passed through a Neural Network which will output if the traffic captured is malicious or benign.

The idea behind this project is develop a system that will lower the costs of network security for organizations and provide a high level of sophistication and accuracy when dealing with network security.

I believe machine learning will have a major impact on security and how security is implemented in a company. The reason I decided on this project was a personal one, I worked in IT for a company which had a lot of clients, in this company there was a SOC department which monitored many external networks. This was a 24/7 operation and as a virtualization systems engineer, we would get calls at all hours from this team. Then I started to wonder why are they there at this time, is technology not advanced enough to have autonomous systems, could they not check a report or a confusion matrix in the morning instead of working 24/7. How accurate would a neural network algorithm be monitoring alerts? Is it trust worthy? Would the company benefit financially from this?


# Project Overview

## Performing DDOS Attack from KALI Linux

To collect network traffic for our Neural Network to evaulate we will need to create a virtual environment between two VM's using virtual box.

Kali Linux is an open source, Debian-based Linux distribution geared towards various information security tasks, such as Penetration Testing, Security Research, Computer Forensics and Reverse Engineering. [4]
A prebuilt Kali Linux will be used to perform a DOS (denial of service) attack to a victim machine. To aim of this task is to target a victim target and have the network captured by a SIEM.

To perform the DOS attack, we will need a stresser tool, after some consideration between a Slowloris attack, HPing3 and LOIC(Low Orbit Ion Cannon), I have decided to work with LOIC. It works by flooding a target server with TCP, UDP or HTTP packets with the goal of disrupting service.

We download a Kali Linux image from https://www.kali.org/get-kali/#kali-virtual-machines and install 

![Kali Linux running vm](https://github.com/user-attachments/assets/ec9c3108-60f9-4463-9f3a-25135cc79c0e)


Launch the application

![loic image running ip](https://github.com/user-attachments/assets/1fbead3f-92d3-4009-9022-6aa157575434)


We can see the LOIC application UI. To perform a DOS attack on a victim VM, we enter the victim’s IP address, select Lock on, select our port and method type (TCP, UDP, HTTP) and select IMMA CHARGIN MAH LAZER.


### Windows Server 2022 Configuraton

Our victim machine will be a Windows server 2022. We download the windows server iso file from the Microsoft evaluation center.
Import the iso file into Virtualbox and configure all system hardware settings. The windows server will be assigned the minimum required hardware.
System hardware is 1 CPU, 25GB VHD, 4GB of memory.

To allow malicious network activity to the Windows VM we disable Windows Defender Firewalls and disable Microsft Defender AntiVirus in the group policy and as an extra step I allow inbound firewall rules.

![image](https://github.com/user-attachments/assets/827c1464-c137-47c0-a179-556fdcac3d48)


![image](https://github.com/user-attachments/assets/8020c781-9e0e-4784-82de-da0408600adb)


![image](https://github.com/user-attachments/assets/5447b7a4-ebf4-43b2-9043-c0cf1c2b47dc)


![image](https://github.com/user-attachments/assets/2f569527-2a27-4b1a-9294-8c538b9c3a6b)


### Solarwinds Security Event Manager

In this project Solarwinds Security Event Manager will be used to monitor the network traffic between the master (Kali Linux) and the victim (Windows server). The captured network traffic will be exported to a CSV file, pre-processed and passed to our neural network

A 30-day trial fully functional version of Solarwinds SEM is available to download as an OVF file.
After the OVF file is downloaded, it is imported into Virtualbox.

![image](https://github.com/user-attachments/assets/0444c2a1-428e-4845-b982-0bfba31f4c13)

The installer files for the Solarwinds agent is on the SEM server, we download and install for Linux and Windows

![image](https://github.com/user-attachments/assets/3012b446-951a-4187-8a21-713d09fe551d)

Register the Kali Linux VM and Windows Server 2022 in Solarwinds SEM Management Server

Note: The installtion for the Windows Server failed numerous times so I has to create a silent installer to get the device to register on the SEM Management Server

![image](https://github.com/user-attachments/assets/90e940b7-496c-4490-a2b9-63886cdca6d5)

Now the Virtual Machines are registered in the SEM Management Server, let's move on the Network configuration for Virtual Box.


### VirtualBox Network Configuration

To allow all VM’s to communication, I implemented a Host-only network, which can be thought of as a hybrid between bridged and internal networking modes.

With bridged networking, the virtual machines can talk to each other and the host as if they were connected through a physical Ethernet switch, as with Internal networking, a physical interface does not need to be present and the virtual machines cannot talk to the world outside since they are not connected to a physical networking interface. 

Host-only networking is particularly useful for preconfigured virtual appliances, where virtual machines are shipped together and designed to cooperate.
A host-only network allowed me create a DHCP server which would automatically assign an IP address in a configured range

![image](https://github.com/user-attachments/assets/a9b6c142-aebd-43eb-b37f-b82772c74325)

![image](https://github.com/user-attachments/assets/d5624eb9-51ae-4bb6-a866-c65bdba69e00)


### Denial of Service Attack

A Denial-of service (DoS) attack is an attack meant to shut down a machine or a network, making it inaccessible to its intended users. DoS attacks accomplish this by flooding the target with traffic or sending it information that triggers a crash

To attack the Windows server virtual machine from kali Linux virtual machine on VirtualBox, first we check if the machines can talk to each other, we do a ping test. 

![image](https://github.com/user-attachments/assets/178f93f5-b0e0-4d2f-b6d6-45b3941f7f8e)

Launch LOIC application on the Kali Linux machine from the terminal. We then decide what port and method (TCP, UDP, HTTP) we will attack. For this attack we will flood port 80 over UDP

![image](https://github.com/user-attachments/assets/5d212f4c-f238-47de-9a11-145a38afa8ca)

Check from Wireshark if the packets have been successfully sent to the windows machine. 

![image](https://github.com/user-attachments/assets/868ea1c5-dc3c-410c-8484-76239e6d2612)

Now we check the Windows server task manager to see if the attack was successful. If the attack was successful there should be a sharp jump on the machines resources and the virtual machine should not be usable

![image](https://github.com/user-attachments/assets/cbb9a019-9b5e-4d9c-93ec-9764482d8d5f)

The attack was successful. This attack would not register on the Solarwinds security event manager, I tried two more DoS attacks to register malicious traffic on the SEM server, using Slowloris and HPing3 but they were unsuccessful.


# Neural Network - Network Intrusion Detecion System


## TensorFlow and Keras

To show how our Neural Network would classify Network activity if the SEM worked, I will be building a Neural Network using the kkpcupp.data_10_percent.gz dataset

This database contains a standard set of data to be audited, which includes a wide variety of intrusions simulated in a military network environment.

I will build a Neural Network using this dataset to classify the Network activity, as this dataset has a large number of data entrys, I will be using google colab to run my model using GPU's.

In my coling example I will be using Tensorflow and Keras, which is a high-level API integrated with Tensorflow

Keras is the high level API of the TensorFlow platform. It provides an approachable, highly-productive interface for solving machine learning problems, with a focus on modern deep learning. Keras covers every step of machine learning workflow, from dta processing to hyperparameter tuning to deployment. It was developed with a focus on enabling fast experimentation.

A model is an object that groups layers together and that can be trained on data. The simplest type of model is the Sequential model, which is a linear stack of layers. Sequential model uses the Dense layer to build the model

The tf.keras.Model class features built-in training and evaluation models that are used through this project:

-Tf.keras.Model.fit: Trains the model for a fixed number of epochs.

-Tf.keras.Model.predict: Generates output predictions for the input samples.

-Tf.keras.Model.evaluate: Returns the loss and metrics values for the model, configured via the tf.keras.Model.compile method 


## Neural Network - Tensforlow

Preparing your data for the Neural Network is pivotal, below are the steps I will take to deliver the best possible results using our data set in a Neural Network.

1.	Pre-processing the data
2.	Visualizing the data
3.	Remove highly correlated columns
4.	Label encoding the features
5.	Building and training a Neural network
6.	Testing the Neural Network


## Dataset

There are 4,898,430 total entries in this data set, 3,883,370 are DoS (Denial of Service) attacks, 41,102 are Probe (probing attacks), 1126 are R2L (root to local), 52 are U2R (User to Root) and 972,780 are registered as Normal. 

To make it easier on my machine and to speed up the process I will be using the data set which contains 10% of the full data set (kddcup.data_10_percent)[22]. The data set entries are as follows Total entries 494,020, DoS 391,458, Probe 4107, U2R 52, and Normal 97,277

1.	Denial of Service: is an attack which disrupts a service through various stressing tools
2.	Root2Local: unauthorized access from a remote machine
3.	User2Root: unauthorized access to local superuser (root) privileges
4.	Probe: surveillance and invasive methods to bypass security measures








