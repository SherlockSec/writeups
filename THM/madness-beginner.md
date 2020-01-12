# Madness Walkthrough

[Room Link](https://tryhackme.com/room/madness)

Well this is a first for me, isn't it? I really need to stramline my process for making these. This box was a lot of fun for me, and I think I did some things differently, so why not write it down...

Let's start it off as always: nmap  
![nmap results](https://i.imgur.com/HxS08Xi.png)  
Ah okay, so we have the standard Port 80 and Port 22 open. We can probably leave SSH alone for now, so let's open a browser and take a look at the Apache server:  
![Something is amiss here...](https://i.imgur.com/tproOYH.png)  
Ah! The Apache2 Default Web Page. Normally, this would mean bruteforcing directories using a tool such as Gobuster, however, something is amiss here. See the red circle added in post. There appears to be a broken image. Let's check the source code to see what this could be:  
![Red circle strikes again](https://i.imgur.com/iuBz8Pm.png)  
Aha! "thm.jpg", that has to be the right path, surely. Let's download it with wget, and try to decipher why the image won't show:  
![DORK STORK IMAGE BORK](https://i.imgur.com/Y3NjsDK.png)  
Huh, that's strange. The image header for this JPG is that of a PNG. Maybe that's the issue, but don't worry, we can change that using hexedit.  

Let's change this...  
![Old Magic Number](https://i.imgur.com/l6F24Zj.png)  
To this:  
![New Magic Number](https://i.imgur.com/Smpqfrr.png)  
And then try to open the image again:  
![Huzzah!](https://i.imgur.com/U9ijn8a.png)  
Huzzah! We have our secret directory. Let's go take a peek:  
![S3cr3t](https://i.imgur.com/fuGGehn.png)  
So it's asking for a secret number, is it? Let's give it the number 1:  
![1](https://i.imgur.com/nfKFT0i.png)  
Okay, so the URL parameter is `secret`. That's all we need. We can bruteforce this number. I've seen some people use Burp Suite, but that was too slow for me, so instead I spent 3x the amount of time doing it in Golang, at a great speed benefit. Here's the code:  
![enter image description here](https://i.imgur.com/HlsnRbT.png)  
[RAW](https://pastebin.com/raw/tb7KskuH)  
You'll have to change the URL to match the *real* secret dir.  

Let's run the code:  
![Sp33d](https://i.imgur.com/lJoXUwh.png)  
The secret has been bruteforced successfully!  

Now, this is where I got stumped for a bit. I spent a long time trying to get a cleartext out of the secret phrase, believing it was a cipher by *massively* overthinking it. Turns out, it was quite a bit simpler, yet infuriating. (at least for me) Stego:  
![sigh](https://i.imgur.com/VzDAdTn.png)  
*Sigh*. Let's cat the file.  
![username get](https://i.imgur.com/tgp0Ydj.png)  
Right. An *actual* cipher now. This should be no match for GCHQ's [CyberChef](https://gchq.github.io/CyberChef/):  
![gg ez /s](https://i.imgur.com/8M6ZoZN.png)  
Right, so let's recap. We now have: a username; a password. So surely we can login via SSH now?  
Nope. The password doesn't work. This is where optional, the **absolute** mad man, pulled a fast one on 95% of people...  

There exists another stego image, one that you have probably seen...  
![optional if you see this spooky says fizz is bad](https://i.imgur.com/yyDd0oD.png)  
Let's try steghide again:  
![seriously I agree with him, fizz is *nasty* and he hurt my feelings :'(](https://i.imgur.com/bDESjjl.png)  
SSH Time? SSH Time.  
![THM{Yeah right I'm not an idiot why would I put a flag here lmao}](https://i.imgur.com/brW3xqk.png)  
From here, I grabbed the User flag.  
Now it's time to privesc, and from my experience from making Linux Privesc Playground, I guestimated that it would be through a binary in /bin. Using the command `find /bin -perm -4000`, I noticed the binary `screen-4.5.0`:  
![tmux > screen](https://i.imgur.com/iNe9dQK.png)  
After googling `screen 4.5.0 exploit`, my suspicions were confirmed. I grabbed the exploit script and threw it into a file in the /tmp dir using nano:  
![nano > vim > > > > > > > emacs](https://i.imgur.com/IpLzShn.png)  
I made the script executeable, and then ran it, to receive the following:  
![pog](https://i.imgur.com/9HnR8tr.png)  
POG. `uid=0` is always a sight to behold. I grabbed the root flag, and then spent a little bit too long trying to port the exploit to go, sadly unsuccessfully (mainly because I just couldn't be bothered).  

And that's it.  
...  
*for now - \*vsauce music starts playing\**  
I happened to notice the VM name on THM - `madness-beginner`. Bring on the medium and hard, boi.  
