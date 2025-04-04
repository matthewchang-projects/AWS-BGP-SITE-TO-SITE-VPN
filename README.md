# AWS-BGP-SITE-TO-SITE-VPN

This project is a walkthrough of an **AWS BGP Site-to-Site VPN** setup, based on the original work by **Adrian Cantrill**, used under the MIT License. 
It demonstrates how to establish a **highly available, BGP-based VPN connection** between AWS and an on-premises network. 
In this project I will be creating an initial AWS environment with 2 subnets, 2 EC2 instances, a TGW and VPC attachment and a default route pointing at the TGW. 
The simulated on-premises environment - 1 public subnet, 2 private subnets. The public subnet has 2 Ubuntu + strongSwan + Free VPN endpoints.

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

In Stage two we will be will be creating two VPN attachments for the Transit Gateway. This has the effect of creating two VPN connections, 1 for each of the customer gateways. Each connection has 2 Tunnels: one between AWS Endpoint A => Customer Gateway and one between AWS Endpoint B => Customer Gateway.

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
- 



      

























## ACKNOWLEDGEMENTS

This project is a walkthrough based on the original work by **Adrian Cantrill**, used under the MIT License.  
Check out his content: [Adrian Cantrill’s Website](https://learn.cantrill.io/)  
All credit for the original concept and implementation goes to him. 
