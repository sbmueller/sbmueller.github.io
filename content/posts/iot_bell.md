---
title: "A Digital Doorbell in 10 Minutes"
date: 2019-06-15T21:20:28+02:00
tags: ["hacking"]
draft: false
---

Today I discovered that my doorbell is broken. Since I'm awaiting some deliveries,
I figured I need a hotfix until my landlord repairs the analog doorbell. It took me
only 10 minutes to implement a digital solution.

## Concept

I immediately thought of [IFTTT](https://ifttt.com/), an online service that
acts as hub between several IoT devices or internet services. The scheme is
always the same:

> If **this** happens, then do **that**.

For **this** I thought of a QR code that someone in front of my door could
scan, which directs her to a website. As soon as that website is accessed, the
trigger is fired.

The **that** would just be a simple notification on my phone, displayed by the
IFTTT app.

## Implementation

I went to set up a new applet in my IFTTT account. The suitable trigger for my
purpose is called "Webhook". It can easily be enabled and in the trigger's
settings page, I can see my custom URL, which looks something like
`https://maker.ifttt.com/use/{key}`. When accessing that page,
the instructions are very clear:

> To trigger an event, make a POST or GET web request to
`https://maker.ifttt.com/trigger/{event}/with/key/{key}`

where `{event}` is a custom key to identify the event. I chose `ring_bell`.

With that set up, I only needed to create a QR code, that directs to that
website. There are enough options do to this, but I used [QR Code Generator](https://www.qrcode-generator.de/).
When setting up a free account there, I even get the possibility to add a text
field to the code. I chose to use my name, obviously.

For the action, I chose the notification service which just sends a push
notification to an IFTTT Android or iOS app. I set it up to notify me with the
text "Someone just rang the doorbell".

Now all I had to do was print out the code and glue it onto the "out of order"
sign someone has put up to cover the broken doorbells. Here is the notification
I get (in German) with about 3 seconds delay after someone accesses the
website:

![Notification](/images/ifttt_notification.png)
