# tryhackme-U.A.-HighSchool
![intro](https://github.com/user-attachments/assets/a10ad4ac-e9d8-4897-ab87-126f9621d73d)

This is a walkthrough for the tryhackme CTF U.A. High School. I will not provide any flags or passwords as this is intended to be used as a guide. 

## Scanning/Reconnaissance

First off, let's store the target IP as a variable for easy access.

Command: export ip=xx.xx.xx.xx

Next, let's run an nmap scan on the target IP:
```bash
nmap -sV -sC -A -v $ip -oN
```

Command break down:

-sV

Service Version Detection: This option enables version detection, which attempts to determine the version of the software running on open ports. For example, it might identify an HTTP server as Apache with a specific version.
-sC

Default Scripts: This option runs a collection of default NSE (Nmap Scripting Engine) scripts that are commonly useful. These scripts perform various functions like checking for vulnerabilities, gathering additional information, and identifying open services. They’re a good starting point for gathering basic information about a host.
-A

Aggressive Scan: This option enables several scans at once. It combines OS detection (-O), version detection (-sV), script scanning (-sC), and traceroute (--traceroute). It’s useful for a comprehensive scan but can be intrusive and time-consuming.
-v

Verbose Mode: Enables verbose output, which provides more detailed information about the scan’s progress and results.
$ip

Target IP: This is a placeholder for the target IP address you want to scan. In practice, replace $ip with the actual IP of the machine you are targeting.
-oN

Output in Normal Format: This option saves the scan results in a plain text file format. After -oN, specify a filename where you want to store the output.

The scan reveals many open ports. Let's check out the webserver on tcp/80:

![website80](https://github.com/user-attachments/assets/37f54b95-96ad-4772-aa43-37831f79491c)

There doesn't seem to be much here. The gobuster scan reveals somethings of interest:

![assests](https://github.com/user-attachments/assets/ffff1c62-a5c8-43f3-8e89-0054c4f9cc46)

There doesn't seem to be content in the assets dir, but let's run another gobuster scan on this dir:

![buster2](https://github.com/user-attachments/assets/af5bc092-39a7-482e-97b3-831510da8fff)

The images dir is forbidden, but the index.php dir just shows a blank page. I tested the url for php Local File Inclusion (LFI) with the ls cmd and it seemed to return the result base 64 encoded:

![ls](https://github.com/user-attachments/assets/ab93eefa-1ba7-4ba8-9545-b22e60cade82)

If we unencode the result it shows a directory listing:

![base64_ls](https://github.com/user-attachments/assets/0aca6969-1403-4c50-8c2f-071492f4a6ac)

Next, I tried using LFI with "index.php?cmd=cat /etc/passwd", and repeated the previous process of decoding the output:

/etc/passwd reveals a user deku:

![deku](https://github.com/user-attachments/assets/805e16cb-0308-45d5-b6cd-edcc6dd1b065)

I searched around file system looking for credentials or an ssh key, but no luck. 
I then decided to step this up a notch and try to execute a revshell to get Remote Code Execution(RCE). 
I use a php exec shell, and url encode it:

Then paste it in after "index.php?cmd=" and set up the netcat listener on your choosen port.
I got the revshell as wwww-data and upgraded it:

![shell](https://github.com/user-attachments/assets/cbf9f742-e95a-4fab-bf0e-0430c6cb6291)

One of the first things I noticed when exploring the file system, is the images directory has content that we can now access. I ran the file cmd on both images and the oneforall.jpg contains data. I'll use a python3 websever to transfer the file to my attack box:

![oneforall jpg](https://github.com/user-attachments/assets/f2096009-4378-4181-90ec-0acf0cab0288)

![wget](https://github.com/user-attachments/assets/7fb0c16c-5478-441f-8f14-3673772a680a)

When trying to open the oneforall.jpg we get this error message:

![open_oneforall](https://github.com/user-attachments/assets/f5a1ef06-505b-448b-a118-ba621ce5b7d7)

This led me to believe that the file is corrupt and to use hexeditor to check the magic numbers of the file.







