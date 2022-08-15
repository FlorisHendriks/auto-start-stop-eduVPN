# Introduction
If someone connects to a public network with a device, they are putting themselves at risk. Malicious attackers can capture their network traffic and read private and sensitive information. With a Virtual Private Network (VPN), one can defend themselves against these attackers as it can encrypt the network traffic. Many IT organisations offer VPN services, one of which is [SURF](https://www.surf.nl/en/about-surf). SURF is a cooperative association of Dutch educational and research institutions. In 2014, SURF started a new VPN service called [eduVPN](https://www.surf.nl/en/eduvpn/about-eduvpn?dst=n1173) with the goal to replace typical closed source VPNs with an open source audited alternative. Nowadays, eduVPN is often used by students, researchers, and employees of educational institutions worldwide

Users often have to connect to eduVPN in order to get access to the resources of an organisation. To establish an eduVPN connection with the default settings, the user has to run the eduVPN client and authenticate (if the previous VPN connection is expired). 

However, sometimes the user forgets to establish the eduVPN connection and tries to access the organisation's resources without one. The user is prompted with a generic error message and is confused why he or she can not access the resource. As a result, the user often contacts the eduVPN support regarding the issue. The eduVPN support points out to the user that they need to establish the eduVPN connection in order to get access to the organisation's resources. These kind of trivial support requests can cause unnecessary overhead for the eduVPN support team.

The Windows eduVPN developer has partially mitigated this limitation by offering the option to start the eduVPN client on sign-on. As long as the VPN configuration has not expired, the eduVPN client is able to establish the VPN connection automatically. However, this option is not offered by default. Moreover, if one enables this option for every VPN user the amount of concurrent VPN users can increase tremendously, as the VPN is on most of the time.

It would therefore be useful to find a way in order to automatically start and stop eduVPN so that we minimalize unnecessary overhead for the eduVPN support team and the eduVPN server. This leads to the main research question:

**How can we make eduVPN automatically start and stop?**

We will limit the scope of our study to eduVPN users that have bring your own devices. [For managed devices users we can make eduVPN a system VPN that is always on.](https://github.com/FlorisHendriks98/HTTP_bulk_provisioning) Users are always connected to the eduVPN and therefore do not request organisation's resources without one.


# Technical paths

In order to explore the technical paths to make eduVPN start and stop automatically, we will first take a look at other vpn solutions. When searching for OpenVPN or WireGuard client implementations we did not find any [auto start / stop solutions](https://forums.openvpn.net/viewtopic.php?t=32426).

We did find out that NordVPN has such [functionality](https://nordvpn.com/blog/automatic-vpn-wifi-connect/). The app has the option to initiate a VPN connection either only on secured networks, only on wifi-networks or always. Unfortunately we can not take a detailed look at how they have implemented this as it is closed source. Our guess is that NordVPN installs a service on the client's computer that listens for event id 10000. Event id 10000 is a Windows log that gets logged whenever a user connects to a network. If the user sets the option to intiate a VPN connection for example to only on secured networks then the service also checks if the network is in the public category. 

![image](https://user-images.githubusercontent.com/47246332/184669417-5c226550-e845-4ae3-8cef-af6ade46442a.png)


Windows' built-in VPN documentation does show how it can automatically trigger a VPN connection.
## VPN auto-triggered options
Windows has the possibility to built-in a VPN. It has support for VPN protocols such as IKEv2 and SSTP. However, eduVPN only supports WireGuard and OpenVPN so it can not integrate with Windows built-in VPN functionality.
[Intrestingly, Windows has multiple ways to automatically start and stop a VPN connection.](https://docs.microsoft.com/en-us/windows/security/identity-protection/vpn/vpn-auto-trigger-profile)
Windows gives one the ability to define triggers, events that determine whether or not a VPN connection should be established.
### Application trigger
Microsoft offers the ability to trigger a Windows built-in VPN based on the application that is used.
### Name-based trigger
In windows one also has ability to activate a VPN based on specific or all DNS queries.

This can be useful to implement in eduVPN. Instead of routing network packets via a specific ip adress it can be handy to do it via DNS as it can handle IP changes. 

### Always On
Windows has the ability to enable the VPN when the user signs in, when there is a network change and when the device screen is on. 

eduVPN already has the ability to start on sign-on. One can also extend this functionality for network changes and when the device screen is on. If the user changes to the public network we want to have the VPN route all the traffic and when we are on corporate network we might want to route less over the VPN or perhaps turn the VPN off.

### Untrusted network
Lastly, the Windows built-in VPN can detect if the network is trusted or not. Windows retrieves a list of DNS suffixes and checks if it matches the network name of the physical interface connection profile.

## Split tunneling
We can define a lot of triggers (e.g. for a specific application, a set of DNS queries, detect if we are on corporate network) to start and stop eduVPN. But in order to realize this we need to monitor quite a lot from the client computer. We wonder if this is feasible and if virus scanners will not obstruct this implementation. Instead of using triggers to determine when we start or stop the VPN we can take a different technical path. Using split tunneling, we can specify what network traffic goes via the VPN tunnel and what traffic does not.

Using eduVPN's sign-on functionality we can have eduVPN always-on for a large portion of the time. Whenever the user is on a password protected network we will only route traffic for organisational resources over the VPN. All other traffic will be routed over the regular interface in order to alleviate the resources of the eduVPN server. However, if the user is on a public network we will route all the traffic over the VPN as otherwise all traffic can be easily eavesdropped.

We are going to determine whether the user is on a public network or not by looking at the wlan interface that is currently used. If the wlan interface is authenticated by ... we know that the user is on a public network. We then retrieve a configuration where we route all traffic over the VPN. If the wlan is authenticated with a different protocol we will retrieve a configuration where we partially rout traffic over the VPN.

If the user switches network we also need to check if the user switches to a public network. We therefore listen to event id 10000. Event id 1000 is created by Windows whenever we connect to a different network.
