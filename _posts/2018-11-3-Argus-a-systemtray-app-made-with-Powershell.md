---
Layout: post
title: Argus, my loyal dog!
Author_profile: true
published: true
date: '2018-11-4'
---
Hi! This blog-post is about one of the projects i'm most proud of! The development of "Argus" started around March of 2018. This project grew while i was working on it. I have tried to structure this blog-post as best as i could but in truth most ideas and discoveries were made during the development. This post is less technical and more an in depth overview of what the application actually is. Here goes ...

## The birth of the idea
One part of my job is being part of the help desk. When called by a customer or colleague, i usually ask for their computer-name. Most users tend to not know the name by heart (*com'on guys!)* or have a hard time figuring out where to find it. With the deployment of Windows 10 this became even more prevalent.

So one day i was reading r/Powershell over at Reddit and i found  [this](https://www.reddit.com/r/PowerShell/comments/8ovw38/_/e06kepd/) post.

A user named *Lee_Dailey* linked to a script over at the Microsoft's script gallery. I took a look and the screenshot kinda gave me the idea to build this for our end-users. 

![enter image description here](https://i1.gallery.technet.s-msft.com/windows-powershell-system-792a1db9/image/file/181340/1/untitled.png)

This would make things SO much easier to explain to them. 

## Oh oh, XAML (WPF)
When opening the script, i found out that the GUI was in XAML-format. So it used the Windows Presentation Framework or WPF in short. When i first started with Powerhell GUI's i had come a cross WPF but I had never used this before and had always skipped past it because i always use PS Studio.

So after doing my research i concluded that the *easiest way* to edit this was to install Visual Studio Community edition, a free version of Visual Studio.

Some parts were a true struggle like creating a button with a drop-down menu or creating a hoover effect on a button. The amount of code it generated for that was, in my opinion, insane. Anyway,  I'm glad that i stumbled into XAML, i'm no expert by any means but at least now i have **some experience** with it.

So after **a lot of Googling** and tinkering i had my basic design finished and i decided on what content i needed in the application. The result looked like this :

![Design]({{site.baseurl}}/assets/images/argus/design.png)  

## XAML in Powershell Studio ?
So the next big question was **" How the hell am i going to get this into an executable? "** This would mean that i would combine Winforms and WPF.

Since the example script was ready to go, i figured i would try to get that into an EXE first. So i opened up Powershell Studio 2018 and after some configuration and trial & error, i got a working sytemtray app, designed with XAML and running as a compiled EXE.

This lead me to believe that my project was a doable one. 

## One EXE - no installation needed
The application is one EXE and that's it. When the application starts, it **creates** the DLL  for MahApps (see below) and all its images (banner, icons, images for toast notifications)  in a temp-location. This temp-location is under the APPDATA of the user who starts the application. So you have no issues with *user-permissions*.

```powershell
$env:temp
```
I'm creating the DLL and the images thanks to the following functions:

```powershell
[System.Convert]::ToBase64String
[System.Convert]::FromBase64String
```

This allows you to store the data\content from the DLL or  image into a variable (hash).
This looks something like :

```powershell
$MahAppsMetro = [System.Convert]::FromBase64String('TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQA ..... ')
```
You can then create the file with the following code :

```powershell
Set-Content -Path $env:temp\MahApps.Metro.dll -Value $MahAppsMetro -Encoding Byte
```
This would be like copy\pasting the DLL, only now i don't need to store the sourcefiles at a location that every user has access to.

# Expanding on the idea

I looked for additional features and this is what i ended up with:

- a link to the helpdesk-system to create a ticket\call
- a link to our service-monitor to check if there are issues with program\service X or Y
- the incorporation of my other tool, the [TCP\IP-printtool](https://cookiecrumbles.github.io/Deploying-TCPIP-printers-with-a-Powershell-GUI/)
- a way to manage your default printer
- a SOS-warning system

This is a lot to fit into a small GUI like Argus. This is where **MahApps** comes in.

### MahApps
During my research i had come a cross [MahApps](https://mahapps.com/). This is a UI toolkit for WPF. It gives your application the Metro-look and feel. Something we are accustomed to with Windows 10. One of the features that appealed to me the most was the "fly-in" option. This would give me the option of a "hidden" menu and would keep my design nice and simple.

I'm not gonna lie, this was something i spend **a lot of time on** figuring it out. Step-by-step things became clear and some parts started to work. 

Now that i'm writing this article and i'm reviewing the XAML\MAhapps  part, it's already a bit of a blur on how i got that part done. I should re-visit it in the coming months.

Anyway, with the Helpdesk-button on the top and the wheel-cog on the buttom left, i had most of my features implemented. The gray wheel cog will open a fly-in menu that has the option to install a printer. This would open the [TCP\IP-printtool](https://cookiecrumbles.github.io/Deploying-TCPIP-printers-with-a-Powershell-GUI/) i wrote about.

The green cog will just start the command:
> control printers

This way users can quickly change their default printer. 

So this is what Argus looks like as a finished product, with and without the fly-in menu.

![Design]({{site.baseurl}}/assets/images/argus/gui.png)  


## SOS-warning system

### Background story
Some years ago we were unfortunate enough to have both the exchange-server as the telecommunication-server down at the same time. The problem here, aside the obvious, was that we couldn't reach and inform our users about the issues. 

Afterwards a colleague asks if we could figure out some sort of SOS\Emergency system if this were ever to happen again.

One request of the end-user was that the message or notification would not be too intrusive. This brought me back to the toast-notifications that would just pop-up for a few seconds to then disappear. 

In Active Directory, we have computer groups per department. So it wouldn't be too hard to figure out what computers we'd like to reach. As long as we had network, it is doable.

### Argus was born
#### Part I - The Client

I incorporated a function within the system-tray application and named it Argus. This would become the name of the whole application.
Argus looks for a JSON-file on it's own hard-drive every 60 seconds. The location of this JSON-file is one where a default user has all permissions. This is only half of the story tho ... .

#### Part II -  The monitor\feeder
On our department we created a website that keeps track of tickets, the service-monitor, server-cooling, ... . We display all this information on a TV. Picture it as a slide-show where this information 'rolls over the screen'. This is all driven by a laptop that acts like a "TV-server".
On this laptop i made a **second tool** that servers as the feeder for the "Argus SOS"-notification.

It **monitors and scrapes our service monitor every 60 seconds** and stores its content as an MD5-hash. The scraping is done with XPATH in combination with the [HtmlAgilityPack](https://html-agility-pack.net/).   
I compare the new hash with the previous result. If the hashes don't match, a new message is send.
The feeder will query AD for all existing computers in our OU. It will then quickly ping all these computers and filter out the online ones.

Once it has a list of online computers, it will create a JSON-file on their C-drive. It is this location that the Argus-function monitors. If it finds a JSON-file, it will grab the content and will display it in the form of a toast-notification as shown here:

![Design]({{site.baseurl}}/assets/images/argus/toasts.png)  

There are 4 types of messages (each with their own color) that can exist on the servicemonitor.

- Information (info)
- Outage (storingen)
- Unavailable (onbeschikbaar)
- Available (beschikbaar)

#### Requirements and benefits

One of the **requirements** is that the "feeder\monitor-application" has suffiecient rights both on AD and on the C$-shares of the clients. 

I could have gone a different route and let Argus scrape the servicemonitor on each client. But that would have caused a lot of networktraffic and server-requests. I disliked that idea and i figured that having one monitor-system that then copies the message to the clients a much more elegant way of going about it.

## Closing words

This project litteraly came to life by reading a Reddit post. It has shown me some new techniques like Visual Studio, WPF, XAML and  i even briefly touched on Runspaces. I came a cross many different problems but i'm proud of the end-result.

Since there is no installation proces and the EXE contains everything that is needed to run the application, it should be around for quite some time.

Good boy Argus !