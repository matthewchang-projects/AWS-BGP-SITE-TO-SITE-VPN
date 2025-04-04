# AWS-BGP-SITE-TO-SITE-VPN

This project is a walkthrough of an **AWS BGP Site-to-Site VPN** setup, based on the original work by **Adrian Cantrill**, used under the MIT License. 
It demonstrates how to establish a **highly available, BGP-based VPN connection** between AWS and an on-premises network. 
In this project I will be creating an initial AWS environment with 2 subnets, 2 EC2 instances, a TGW and VPC attachment and a default route pointing at the TGW. 
The simulated on-premises environment - 1 public subnet, 2 private subnets. The public subnet has 2 Ubuntu + strongSwan + Free VPN endpoints.

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
   - <img src="https://github.com/user-attachments/assets/f09cc632-dd6a-4899-ae5b-9844aea936af" alt="image" width="820" height="240"/>


## STAGE TWO



























## ACKNOWLEDGEMENTS

This project is a walkthrough based on the original work by **Adrian Cantrill**, used under the MIT License.  
Check out his content: [Adrian Cantrill’s Website](https://learn.cantrill.io/)  
All credit for the original concept and implementation goes to him. 
