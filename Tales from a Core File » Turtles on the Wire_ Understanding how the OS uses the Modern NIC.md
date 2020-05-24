Tales from a Core File » Turtles on the Wire: Understanding how the OS uses the Modern NIC

# Turtles on the Wire: Understanding how the OS uses the Modern NIC

The modern networking card (NIC) has evolved quite a bit from the simple Ethernet cards of yesteryear. As such, the way that the operating system uses them has had to evolve in tandem. Gone are the simple 10 Mbit/s copper or ([BNC](https://en.wikipedia.org/wiki/BNC_connector)) devices. Instead, 1 Gb/s is common-place in the home, 10 Gb/s rules the server, and you can buy cards that come in speeds like 25 Gb/s, 40 Gb/s, and even 100 Gb/s! Speed isn’t the only thing that’s changed, there’s been a big push to virtualization. What used to be one app on one server, transformed into a couple apps on a few Hardware Virtual Machines, and these days can be hundreds of containers all on a single server.

For this entry, we’re going to be focusing on how the *Operating System* sees NICs, what abstractions they provide together, how things have changed to deal with the increased tenancy and performance demands, and then finally where we’re going next with all this. We’re going to focus on where scalability problems have come about and talk about how they’ve been solved.

## The Simple NIC

While this is a broad generalization, the simplest way to think of a networking card is that it has five primary pieces:

1. A MAC Address that it can use to filter incoming packets.
2. A ring, or circular buffer, that packets are received into from the network.
3. A ring, or circular buffer, that packets are sent from to the network.
4. The ability to generate interrupts.

5. A way to program all of the above, generally done with PCI memory-mapped registers.

First, let’s talk about rings. Both of these rings are considered a type of [circular buffer](https://en.wikipedia.org/wiki/Circular_buffer). With a circular buffer, the valid region of the buffer changes over time. Circular buffers are often used because of the property that they consume a fixed amount of memory and they handle the model of a single producer and a single consumer rather well. One end, the producer, places data in the buffer and moves a head pointer, while another end, the consumer, removes data from the buffer, moving the tail pointer. For the rest of this article, we won’t be using the term circular buffer, but instead **ring**, which is commonly used in both networking card programming manuals and operating systems to describe a circular buffer used for packet descriptors.

Let’s go into a bit more detail about how this works. Rings are the lifeblood of a networking card. They occupy a fixed chunk of memory in normal system memory and when the hardware wants to access it, it will perform [DMA](https://en.wikipedia.org/wiki/Direct_memory_access), direct memory access. All the data that comes and goes through it is described by an entry in a ring, called a descriptor. The ring is made of a series of these descriptors. A descriptor is generally made up of three parts:

1. A buffer address
2. A buffer length
3. Metadata about the packet

Consider the receive ring. Each entry in it describes a place that the hardware can place an incoming packet. The buffer in the descriptor says where in main memory to place the packet and the length says how big that buffer is. When a packet arrives, a networking card will generally generate an interrupt, thus letting the operating system know that it should check the ring.

The transmit side is similar, but slightly different in orientation. The OS places descriptors into the ring and then indicates to the networking card that it should transmit those packets. The descriptor says where in memory to find the packet and the length says how long the packet is. The metadata may include information such as whether the packet is broken up into one or more descriptors, the type of data in the packet, or optional features to perform.

To make this more concrete, let’s think of this in the context of receiving packets. Initially, the ring is empty. The OS then fills all of the descriptors with pointers to where the hardware can put packets it receives. The OS doesn’t have to fill the entire ring, but it’s standard practice in most operating systems to do so. For this example, we’ll assume that the OS has filled the entire ring.

Because it’s written to the entire ring, the OS will set its pointer to the start of the buffer. Then, as the hardware receives packets, the hardware will bump its pointer and send the OS an interrupt.

The following ASCII art image describes how the ring might look after the hardware has received two packets and delivered the OS an interrupt:

	        +--------------+ <----- OS Pointer
	        | Descriptor 0 |
	        +--------------+
	        | Descriptor 1 |
	        +--------------+ <----- Hardware Pointer
	        | Descriptor 2 |
	        +--------------+
	        |     ...      |
	        +--------------+
	        | Descriptor n |
	        +--------------+

When the OS receives the interrupt, it reads where the hardware pointer is and processes those packets between its pointer and the hardware’s. Once it’s done, it doesn’t have to do anything until it prepares those descriptors with fresh buffers. Once it does, it’ll update its pointer by writing to the hardware. For example, if the OS has processed those first two descriptors and then updates the hardware, the ring will look something like:

	        +--------------+
	        | Descriptor 0 |
	        +--------------+
	        | Descriptor 1 |
	        +--------------+ <----- Hardware Pointer, OS Pointer
	        | Descriptor 2 |
	        +--------------+
	        |     ...      |
	        +--------------+
	        | Descriptor n |
	        +--------------+

When you send packets, it’s similar. The OS fills in descriptors and then notifies the hardware. Once the hardware has sent them out on the wire, it then injects an interrupt and indicates which descriptors it’s written to the network, allowing the OS to free the associated memory.

### MAC Address Filters and Promiscuous Mode

The missing piece we haven’t talked about is the filter. Just about every networking card has support for filtering incoming packets, which is done based on the destination MAC address of the packet. By default, this filter is always programmed with the MAC address of the card. By default, if the destination MAC address of the Ethernet header doesn’t match the networking card’s MAC address, and it isn’t a [broadcast](https://en.wikipedia.org/wiki/Broadcasting_(networking)) or [multicast](https://en.wikipedia.org/wiki/Multicast) packet, then it will be dropped.

When networking cards want to receive more traffic than that which they’ve been programmed to receive, they’re put into what’s called `promiscuous mode`, which means that the card will place every packet it receives into the receive ring for the operating system to process, regardless of the destination MAC address.

Now, the question that comes to mind, is why do cards have this filtering capability at all? Why does it matter? Why would we only care about a subset of packets on the network?

To answer this, we first have to go back to the world before [network switches](https://en.wikipedia.org/wiki/Network_switch), there were only [hubs](https://en.wikipedia.org/wiki/Ethernet_hub). When a packet came into a hub, it was replicated to **every** port. This mean that every NIC would end up seeing everything that went out. If they hadn’t filtered by default, then the sheer amount of traffic and interrupts might have overwhelmed many early systems, particularly on larger networks. Nowadays, this isn’t quite as problematic as we use switches, which learn which MAC addresses are behind which switch ports. Of course, there are limits to this, which have motivated a lot of the work around network virtualization, but more on that in a future blog post.

## Problem: The Single Busy CPU

In the previous section, we talked about how we had an interrupt that fired whenever packets were successfully received or transmitted. By default, there was only ever a single interrupt used and that interrupt vector usually mapped to exactly one CPU, in other words only one CPU could process the initial stream of packets. In many systems, that CPU would then process that incoming packet all the way through the TCP/IP stack until it reached a socket buffer for an application to read.

This has led to many problems. The biggest are that:

1. You have a single CPU that’s spending all its time handling interrupts, unable to do user work

2. You might often have increased latency for incoming packets due to the processing time

These problems were especially prevalent on single CPU systems. While it may be hard to remember, for a long time, most systems only had a single socket with a single CPU. There were no hardware threads and there were no cores. As multi-processing became more common (particularly on x86), the question became how do networking cards take advantage of horizontal scalability.

### A Swing and a Miss

The first solution that springs to mind when we talk about this is say why don’t we just have multiple threads process the same ring? If we have multiple CPUs in the system, why not just have a thread we can schedule to help do some of the heavy lifting along with the thread that’s processing the interrupt?

Unfortunately, this is where [Amdahl](https://en.wikipedia.org/wiki/Amdahl%27s_law) starts to rear his head, glaring at us and reminding us of the harsh truths of multi-processing. Multiple threads processing the same queue doesn’t really do us much good. Instead, we’ll end up changing where our contention is. Instead of having a single CPU 100% busy, we’ll have several CPUs, quite busy, and spending a lot of their time blocked on locks. The problem is that we still have shared state here — the ring itself.

There’s actually another somewhat nastier and much more subtle problem here: packet ordering. While [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) has a lot of logic to deal with out of order packets and UDP users are left on their own, no one likes out of order packets. In many TCP stacks, whenever a packet is detected to have arrived out of order, that sends up red flags in the stack. This will cause TCP to believe there is something wrong in the network and often throttle traffic or require data to be retransmitted, injecting noticeable latency and performance impacts.

Now, if the packets arrive out of order on the networking card, then there’s not a lot we can do here. However, if they arrive in order and we have multiple threads processing the same ring, then due to lock ordering and the scheduler, it all of a sudden becomes very easy to have packets arrive out of order, leading to TCP throwing its hands up, and performance being left behind in the gutter.

So we now actually have two different problems:

1. We need to make sure that we don’t allow packets to be delivered out of order

2. We want to avoid sharing data structures protected by the same lock

### Nine Rings for Packets Doomed to be Hashed

The answer to this problem is two-fold. Earlier we said that a ring is the lifeblood of a NIC, so what if we just put added a bunch more rings to the card. If each ring has its own interrupt associated with it, then we’ve solved our contention problem. Each ring is still processed by only a single thread. The fastest way to deal with shared resources is not to share at all!

So now we’re left with the question of how do we deal with the out of order packet problem. If we simply assigned each incoming packet to a ring in a round-robin fashion, we’d only be making the problem of out of order delivery a certainty. So this means that we need to do something a little bit smarter.

The important observation is that what we care about is that a given TCP or UDP connection always go to the same place. It’s a TCP connection that can become out of order. If there are two different connections ongoing, the order that their packets are received in doesn’t matter. Only the order of a single connection matters. Now all we need to do is assign a given connection to a ring. For a given TCP connection, the source and destination IP addresses, and the source and destination ports will be the same throughout the lifetime of the connection. You might sometimes hear this called a flow, a series of identifying information that identifies some set of traffic.

The way the assignments are made is by **hashing**. Networking cards have a hash function that takes into account various fields in the header that are constant and use them to produce a hash value. Then that hash value is used to assign something to a ring. For example, if there were four rings in use, a simple way to assign traffic is to simply take the hash value and compute its modulus by the number of rings, giving a constant ring index.

Different cards use different strategies for this. You also don’t necessarily need to use all of the members of the header. For example, while you could use both the source and destination ports and IP addresses for a TCP connection, if the card ignored the ports, the right thing would still happen. Traffic wouldn’t end up out of order; however, it might not be spread quite as evenly amongst the rings.

This technology is often referred to as receive side scaling (RSS). On [SmartOS](https://smartos.org/) and other [illumos](http://illumos.org/) derived systems, RSS is enabled automatically if the device and its driver support it.

## Problem: Density, Density, Density

As Steve Ballmer famously once said, “Developers, Developers, Developers…”. For many companies today, it isn’t developers that is the problem, but density. The density and utilization of machines is one of the most important factors for their efficiency and their capex and opex budgets. The first major entry into enterprise for tackling this density was VMware and the Hardware Virtual Machine. However, operating system virtualization had also kicked off. For more on the history, listen to [Bryan Cantrill](http://dtrace.org/blogs/bmc)‘s [talk at Papers we Love](http://paperswelove.org/2016/video/bryan-cantrill-jails-and-solaris-zones/).

Just like airlines don’t want to fly with empty seats on planes, when companies are buying servers, they want to make sure that they are fully utilized. Due to the increasing size of machines, that means that the number of things running on it has to increase. With rare exception, gone are the days of the single app on a server. This means that the number of IP addresses and MAC addresses per server has jumped dramatically. We cannot just load up a box with NICs and assign each physical device to a guest. That’s a lot of cards, ports, cables, and top of rack switches.

However, increasing density isn’t necessarily free. We now have new problems that come up as a result and new scalability problems.

### A Brief Aside: The Virtual NIC

Once you start driving up the density and tenancy of a single machine, then you immediately have the question of how do you represent these different devices. Each of these instances, whether they be hardware virtualized or a container, has their own networking information. They not only have their own IP address, but they also have their own unique MAC address and different properties on them. So how do you represent these along with the physical devices?

Enter the Virtual NIC or VNIC. Each VNIC has its own MAC address and its own link properties. For example, they have their own [MTU](https://en.wikipedia.org/wiki/Maximum_transmission_unit) and may have a [VLAN tag](https://en.wikipedia.org/wiki/IEEE_802.1Q). Each VNIC is created over a physical device and can be given to any kind of instance. This allows a physical device to be shared between multiple different instances, all of which may not trust one another. VNICs allow the administrator or operator to describe logically what they want to exist, without worrying about the mess of the many different possible abstractions that have since shown up in the form of bridges, virtual switches, etc. In addition, VNICs have capabilities that allow us to prevent MAC address spoofing, IP address spoofing, DHCP spoofing, and more, making it very nice for a multi-tenant environment where you don’t trust your neighbor, especially when your neighbor sits next to you.

### Always Promiscuous?

We earlier talked about how devices don’t receive every frame and instead have filters. On the flip side, as density demands increased, so does the number of MAC addresses that exist on one server. When we talked about the simple NIC, it only had one MAC address filter. If the OS wanted to receive traffic for more than one MAC address, it would need to put the device into promiscuous mode.

However, here’s another way that devices have changed substantially. Rather than just having a single MAC address filter, they have added several. If you consider SmartOS and illumos, every time a virtual NIC is created, it tries to use an available MAC address filter. The number of filters present determines how many devices we can support before we have to put a NIC in promiscuous mode. On some devices there can be hundreds of these filters. Some of which also take into account the VLAN tag as well.

### The Classification Challenge

So, we’ve managed to improve things a bit here. We’ve got a couple hundred devices here and we’ve been able to program those devices into our MAC address filters. Our devices are employing RSS so we’re able to better spread the load across every device; however, we still have a problem: when a packet comes in we need to now figure out what virtual or physical device it corresponds to so we deliver it to the right instance.

By pushing all of this density into a single server, that server needs its own logical, virtual switches. At a high level, implementing this is straightforward, we simply need to look at the destination MAC address, find the corresponding device, and then deliver the packet in the context of that device.

NIC manufacturers paid attention to this need and the fact that operating systems were spending a lot of time dealing with this and so they came up with some additional features to help. We’ve already mentioned how devices can support RSS and how they can have MAC address filters. So, what happens if we combine the two ideas: given a piece of hardware multiple rings, each of which can be programmed with its own MAC address filter.

In this world, we assemble rings into what we call a group. A group consists of one or more MAC address filters and one or more rings. Consider the case where each group has one ring and one filter. So each VNIC in the system will be assigned to a group while they’re available. If a group can be assigned then all the traffic coming to the corresponding ring is guaranteed to be for that device. If we know that, then we can bypass any and all classification in software. When we process the ring, we know exactly what device in the OS this corresponds to and we can skip the classification step entirely.

We mentioned that some devices can assign multiple rings to a given group. If multiple rings are assigned to a group, then the NIC will perform RSS across that set of rings. That means that after the traffic gets filtered and assigned to a group, we then hash the incoming packets and assign it to one of those rings.

Now, you might be asking what about the case where we run out of groups? Well, at that point we try and leverage the fact that some groups can support multiple MAC addresses. This default group will be programmed with all the remaining MAC addresses. If those are exceeded, then we can put that default group and only that group into promiscuous mode.

What we’ve done now is taken the basic building blocks of both rings and MAC address filters and combined them in a way to tackle the density problem. This lets a single network card scale up much more dramatically.

## Problem: CPUs are too ‘slow’

One of the other prime features of modern NICs is what various NIC manufacturers call hardware offloads. TCP, UDP, and other networking protocols often have checksums that need to be calculated. The reason this came up is that many CPUs were considered too slow. What this really means is that there was a bit more latency and CPU time spent processing these checksums and verifying them. NIC vendors decided to add additional logic to their silicon (or firmware) to calculate those checksums themselves.

The general idea is that if when the OS needs to transmit the packet, it can ask the NIC to fill in the checksums and when it is receiving a packet, it can ask the NIC to verify the checksum. If the NIC verifies the checksum, then often times the OS will trust the NIC and then not verify it itself.

In general, these offload technologies are fairly common and generally enabled by default. They do add a nice little advantage; however, it often comes at the cost of debugability and may leave you at the mercy of hardware and firmware bugs in the devices. Historically, some devices have had bugs in this logic or had various edge conditions that will lead them to corrupt the data or incorrectly calculate the checksum. Debugging these kinds of problems is often very difficult, because everything that the OS generates itself looks fine.

There are other offload technologies that have also been introduced such as [TCP Segmentation Offload](https://en.wikipedia.org/wiki/Large_segment_offload) which seek to offload parts of the TCP stack processing to networking cards. Whenever looking at or considering hardware offloads, it’s always important to understand the trade offs in performance, debugability, and value. Not every hardware offload is worth it. Always measure.

## Problem: The Interrupts are Coming in too Hot

Let’s set the stage. Originally devices could handle 10 Mbits of traffic in a single second. If you assume a default MTU of 1500 bytes and that every packet was that size (depending on your workload, this can easily be a bad assumption), then that means that a device could in theory receive about 833 packets in a given second (10 Mbit/s * 1,000,000 bits/Mbit / 8 bits/byte / 1500 bytes/packet). When you start accounting for the Ethernet header and the VLAN tag, this number falls a bit more.

So if we had 833 packets per second and then we assume that each interrupt only has a single packet (the worst case), we have 833 interrupts per second and we have over 1 ms to process each packet. Okay, that’s easy.

Now, we’re not using 10 Mbit/s devices, we’re often using 10 Gbit/s devices. That means we have 1000x more packets per second! So that means that if we have a single packet in every interrupt that’s 833,000 interrupts per second. All of a sudden the time we have to just do all of the accounting for the interrupt becomes ridiculous and starts to add a lot of overhead.

### Solution One: Do Less Work

The first way that this is often handled is to simply do less. Modern devices have interrupt throttling. This allows device drivers to limit the number of interrupts that occur per second per ring. The rates and schemes are different on every device, but a simple way to think about it is if you set an upper bound on interrupts per second, then the device will enforce a minimum amount of time between interrupts. Say you wanted to allow 8000 interrupts per second, then this would mean that an interrupt could fire at most every 125 microseconds.

When an interrupt comes in, the operating system can process more than one packet per interrupt. If there are several packets available in the ring, then there’s nothing that stops the system from processing multiple in a single interrupt and in fact, if you want to perform well at higher speeds, you need to.

While most operating systems enable this by default, there is a trade off. You can imagine that there is a small latency hit. For most uses this isn’t very important; however, if you’re in the [finance world where every microsecond counts](http://queue.acm.org/detail.cfm?id=2536492), then you may not care about the CPU cost and would rather avoid the latency. At the end of the day though, most workloads will end up benefiting from using interrupt throttling, especially as it can be used to help reduce the CPU load.

### Solution Two: Turn Off Interrupts

Here we’re going to go and do something entirely different. We’re going to stop bothering with interrupts. Modern [PCI Express](https://en.wikipedia.org/wiki/PCI_Express) devices all support multiple interrupts. We’ve talked about how there are multiple rings, well each ring has its own interrupt identified by a vector. These interrupts are called [MSI-X](https://en.wikipedia.org/wiki/Message_Signaled_Interrupts). These devices allow you to mask the interrupt and turn it off on a per-ring basis.

Regardless of whether interrupts are turned on or off on a given ring, packets will still accumulate in the ring. This means that if the operating system were to look at the ring, it could see that there were packets available to be processed. If the OS marks the received entries processed, then the hardware will continue delivering packets into the ring. When the OS decides to turn off interrupts and process the ring with a dedicated thread, we call this **polling**.

Polling works in conjunction with dedicated rings being used for classification. In essence, when the OS notices that there’s a large number of packets coming in on the ring, it will then transition the ring to **polling** mode, disabling interrupts. The dedicated thread will then continue to poll the ring for work, consuming up to a fixed number of packets in any one poll. After that, if there is still a large backlog, it will keep polling until a low watermark is reached, at which point it will disable polling and transition back to interrupt based processing.

Each time a poll occurs, the packets will be delivered in bulk to the networking stack. So if 50 packets came in in-between polls, then they would all be delivered at once.

As with everything we’ve seen, there is a trade off of sorts. When you’re in polling mode, there can be an additional latency hit to processing some of these packets; however, polling can make it much easier to saturate 10 Gbit/s and faster devices with very little CPU.

## Recapping

We started off with introducing the concept of rings in a networking card. Rings form the basic building block of the NIC and are the basis for a lot of the more interesting trends in hardware. From there, we talked about how you can use rings for RSS and when you combine rings with MAC address filters you can drive up tenancy with hardware classification, which helps enable polling, among other features.

One important thing here is that all of the features we’ve talked about are completely transparent to applications. All of the software built upon the BSD/POSIX sockets APIs, functions like [connect()](http://illumos.org/man/3socket/connect), [accept()](http://illumos.org/man/3socket/accept), [sendmsg()](http://illumos.org/man/3socket/sendmsg), and [recvmsg()](http://illumos.org/man/3socket/recvmsg), automatically get all of the benefits of these developments and enhancements in the operating system and hardware, without having to change a single thing in the application.

## Future Directions and More Reading

This is only the tip of the iceberg both in terms of detail and in terms of what we can do. For example, while we’ve primarily talked about the hardware and software classification of flows for the purpose of delivering them to the right device, there are other things we can do such as throttle bandwidth and more with tools like [flowadm(1M)](https://illumos.org/man/1m/flowadm).

At Joyent, we’re continuing to explore these areas and push ahead in new and interesting ways. For example, as cards have been adding more and more functionality to support things like [VXLAN](https://tools.ietf.org/html/rfc7348) and [Geneve](https://tools.ietf.org/html/draft-ietf-nvo3-geneve-02), we’re exploring how we perform hardware classification of those protocols, leverage newer checksum offloading for them, and coming up with novel and different ways to improve the performance. If any of the following sound interesting, make sure to reach out.

If you found this topic interesting, but find yourself looking for more, you may want to read some of the following:

- The illumos [MAC big theory statement](https://github.com/joyent/illumos-joyent/blob/master/usr/src/uts/common/io/mac/mac_sched.c#L29) which describes how all of this is actually implemented and fits together.
- A paper from Sun Microsystems at SIGCOMM that describe more about [classification](http://conferences.sigcomm.org/sigcomm/2009/workshops/wren/papers/p45.pdf).
- The data sheets for the recent [Intel 10/40 GbE chips](http://www.intel.com/content/www/us/en/embedded/products/networking/xl710-10-40-controller-datasheet.html)
- The building blocks of our [SDN/network virtualization](https://smartos.org/man/5/overlay) story and the [gory details](https://github.com/joyent/illumos-joyent/blob/master/usr/src/uts/common/io/overlay/overlay.c#L16-L792).

Posted on September 15, 2016 at 10:06 am by rm · [Permalink](http://dtrace.org/blogs/rm/2016/09/15/turtles-on-the-wire-understanding-how-the-os-uses-the-modern-nic/)

In: [Miscellaneous](http://dtrace.org/blogs/rm/category/miscellaneous/)

[« Previous post](http://dtrace.org/blogs/rm/2014/10/01/illumos-day-2014/)