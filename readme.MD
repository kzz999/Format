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

{
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
}
