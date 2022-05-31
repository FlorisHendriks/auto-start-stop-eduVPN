# Introduction
[eduVPN](https://www.eduvpn.org) is used so that organisations can authenticate and authorize their employees before they can access organisational resources.

When a user In order to establish an eduVPN connection with the default settings, the user has to run the eduVPN client and authenticate (if the previous VPN connection is expired).

However, sometimes the user tries to access the organisational resources without a eduVPN connection. The user is prompted with a generic error message and is confused why he or she can not access the resource. As a result, the user often contacts the eduVPN support and the support tells the users to establish the VPN connection. These kind of trivial issues can cause unnecessary overhead for the eduVPN support team.  


The Windows eduVPN developer has partially mitigated this limitation by offering the option to start the eduVPN client on sign-on. As long as the VPN configuration has not expired, the eduVPN client is able to establish the VPN connection automatically. However, this option is not offered by default. Moreover, if one enables this option for every VPN user the amount of concurrent VPN users can increase tremendously, as the VPN is on most of the time.

We therefore need to find a way in order to automatically start and stop eduVPN so that we minimalize unnecessary overhead for the eduVPN support team and the eduVPN server. 

In order to explore the technical paths to make start and stop automatically, we look at established VPN solutions  (e.g. [StormShield VPN SSL](https://vpn.stormshield.eu/) and [Pulse Secure](https://www.pulsesecure.net/products/remote-access-overview). Unfortunately, these solutions are closed source, but by reading their documentation, we can have a rough idea of what kind of methods they support to make their VPN automatically start and stop. We then try to translate these authentication methods within the context of eduVPN.

# VPN auto-triggered options
[Windows has multiple ways to automatically establish the VPN connection.](https://docs.microsoft.com/en-us/windows/security/identity-protection/vpn/vpn-auto-trigger-profile) 
Based on an event 
## App trigger
We have the ability to trigger the VPN based on an app that is used, 


## Name-based trigger
DNS resolution, if the user uses an untrusted network connection

## Always On

## Untrusted network

# Split tunneling

