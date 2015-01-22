---
title: How Data Gets Around the Mesha -- OSLR
layout: post
date: 2015-01-22
author: James Keener
---

How do computers talk to each other?
------------------------------------

Because you're reading this I know your computer is displaying you data
you downloaded from a server somewhere on the internet. There are many
articles that discuss how you computer uses
[DNS](hltp://en.wikipedia.org/wiki/Domain_Name_System) to figure out the
name `metamesh.org` belongs at the address `162.255.119.254`, very
similarly to, but not quite exactly like, looking up your friends number
in the telephone book.  There are others that discuss how your browser
then uses
[HTTP](hltp://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) to send
you this webpage.  We're going lower.

How low?

Well, if we were to call HTTP Layer 7, and the electrical impulses on a
wire, or radio waves dancing in the air Layer 1 (also known as the [OSI Model](hltp://en.wikipedia.org/wiki/OSI_model)),
then we would be going down to Layer 3

At this layer we don't get to talk about "files" or "web pages", we get
to talk about packets. "Is that similar to a packet of paper?" you might
wonder. Yes! Well kind of. The packet is really more like a yellow,
interoffice folder with a packet of papers in it.  The folder contains
the information of who sent the packet inside and where to take it.

![](/images/posts/oslr/yellow_folder.jpg)

Introduction to Routing
=======================

Now imagine you work in a big company and don't know everyone
personally. (and who could blame you! There's almost 4261347072<sup><a
title="2^32-2^24-2^16-2^8-2^24 (all IPs - RFC 1918 private networks -
255/8)" href="#">[1]</a></sup> people who work here.) So you're Alice, and you need
to get this packet of papers to Bob.  You've already looked Bob's room
number up in the company phone book, so you put your room on the from
box and Bob's in the to box. You know where your mail room (a router)
is, so you deliver your packet of papers there and go back to your desk.

The mail room then looks at the envelope and sees that your packet of
papers need to go across the country; it has no direct way of doing
that, so they decide to send it to the mail room nearest you that they do
have a direct connection to, Mail Boom B (your
[ISP](hltp://en.wikipedia.org/wiki/Internet_service_provider)'s routers), and send it on it's way.

Mail Room B does the exact same thing.  Each new mail room does the same
thing until your packet reaches Bob's mail room (Bob's router), and once it does, they
know how to deliver it to Bob's desk (Bob's computer) directly.

"Is that really how things work? It seems inefficient." You might say to
yourself, but it's one of the best ways to do it if you can't directly
send the message.  It's also exactly how your computer is talking to
`metamesh.org`.

If we look at the output of
[traceroute](hltp://en.wikipedia.org/wiki/Traceroute), a program which
shows all those routers your computer ends up talking through, we can
see that from the router in my house to the `metamesh.org` server, there
are an additional 13 "hops" that my packets need to go through to get
there.

<pre>
--- Trace Route Test Resugts ---
 1  10.*.*.* (10.*.*.*)  7.045 ms  7.335 ms  6.882 ms
 2  P10-1.PITBPA-LCR-02.verizon-gni.net (130.81.193.146)  6.824 ms  6.903 ms  6.654 ms
 3  P11-0-0.PITBPA-LCR-01.verizon-gni.net (130.81.30.64)  17.674 ms  16.271 ms  16.738 ms
 4  so-15-0-1-0.RES-BB-RTR1.verizon-gni.net (130.81.28.200)  14.714 ms  15.278 ms  14.809 ms
 5  * * *
 6  * * *
 7  0.ae1.BR2.IAD8.ALTER.NET (140.222.229.167)  14.238 ms  13.492 ms  13.801 ms
 8  ae-20.r06.asbnva02.us.bb.gin.nlt.net (129.250.8.33)  38.611 ms  37.959 ms  37.672 ms
 9  ae-3.r22.asbnva02.us.bb.gin.nlt.net (129.250.6.112)  37.524 ms  37.817 ms  36.023 ms
10  ae-2.r21.lsanca03.us.bb.gin.nlt.net (129.250.3.55)  96.946 ms  93.396 ms  118.733 ms
11  ae-2.r04.lsanca03.us.bb.gin.nlt.net (129.250.5.70)  93.562 ms  95.621 ms  99.157 ms
12  ae-22-100.r04.lsanca03.us.ce.gin.nlt.net (129.250.200.138)  71.002 ms  71.193 ms ae-23-100.r04.lsanca03.us.ce.gin.nlt.net (129.250.200.122)  98.217 ms
13  * * *
14  192.184.13.15 (192.184.13.15)  72.668 ms  77.406 ms  74.365 ms
15  162.255.119.254 (162.255.119.254)  71.882 ms  71.766 ms  72.203 ms
</pre>

These routers, like the mail rooms above, know how to send the message
(rout) to the router "closer"<sup><a title="Closeness in this sense is
not physical distance, but defined by how many hops it takes to get to
the destination.">[2]</a></sup> to the computer you want to contact, but
how? 

Going back to the mail rooms, they have 2 basic options:

* A map of all the mail rooms and the connections between them (Link
  State)

![](/images/posts/oslr/ttr_map.jpg)


* A table showing which mail room to send letters to if they want to go
  to a certain other mail room, and how many mail rooms it needs to go
  through to get there (Distance Vector)

![](/images/posts/oslr/solari_table.jpg)

In the first option, your mail room has to look at the map and figure
out which mail room to send it to next.

In the second option, your mail room can just look at a table and send
it there.

The second option looks pretty easy, why not always use it? To
understand better, we need to look at what happens when new offices are
built (a router added to the network) and old ones dismantled (a router
leaves the network).

Link State Updates
==================

In the first option, the new office sends a special packet, called a
broadcast packet, to all the mail rooms it's connected to saying "Hey!
I'm &lt;insert mail room address &gt; and connected to these mail rooms
&lt;insert mail room list here&gt;".  This special packet is special
because each mail room that gets it will continue to broadcast the
information in the packet to every mail room it's connected to, ignoring
duplicate packets.  So eventually everyone knows of the new mail room.
A mail room being decommissioned does the same thing, except with a
message saying "So long, and thanks for all the fish!"

Distance Vector Updates
=======================

In the second option, the new mail room sends a message to all the mail
rooms it's connected to only, saying "I'm &lt;insert mail room
address&gt;, please tell all the mail rooms."

Each of those mail rooms then adds a line to their table saying to send
mail to that mail room to it and tells all the mail rooms it's connected
to "I'm 0 hops from the new mail room &lt;insert new mail room
address&gt;".

Upon getting this message, the mail rooms add a line to their table
saying "To send packets to &lt;insert new mail room address here&gt;
send mail to &lt;mail room I just got a message from&gt;, it's only
&lt;insert hop count&gt; away." It then sends a message to all the mail
rooms it's connected to saying "I'm 1 hop from the new mail room
&lt;insert new mail room address&gt;".

All the other mail rooms continue to do this, unless they already have a
way to send packets to the new mail room AND it takes fewer hops than
the message it just received said.

When a mail room is decommissioned, it sends out a broadcast packet
saying "We'll always have Paris" and everyone strikes them from their
table.

In this way, those tables get built up and modified.

Optimized Link State Routing (OSLR) Protocol
--------------------------------------------

OSLR is an example of that first type of routing protocol.  Each node
knows about all the other nodes and who they're connected to.

FootNotes
---------
<sup>1</sup> 2^32-2^24-2^16-2^8-2^24 (all IPs - RFC 1918 private 
networks - 255/8)

<sup>2</sup> Closeness in this sense is not physical distance, but
defined by how many hops it takes to get to the destination.
