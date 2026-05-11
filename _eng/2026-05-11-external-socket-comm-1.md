---
title: (Python) Socket Communication Between External IPs Using GET/POST Requests - Part 1
date: 2026-05-11 09:00:00 +0900
categories: [python, socket communication]
tags: [socket, socket communication, python, ip]
---

# (Python) Socket Communication Between External IPs Using GET/POST Requests - Part 1

## Introduction

### Overview

Socket communication between external IPs is much more complicated than communication within a local network. This is mainly due to security-related issues.

In a local network, communication usually happens through addresses like `192.168.0.xxx`. This is a virtual network IP, and you can think of it as an address created inside a single router. In other words, one actual IP address is virtually divided into multiple internal IP addresses using the `192.168.0.xxx` format.

Put differently, even if `192.168.0.7` and `192.168.0.13` communicate with each other through sockets, they are not communicating over the external network. The communication is happening inside a single IP network. Therefore, since the communication is essentially happening within your own local network, there are usually no serious security issues.

If you are only communicating inside a local network, you can easily write the code using only Python’s built-in `socket` module. However, communication between external IPs is different.

### Prerequisites

For communication between external IPs, using other modules makes the implementation simpler and faster. Before writing the Python code, we need the following modules. Install them as shown below.

```shell
pip3 install Flask    # or pip
pip3 install requests # or pip
```

## Environment Setup

### Implementation Environment

To communicate between external IPs, I needed devices using different IP addresses. To build this environment, I prepared a desktop PC connected through Ethernet LAN, and a MacBook as the other device.

In particular, the MacBook was not connected through Ethernet. Instead, I connected it through a phone hotspot so that it would have a different external IP address. For the server, I recommend using a wired LAN connection.

> SERVER: Desktop PC (Windows & Ethernet) 
> CLIENT: Laptop (macOS & Hotspot)

### [SERVER] Checking the Global IP

To check your actual external IP, you need to check your global IP. You can check it through this [website](https://myexternalip.com). Your IP address will be displayed very clearly.

For convenience, let’s assume my external IP addresses are `111.222.33.123` for the server and `55.66.77.8` for the client.

### [SERVER] Router Port Forwarding

To perform router port forwarding, you first need to access the router closest to you. To access the gateway router, you need to know the IP address of the gateway router.

In general, you can access it through `192.168.0.1` or `192.168.1.1`. You can simply enter this address in your browser.

However, if you are using a static IP or a shared building-wide internet connection, the address may be different. In that case, you need to find the gateway router IP manually. You can easily find it with a single command.

**Windows version**

```powershell
ipconfig
```

**macOS version**

```bash
route -n get default
```

In the result, find the part that says something like gateway. If you look for something in an IP address format, it should be easy to find. For example, you may see something like `192.168.99.1`. Only the C-class part of the IP address, which is the third part of the address, may be different. In this example, that part is `99`.

My actual gateway router address is not `192.168.99.1`, but I will use this address as an example. If you are using a LAN cable from your router or a wireless network from your router, you will probably be able to access it through `192.168.0.1`.

Once you access the gateway router page, a login page will appear. Log in. If this is your first time accessing it, the page will usually explain how to log in.

After logging in, you need to find the port forwarding tab. Since this differs depending on the internet provider, you may just need to look around a bit. It should not take too long.

For **SK Broadband**, it is located under `Firewall > Port Forwarding`. If you are using your own router, for example an ipTIME router, it is located under `Advanced Settings > NAT/Router Management > Port Forwarding Settings`.

Now, open a specific port on this router. Based on **SK Broadband**, I entered the information as shown in the table below. You can choose whichever port you want.

I am providing this information as a table because the UI differs slightly depending on the internet provider. However, the required information is usually almost the same.

The service port refers to the range of ports to be serviced. In my case, I only planned to use one port, so I did not specify a range of multiple ports. It is also better to match the internal port number. You also need to find your internal IP address manually.

On Windows, you can find it using the `ipconfig` command mentioned above. For macOS, check the command below the table.

| Service Port | Protocol | Internal IP Address | Port | Description |
| ------------ | -------- | ------------------- | ---- | ----------- |
| 5000-5000    | TCP+UDP  | 192.168.99.123      | 5000 | Note        |

> Checking the internal IP address on macOS
>
> ```bash
> ifconfig | grep inet
> ```
>
> Among the resulting values, find the one that looks like `192.168.xxx.xxx`. This is the internal IP address currently being used by your device.
> {: .prompt-tip }

If you configure the server side using Wi-Fi, this internal IP address may keep changing. Therefore, even if you are using a laptop, I recommend connecting it through a wired LAN cable from the router.

In the case of Wi-Fi, the IP address may keep changing through DHCP, so using a wired LAN connection helps keep it fixed. Otherwise, the address may change and cause connection errors.

If you have successfully followed everything up to this point, port forwarding is now complete.

## Continued in the Next Part

I will end this post here and continue in the next post.