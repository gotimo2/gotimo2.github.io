---
layout: post
title: EC2 minecraft server
date: 2023-01-30 12:02 +0100
tags: AWS programming
categories: programming
---

## Running a minecraft server on an amazon EC2 instance

> This is an old project; Currently this server doesn't run anymore.

Have you heard? I made a minecraft server. i've certainly heard. many times, in my DM's. Mostly whenever it goes down. To make this server i needed somewhere to host it, preferably without paying an arm and a leg. Now, with a little experience from the internship i was doing, i'd figured out that AWS is actually pretty good at servers, unlike my last server provider, where my server physically burnt down.  

For this story two AWS services are relevant: EC2 instances and lambda functions. I have a Lambda function set up so when the HTTP endpoint receives a request, it checks the payload for a password and starts or stops the server accordingly. The server is set up in such a way that whenever it starts up, it fires up a tmux window, and then start up our minecraft server startup script. Another script is run when the server receives a graceful shutdown command, that basically just sends a /stop to the minecraft server.

This piece of code is ran when you send a request to the endpoint:
![Some lambda code](/assets/img/Minecraft/lambda_code.png)

it's not incredibly well optimized or secure, but it does what it's supposed to, and stops people that aren't me from doing it. so why is this all needed? Money.

The type of EC2 instance I started out with for the server, t2.small, is cheap** and performant* at about 2c/hour and 2GB of ram with 1 solid CPU. And that is what you'd think, until you read the little asterisks on the T2 types' specification:

!["Burstable performance"](/assets/img/Minecraft/burstable_perf.png)

"Oh wow, it gets even more powerful! Nope, you can use about 30% of the CPU, and when it's below that it accumulates "credits". These "credits" get consumed when you use more than 30%, and when you run out of credits your CPU gets capped at 30%! And for the low low price of 5c per hours you can turn on "unlimited mode" that will still try to not use credits, and run horribly!

So, new approach: run a different server type. minecraft is a rather CPU-focused game for servers, so the c6 type for servers works decently well for this, as you've all mostly been feeling.

Downside: c6a is expensive as hell and costs 8c/hour.

It also does not help that this shit runs for entire days at a time on weekends. So, solution: keep the price in check by not running it all the time.
(while not running i still have to pay for the IP, because i'm holding onto a static IP that i'm not using, i don't have to pay for the IP while the server is on).
(in retrospect, using a [duckDNS](https://www.duckdns.org/) would've been a good solution to that)

So how do i let *other people* turn a server on and off without setting up an annoying amount of permissions and trying to familliarize them with AWS? 

Discord! I've got everyone i'd like to be able to turn on the server in a discord anyways. I can make a bot that turns it on and off, and run it from a raspberry pi. I could probably make it detect the amount of players on a server before turning it off, but that is more work than i'd like, so i'll just automatically turn it off after ~2 hours. They can always turn it on again.

So, that leaves us with something like this:

```python
@bot.event
async def on_message(message: discord.message):
    global is_running
    if message.author == bot.user: #bot does not reply to itself
        return
    if not message.guild: #if in DM's

        ids = open("users.txt", "r").read().splitlines() #read the user file

        print(f"[{message.created_at}] {message.author.name} : {message.content}") #print out whats happening   
        if str(message.author.id) in ids:
            if message.content == "stop": #if the word "stop" is sent, stop the server, if anything else start it.
                print("stopping server")
                await stop_server()
                await message.channel.send("Stopping server")
                is_running = False #set is_running to false
                return
            if is_running:
                await message.channel.send("The server is still on!")
                return
            await message.channel.send("server starting!")
            await start_server()
            is_running = True
            await asyncio.sleep(7200) #wait 2 hours, without blocking the main thread
            await stop_server() #(that means the bot can still tell you when the server is on)
            is_running = False
            
        else:
            await message.channel.send("You're not on the list!")
```

```python
async def start_server():
    
    response = requests.post(ENDPOINT, json={
        "action":"start",
        "secret":AWS_SECRET
    })

    print (response.request.body)

    print (response.request, response.headers, response.reason, response.json, response.content)
```

So, now that i've got that running on a raspberry pi at home, and the server sort of running in AWS, we've got something akin to this:

![Diagram](/assets/img/Minecraft/diagram.png)

Now that i've got everything taken care of, the server can just run whenever it is needed, and i don't need to worry about the bills for time people didn't spend using the server. 






