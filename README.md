# AWS-BGP-SITE-TO-SITE-VPN

This project is a walkthrough of an **AWS BGP Site-to-Site VPN** setup, based on the original work by **Adrian Cantrill**, used under the MIT License.
It demonstrates how to establish a **highly available, BGP-based VPN connection** between AWS and an on-premises network. In this project, I will be creating an initial AWS environment with **2 subnets, 2 EC2 instances, a Transit Gateway (TGW), VPC attachments, and a default route pointing to the TGW.** The simulated on-premises environment will include **1 public subnet and 2 private subnets.** The public subnet hosts **two Ubuntu instances** running **StrongSwan** and acting as **VPN endpoints.** The goal of this setup is to demonstrate how BGP routing can be used to dynamically exchange routes between the AWS Transit Gateway and the on-premises routers, enabling seamless and resilient communication across the IPsec tunnels.

<br>

![image](https://github.com/user-attachments/assets/d32611a4-a5be-49de-ac97-5fa4d9fccf6d)
Original architecture diagram from [Adrian Cantrill](https://www.youtube.com/adriancantrill)

## STAGE ONE

- To first create the intial environments click on the provided [1-click deployment link](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/review?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/aws-hybrid-bgpvpn/BGPVPNINFRA.yaml&stackName=ADVANCEDVPNDEMO). Make sure to login to an AWS account with full admin permission and you're using the us-east-1 region. Once you clicked on the link scroll down to the bottom, check the box, and click create stack.
Make sure the stack has moved to a **create complete** state before moving on.

<br>

  - <img src="https://github.com/user-attachments/assets/fe0a0ee4-05fd-48fc-b10f-8dd9e12e4be9" alt="image" width="400" height="250"/> 
  
<br>

- Next we are going to be creating two customer gateway objects. These objects tell AWS how to connect with them and represent the phsyical on-premises routers. 
  - First open a new tab and move into the VPC console. Scroll down on the left bar menu and under **Virtual Private Network** click on **Customer Gateways** and then [Create Customer Gateways.](https://us-east-1.console.aws.amazon.com/vpcconsole/home?region=us-east-1#CreateCustomerGateway:)
  - Set the name to ONPREM-ROUTER1
  - Set the BGP ASN to 65016
  - Set the IP address to Router1Public's IP address under the CloudFormation Stack
  - Scroll down and click Create Customer Gateway
- We are now going to create the second customer gatweay object
  - Follow the same instructions as above but set the name to ONPREM-ROUTER2
  - Set the BGP ASN to 65016
  - Set the IP address to Router2Public's IP address under the CloudFormation Stack
  - Scroll down and click Create Customer Gateway
  - It should look like this before moving on with the IP addresses being adjust to your own
  
 ![image](https://github.com/user-attachments/assets/db70d8e0-d8f3-4413-93b9-1038e50339a2)

 - Now at this point we are going to verify that there is no connectivity between the AWS side and the on-premise side.
   - Move into the EC2 console and into instances
   - Click on AWS-EC2-B and copy the Private IPv4 address under details
   - Right click ONPREM-SERVER2 and click connect and use session manager to connect to the instance
   - Once connected run the command **ping IP_ADDRESS_OF_AWS_EC2-B** and there should be no response
   - Click terminate on Session Manager and that is the end of Stage 1.
   <img src="https://github.com/user-attachments/assets/f09cc632-dd6a-4899-ae5b-9844aea936af" alt="image" width="820" height="240"/>
   
<br>

This is what we have created so far

<br>

![image](https://github.com/user-attachments/assets/2d50f2f2-b47d-40db-9a01-775f79008df6)

<br>

## STAGE TWO

In Stage 2, we will create two VPN attachments for the Transit Gateway, resulting in two VPN connections—one for each Customer Gateway.
Each VPN connection includes two IPsec tunnels: One from AWS Endpoint A to the Customer Gateway, One from AWS Endpoint B to the Customer Gateway.

  - To start move into the VPC console and scroll down and click on [Transit Gateway Attachments](https://us-east-1.console.aws.amazon.com/vpcconsole/home?region=us-east-1#TransitGatewayAttachments:)
    - Click Create Transit Gateway Attachment
    - Set the Transit Gateway ID to A4LTGW and the attachment type to VPN
    - Select Existing for Customer Gateway
    - Click Customer gateway ID dropdown and select ONPREM-ROUTER1
    - Click Dynamic (requires BGP) for Routing options
    - Click Enable Acceleration
    - Click Create Attachment
    - Do the same process for ONPREM-ROUTER 2
  - Now scroll back up on the left bar and click [Site-to-Site VPN](https://us-east-1.console.aws.amazon.com/vpcconsole/home?region=us-east-1#VpnConnections:)
    - In another tab above **Site-to-Site VPN** open **Customer Gateways**
    - Now match the Customer Gateway ID from **Customer Gateways** to **Site-to-Site VPN**
![image](https://github.com/user-attachments/assets/8a551a30-da78-4c84-b842-800dbaf3bcfd)
![image](https://github.com/user-attachments/assets/e62cae85-23eb-4efd-aba9-0be1ffc22231)
<br>

- On **Site-to-Site VPN** right-click on the ONPREM-ROUTER1 VPN and click download configurations
  - Change the Vendor to Generic and click download
  - Find the file you just downloaded and rename it to **CONNECTION1CONFIG.TXT**
  - Now repeat this process for ONPREM-ROUTER 2 but rename the file to **CONNECTION2CONFIG.TXT**
- At this point, we will be extracting important configuration items from the two files we have just downloaded.
  - Open the CONNECTION1CONFIG.TXT and also this [document template](https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/aws-hybrid-bgpvpn/02_INSTRUCTIONS/DemoValueTemplate.md) which will help us note down important values.
  -  There will be detailed instructions inside the document template on how to extract all of the configuration variables you will need.
  -  Finish writing down all the values inside the document template.
  -  Do this for both CONNECTION1CONFIG.TXT and CONNECTION2CONFIG.TXT
  -  This will be the end of Stage 2.

 <br>
 
![image](https://github.com/user-attachments/assets/f2fe5b1d-958e-484d-8eae-b3fd7932c431)

<br>

## STAGE THREE
 
In Stage 3, we will configure each of the on-premises Ubuntu routers, running StrongSwan, to establish IPsec tunnels to AWS. Each router will create two IPsec tunnels, with each tunnel connecting to a different AWS VPN endpoint for redundancy and high availability. Before proceeding, ensure that both VPN connections in the AWS console are in the **Available** state.

- First move into the EC2 console and click on instances
  - Select ONPREM-ROUTER 1 right click and then connect
  - Connect using session manager
  - Once connected type in **sudo bash** to gain root permissions
  - Then type **cd /home/ubuntu/demo_assets/**
  - Type **nano ipsec.conf**
  - Using the documents template replace **ROUTER1_PRIVATE_IP, CONN1_TUNNEL1_ONPREM_OUTSIDE_IP, CONN1_TUNNEL1_AWS_OUTSIDE_IP, CONN1_TUNNEL1_AWS_OUTSIDE_IP, and, ROUTER1_PRIVATE_IP, CONN1_TUNNEL2_ONPREM_OUTSIDE_IP, CONN1_TUNNEL2_AWS_OUTSIDE_IP, CONN1_TUNNEL2_AWS_OUTSIDE_IP**
![image](https://github.com/user-attachments/assets/fe9ea227-ec48-4adc-bade-b18e827e8e5a)

  - Save your changes with **ctrl + o and ctrl + x** and then type **nano ipsec.secrets**
  - Then replace the values **CONN1_TUNNEL1_ONPREM_OUTSIDE_IP, CONN1_TUNNEL1_AWS_OUTSIDE_IP, CONN1_TUNNEL1_PresharedKey, CONN1_TUNNEL2_ONPREM_OUTSIDE_IP, CONN1_TUNNEL2_AWS_OUTSIDE_IP, CONN1_TUNNEL2_PresharedKey** with the document template values.
![image](https://github.com/user-attachments/assets/78c1eeee-2d5d-47cd-bfcf-30421be0bb7e)

  - Save your changes the same way and then type **nano ipsec-vti.sh**
  - Then replace the values **CONN1_TUNNEL1_ONPREM_INSIDE_IP (ensuring the /30 is at the end), CONN1_TUNNEL1_AWS_INSIDE_IP (ensuring the /30 is at the end), CONN1_TUNNEL2_ONPREM_INSIDE_IP (ensuring the /30 is at the end), CONN1_TUNNEL2_AWS_INSIDE_IP (ensuring the /30 is at the end)**
![image](https://github.com/user-attachments/assets/766f12bb-1f28-486e-ad4b-56592921b498)

  - Save your changes and then copy the three files to the etc directory using cp ipsec* /etc
  - Then to make the file executable use the command **chmod +x /etc/ipsec-vti.sh**
  - Then type **systemctl restart strongswan** to restart StrongSwan and type **ifconfig**
  - You should see two virtual tunnels vti1 and vti2 which means that the two tunnels are connected and active in AWS.
  - <img src="https://github.com/user-attachments/assets/b33dcfd4-eeef-4b19-b0b5-2764d9751164" width="550" height="300" alt="image"/>

<br>

- Now we are going to the do same thing for ONPREM-ROUTER2
  - Select ONPREM-ROUTER2 and right click and connect using Session Manager
  - Once connected type in **sudo bash** to gain root permissions
  - Then type **cd /home/ubuntu/demo_assets/**
  - Type **nano ipsec.conf**
  - Using the documents template replace **ROUTER2_PRIVATE_IP, CONN2_TUNNEL1_ONPREM_OUTSIDE_IP, CONN2_TUNNEL1_AWS_OUTSIDE_IP, CONN2_TUNNEL1_AWS_OUTSIDE_IP, ROUTER2_PRIVATE_IP, CONN2_TUNNEL2_ONPREM_OUTSIDE_IP, CONN2_TUNNEL2_AWS_OUTSIDE_IP, CONN2_TUNNEL2_AWS_OUTSIDE_IP**
  - Save your changes with **ctrl + o and ctrl + x** and then type **nano ipsec.secrets**
  - Then replace the values **CONN2_TUNNEL1_ONPREM_OUTSIDE_IP, CONN2_TUNNEL1_AWS_OUTSIDE_IP, CONN2_TUNNEL1_PresharedKey, CONN2_TUNNEL2_ONPREM_OUTSIDE_IP, CONN2_TUNNEL2_AWS_OUTSIDE_IP, CONN2_TUNNEL2_PresharedKey**
  - Save your changes the same way and then type **nano ipsec-vti.sh**
  - Then replace the values **CONN2_TUNNEL1_ONPREM_INSIDE_IP (ensuring the /30 is at the end), CONN2_TUNNEL1_AWS_INSIDE_IP (ensuring the /30 is at the end), CONN2_TUNNEL2_ONPREM_INSIDE_IP (ensuring the /30 is at the end), CONN2_TUNNEL2_AWS_INSIDE_IP (ensuring the /30 is at the end)**
  - Save your changes and then copy the three files to the etc directory using cp ipsec* /etc
  - Then to make the file executable use the command **chmod +x /etc/ipsec-vti.sh**
  - Then type **systemctl restart strongswan** to restart StrongSwan and type **ifconfig**
  - You should again see two virtual tunnels vti1 and vti2 which means that the two tunnels are connected and active in AWS.
  - <img src="https://github.com/user-attachments/assets/72ac29c3-8b04-4a09-8b20-9d6f7a049cd5" width="550" height="300" alt="image"/>

<br>

- This is the end of Stage 3. We have now connected the AWS VPC endpoint to the On premise routers.

<br>

  ![image](https://github.com/user-attachments/assets/c0b5f026-3e68-46cc-bab1-cf08fff69a27)

## STAGE FOUR

In Stage 4, we’ll build on the IPsec tunnels established in Stage 3 by configuring BGP (Border Gateway Protocol) sessions over each tunnel. These BGP sessions will enable the on-premises routers to dynamically exchange routing information with the AWS Transit Gateway. Once BGP is configured and routes are exchanged, traffic will be able to flow seamlessly between the AWS cloud and your on-premises network. To support BGP functionality, we will install and configure FRR (Free Range Routing) during this stage.

- To begin we will be installing FRR onto Router 1 and Router 2 with a pre-installed script.
  - Move into the **EC2** Console and right click on your **ONPREM-ROUTER 1** instance and connect through Session Manager.
  - Type **sudo bash** then **cd /home/ubuntu/demo_assets** then **chmod +x ffrouting-install.sh** to make the **ffrouting-install.sh** file executable.
  - The type **./ffrouting-install.sh** to run the install process. 
  - In a different **EC2** tab do the same process and commands with the **ONPREM-ROUTER2** instance connecting through Session Manager
  - Type **sudo bash** then **cd /home/ubuntu/demo_assets** then **chmod +x ffrouting-install.sh** to make the **ffrouting-install.sh** file executable.
  - The type **./ffrouting-install.sh** to run the install process.
  - This process will take about 10-20 minutes and we will need both installations to be completed before moving on.
  - Once completed you will type **vtysh** on Router 1 as we will be configuring BGP for the On Premise side of this architecture
  - Then **conf t**
  - **frr defaults traditional**
  - **router bgp 65016**
  - **neighbor CONN1_TUNNEL1_AWS_BGP_IP remote-as 64512**
  - **neighbor CONN1_TUNNEL2_AWS_BGP_IP remote-as 64512**
  - no bgp ebgp-requires-policy**
  - **address-family ipv4 unicast**
  - **redistribute connected**
  - **exit-address-family**
  - **exit**
  - **exit**
  - **wr**
  - **exit**
  - **sudo reboot**
- ONPREM-ROUTER1, once reactivated, will function as both an **IPSEC endpoint and a BGP endpoint**, facilitating route exchanges with the **AWS transit gateway.**
  - Now do the same process for Router 2
  - **vtysh**
  - Then **conf t**
  - **frr defaults traditional**
  - **router bgp 65016**
  - **neighbor CONN1_TUNNEL1_AWS_BGP_IP remote-as 64512**
  - **neighbor CONN1_TUNNEL2_AWS_BGP_IP remote-as 64512**
  - no bgp ebgp-requires-policy**
  - **address-family ipv4 unicast**
  - **redistribute connected**
  - **exit-address-family**
  - **exit**
  - **exit**
  - **wr**
  - **exit**
  - **sudo reboot**
- ONPREM-ROUTER2, once reactivated, will function as both an **IPSEC endpoint and a BGP endpoint**, facilitating route exchanges with the **AWS transit gateway.**
  - You can now verify if the tunnels are up by moving into the **VPC** console and then **Site-to-Site VPN connections** and click **Tunnel details**
  - You can also see this in **Transit Gateway Route Tables** and under **Routes**, all the different routes we have configured.
  - Finally we can also verify this by connecting to **ONPREM-SERVER2** via **Session Manager** and then running **ping IP_ADDRESS_OF_EC2-B**
  - And the other way around connect to **EC2-B** via **Session Manager** and then running **ping IP_ADDRESS_OF_ONPREM-SERVER2**

 <br>
    
- This is the end of Stage 4 and the final architecture that has been created.

<br> 

  ![image](https://github.com/user-attachments/assets/3b3f6b3f-836e-4b1a-92e3-71360bda7e92)

## Stage Five

In Stage 5 we will be returning our AWS account back to how it was before this project

- First we will Return move to the [VPC Console](https://us-east-1.console.aws.amazon.com/vpcconsole/home?region=us-east-1#VpnConnections:sort=VpnConnectionId)
- Then delete both VPN Connections
- Move to [Customer Gateways](https://console.aws.amazon.com/vpc/home?region=us-east-1#CustomerGateways:sort=CustomerGatewayId)
- Delete both Customer Gateways you created earlier
- Open a new tab into [CloudFormation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/)
- Delete the ONPREM stack and wait for the VPN connections to full delete
- Finally delete the AWS Stack










    

























## ACKNOWLEDGEMENTS

This project is a walkthrough based on the original work by **Adrian Cantrill**, used under the MIT License.  
Check out his content: [Adrian Cantrill’s Website](https://learn.cantrill.io/)  
All credit for the original concept and implementation goes to him. 
