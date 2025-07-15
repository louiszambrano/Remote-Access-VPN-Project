# Remote-Access-VPN-Project
A Cisco Packet Tracer project that sets up a secure Remote Access VPN using IPsec and ISAKMP. Includes router configuration, NAT, static routing, and VPN client setup to allow a remote employee to securely access the corporate network.


<img width="1049" height="771" alt="Screenshot 2025-07-14 at 10 07 32 PM" src="https://github.com/user-attachments/assets/659f4d81-3934-4bf8-a652-db1c8a277f1b" />

Here's what I did:
- Configured routers with static IPs & routing.
- Set up NAT on the router to allow outbound traffic to the internet.
- Built and verified static routes connectivity.
- Deployed a Remote Access VPN using ISAKMP and IPsec.
- Configured a VPN client to securely connect to the corporate network.
- Verified tunnel functionality using ipconfig, ping, and tracert

This project was a great practical experience in end-to-end VPN configuration, including encryption, authentication, and secure remote access using Cisco Packet Tracer. 


WALKTHROUGH
1. Configure Employees Router 
- Router>en
- Router# config t
- Router (config) #hostname HomeRouter
- HomeRouter (config) #int g0/0
- HomeRouter (config-if) #ip address 192.168.1.1 255.255.255.0
- HomeRouter (config-if) #no shutdown
- HomeRouter (config-if) #int se0/0/0
- HomeRouter (config-if) #ip address 209.165.200.226 255.255.255.252
- HomeRouter (config-if) #no shutdown

2. Configure Verizon (Internet) Router 
- Router>en
- #config t
- #Hostname VerizonRouter
- #int s0/0/0
- #ip address 209.165.200.225 255.255.255.252
- #clock rate 4000000
- #no shutdown
- #int s0/0/1
- #ip address 209.165.200.229 255.255.255.252
- #clock rate 400000
- #no shutdown

3. Configure SONY Router
- Router>en
- #config t
- #hostname SonyRouter
- #int g0/0
- #ip address 192.168.2.1 255.255.255.0
- #no shutdown
- #int s0/0/1
- #ip address 209.165.200.230 255.255.255.252
- #no shutdown

4. Configure Employees PC
- Click on Desktop > IP Configuration
- IP Address 192.168.1.10
- Default Gateway 192.168.1.1

5. Configure ABC Inc. PC
- Click on Desktop > IP Configuration
- IP Address 192.168.2.10
- Default Gateway 192.168.2.10

6. Configure Sony Server 
- IP Address 192.168.254
- Default Gateway 192.168.2.1

7. Configure Static Default Route on Employee Router 
- #config t
- #ip route 0.0.0.0 0.0.0.0 s0/0/0

8. Configure Static Default Route on ABC Inc. Router
- #config t
- #ip route 0.0.0.0 0.0.0.0 s0/0/1

9. Try to ping from Employee PC to Default Gateway 
- Click on Desktop > Command Prompt
- C:\>ipconfig
- C:\>ping 192.168.1.1
**Should be successful

10. Try to ping from Employee PC to Internet Router
- C:\>ping 209.165.200.225
**This ping should fail because the internet router isn’t configured with static routes or dynamic router for Employees network (192.168.1.0/24)

11. Try to ping from Employee PC to ABC Inc. Router 
- C:\>ping 209.165.200.230
**This ping should also fail because the internet router doesn’t know how to reach Employees network.

12. Configure NAT on Employees Router 
- #ip access-list standard ACL_NAT
- #permit 192.168.1.0 0.0.0.255
- #ip nat inside source list ACL_NAT interface s0/0/0 
- #int s0/0/0
- #ip nat outside
- #int g0/0
- #ip nat inside 

13. Try to ping from Employees PC to Internet Router
- C:\>ping 209.165.200.225
**Now it should be successful 

14. Try to ping from Employees PC to ABC Inc. Router 
- C:\>ping 209.165.200.230
**Now it should be successful also 

15.Try to ping from Employees PC to ABC INC. PC
- C:\>ping 192.168.2.1
**This ping would fail because the Internet Router doesn’t know how to reach the 192.168.2.0 network. 

16. Let create a Remote Access VPN for the Employee to gain access to the ABC Inc. network. 
- Click on Sony Router 
- #config t
- #license boot module c1900 technology-package securityk9
- ACCEPT? [yes/no]: yes
- #copy running-config startup-config 
- #reload 
- #config t
- #ip local pool PoolVPN 192.168.2.100 192.168.2.115
- #aaa new-model
- #aaa authentication login UserVPN local
- #aaa authorization network GroupVPN local
- #username employee1 secret ciscovpn1
- #crypto isakmp policy 100 
- #encryption aes 256
- #hash sha
- #authentication pre-share
- #group 5
- #lifetime 3600
- #exit
- #crypto isakmp client configuration group GroupVPN
- #key ciscogroupvpn
- #pool PoolVPN
- #exit
- #crypto ipsec transform-set SetVPN esp-aes esp-sha-hmac
- #crypto dynamic-map DynamicVPN 100
- #set transform-set SetVPN 
- #reverse-route
- #exit
- #crypto map StaticMap client configuration address respond 
- #crypto map StaticMap client authentication list UserVPN 
- #crypto map StaticMap isakmp authorization list GroupVPN
- #crypto map StaticMap 20 ipsec-isakmp dynamic DynamicVPN
- #int s0/0/1 
- #crypto map StaticMap


17. Access VPN from Employee PC
- Click on Desktop > VPN Configuration 
- Group Name: GroupVPN
- Group Key: ciscogroupvpn
- Host IP: 209.165.200.230
- Username: employee1
- Password: ciscovpn1
- Click on Connect 
**VPN connection should be successful with the ip address you assigned  

<img width="1895" height="941" alt="Screenshot 2025-07-14 at 10 15 36 PM" src="https://github.com/user-attachments/assets/3f915111-41a6-4ec2-9b3a-7c73c75f8dcb" />

18. Verify with VPN with Command Prompt on Employee PC
- Click on Desktop > Command Prompt 
- C:\>ipconfig all
** You should see a Tunnel Interface IP Address

19. Try to ping from Employee PC to Office PC
- C:\>ping 192.168.2.10
**Should be successful 

20. Try to ping from Employee PC to Sony Server 
- C:\>ping 192.168.2.254
**Should be successful

<img width="1904" height="767" alt="Screenshot 2025-07-14 at 10 16 45 PM" src="https://github.com/user-attachments/assets/0eaf7169-fb31-42bc-bdfb-c26c8e770d47" />

21. Test with Tracert command  
- C:\> tracert 192.168.2.10
**You should see Two Hoops. First hoop should be ABC Inc. Router (209.165.200.230) and the second hoop should be ABC Inc. PC. (192.168.2.10)
<img width="1900" height="766" alt="Screenshot 2025-07-14 at 10 16 58 PM" src="https://github.com/user-attachments/assets/f0c5754a-6336-47c0-810c-9e2218eea757" />


⚠️ Disclaimer: All router names, IP addresses, and configurations in this project are fictional and used for educational purposes only. This project is not affiliated with or based on the real network infrastructure of Sony, Verizon, or any other organization.

