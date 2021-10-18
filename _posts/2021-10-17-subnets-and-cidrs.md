---
layout: post
title: Subnets and CIDRs
categories: Networking
---

Recently I faced several people struggling with subnet IP range conflicts, so I'll try to explain how CIDRs works and how to prevent conflicts on enterprise networks.

Well, networks are part of our everyday life for a while now, but the way it works is almost 'magic' for most people. You usually don't need to configure your router, it's just a plug-n-play device that you bought and suddenly you have a Wi-Fi network on your house.

When talking about enterprise, this kind of magic does not work the same way, usually, because we need to worry about how many IP addresses will be available to which network, how it will interact with other networks, and which IP ranges are available for we to use globally. Here is when CIDRs come into play.

## CIDRs

IP addresses are composed of two parts, a network prefix identifier, and the host's identifier. With CIDRs you can express both values in the same notation.

CIDR notation specifies an IP address followed by a slash ('/') and a decimal number representing the number of bits in the network mask.

Example: 192.168.0.0/16

So, what does it mean when I say a network has this CIDR? and what do those numbers mean?

**192.168.0.0/16**

As with everything in computers, each decimal number here is a representation of a sequence of binary numbers. If you are not familiar with the binary notation, here's a brief explanation:

Binary notation only uses 1 and 0, and depending on its position this value represents a 2ˆx value.

Here we have an 8 length binary number (11000000) and on top of each digit there is the decimal value it is representing:

```
 128 64 32 16 8  4  2  1
 1   1  0  0  0  0  0  0
```

Here is the easy part, for each number **1** on the binary representation, you just need to sum the decimal values on top of it to know its value.

So **11000000** translates to **128 + 64 = 192**.

Now it's easy to understand the IP notation, and why its easier for us to just write:

```
192.168.0.0
```

instead of:

```
11000000.10101000.00000000.0000000
```

Ok, the hard part is already gone, but what it means by the `/16`?

In CIDR notation, this decimal number means how many bits are part of the network mask. Starting from the left, we know that the first 16 bits are the network prefix and the remaining 16 bits are available for all hosts.

Then we have:

```sh
# First IP
192.168.0.0

11000000.10101000.00000000.00000000

# Last IP
192.168.255.255

11000000.10101000.11111111.11111111
```

We have some tools to make this calc for us, like `ipcalc`. 
Also, it's way easier to understand seeing its output:


![ipcalc]({{site.baseurl}}/post_images/2021-10-17/ipcalc.png)

## Usable IPs
 
Just a final note, in subnets not all IPs are usable, the first and the last Ip for each subnet are reserved for the gateway and the broadcast respectively.

So for example, in a `/30` subnet, we have **4** available IPs, but just **2** of them are usable and can be assigned to our hosts. So if you need a subnet that needs to accept **3** hosts, you must define at least a `/29` CIDR.

The exception to this rule is a `/32` CIDR that is just 1 usable IP, and as you might have observed, a `/31` CIDR has no usable IPs.

| CIDR | Subnet Mask     | Wildcard Mask   | Total IPs     | Usable IPs    |
|------|-----------------|-----------------|---------------|---------------|
| /32  | 255.255.255.255 | 0.0.0.0         | 1             | 1             |
| /31  | 255.255.255.254 | 0.0.0.1         | 2             | 0             |
| /30  | 255.255.255.252 | 0.0.0.3         | 4             | 2             |
| /29  | 255.255.255.248 | 0.0.0.7         | 8             | 6             |
| /28  | 255.255.255.240 | 0.0.0.15        | 16            | 14            |
| /27  | 255.255.255.224 | 0.0.0.31        | 32            | 30            |
| /26  | 255.255.255.192 | 0.0.0.63        | 64            | 62            |
| /25  | 255.255.255.128 | 0.0.0.127       | 128           | 126           |
| /24  | 255.255.255.0   | 0.0.0.255       | 256           | 254           |
| /23  | 255.255.254.0   | 0.0.1.255       | 512           | 510           |
| /22  | 255.255.252.0   | 0.0.3.255       | 1024          | 1022          |
| /21  | 255.255.248.0   | 0.0.7.255       | 2048          | 2046          |
| /20  | 255.255.240.0   | 0.0.15.255      | 4096          | 4094          |
| /19  | 255.255.224.0   | 0.0.31.255      | 8192          | 8190          |
| /18  | 255.255.192.0   | 0.0.63.255      | 16,384        | 16,382        |
| /17  | 255.255.128.0   | 0.0.127.255     | 32,768        | 32,766        |
| /16  | 255.255.0.0     | 0.0.255.255     | 65,536        | 65,534        |
| /15  | 255.254.0.0     | 0.1.255.255     | 131,072       | 131,070       |
| /14  | 255.252.0.0     | 0.3.255.255     | 262,144       | 262,142       |
| /13  | 255.248.0.0     | 0.7.255.255     | 524,288       | 524,286       |
| /12  | 255.240.0.0     | 0.15.255.255    | 1,048,576     | 1,048,574     |
| /11  | 255.224.0.0     | 0.31.255.255    | 2,097,152     | 2,097,150     |
| /10  | 255.192.0.0     | 0.63.255.255    | 4,194,304     | 4,194,302     |
| /9   | 255.128.0.0     | 0.127.255.255   | 8,388,608     | 8,388,606     |
| /8   | 255.0.0.0       | 0.255.255.255   | 16,777,216    | 16,777,214    |
| /7   | 254.0.0.0       | 1.255.255.255   | 33,554,432    | 33,554,430    |
| /6   | 252.0.0.0       | 3.255.255.255   | 67,108,864    | 67,108,862    |
| /5   | 248.0.0.0       | 7.255.255.255   | 134,217,728   | 134,217,726   |
| /4   | 240.0.0.0       | 15.255.255.255  | 268,435,456   | 268,435,454   |
| /3   | 224.0.0.0       | 31.255.255.255  | 536,870,912   | 536,870,910   |
| /2   | 192.0.0.0       | 63.255.255.255  | 1,073,741,824 | 1,073,741,822 |
| /1   | 128.0.0.0       | 127.255.255.255 | 2,147,483,648 | 2,147,483,646 |
| /0   | 0.0.0.0         | 255.255.255.255 | 4,294,967,296 | 4,294,967,294 |

