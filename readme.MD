![Format-1024x775](https://github.com/st1llz0r/Format/assets/142597099/41c58bf0-ac14-41de-9e74-8d425beeeefb)

Let's go

information collection;
  Scanning target host.

nmap -sS -sVC target ip;

![image](https://github.com/st1llz0r/Format/assets/142597099/6773bb04-d5b4-4b0d-bbb6-d7159a9cbafb)

  Navigate to target IP port 80;
  Nothing, so try and Spoof DNS;

 <b> echo '10.10.11.213 microblog.htb' >> /etc/hosts </b>;

 Trying again, we can discover http://app.microblog.htb, therefore spoof it. bind the domain to hosts file;

 Browser input > http://app.microblog.htb;

 Browse the web app and see what functionalities it has;

 Create an account and publish your own blog. Test out the Editing page and so on;

 On the editing page let's try and see how the web page reacts to basic XSS; Use the h1 option and input:

 <b> <script>alert(1)</script> </b>;

 Hmm, interesting result;

 Now let's try something crispier;

 Now let's change it to text and capture the traffic by using Burpy suite;

 Analyze the pack

  ![image](https://github.com/st1llz0r/Format/assets/142597099/52e58d19-1292-4a6a-a383-98a9ca924351)

  After analyzing let's start with basic stuff, basic LFI:

 ![image](https://github.com/st1llz0r/Format/assets/142597099/dfc3ff0a-2bac-4c0c-9bcb-cf08433e7a27)


n0w navigate to your browser and refresh:

 ![image](https://github.com/st1llz0r/Format/assets/142597099/2f0d0e67-ddc0-4af7-81e6-10b07a060be5)

Pew pew pew pew
We got a lead
redis and git and user
let's navigate to http://microblog.htb:3000/

Who would ve thought =D



![image](https://github.com/st1llz0r/Format/assets/142597099/e5e3b712-7d3c-45d5-8552-d3b320d04135)



![image](https://github.com/st1llz0r/Format/assets/142597099/12aee48a-ae78-492a-9a95-a12172e64969)


So, there is this Cooper guy (USER) who has created this sunny (blog domain);
Site is pretty basic as in infrastructure so the folders and all will apply for our account also;

/home/ibst1lllz0r/Downloads/index.php
