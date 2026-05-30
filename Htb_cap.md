# Writeup
So ofc we start off with an nmap scan. The one I like to use is :
```
nmap -sS -sV -sC <target_ip>
```
This does service scan , version scans and runs automated python scripts in that order
which gives you :

```
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    gunicorn
```
so three ports are open including a http server on port 80. Lets visit that.
Its a security dashboard with clickable links that lead to directories. Clicking on a "Security Snapshot" link we see:

```
target-url/data/5
```
Interesting , this is a classic setup for an IDOR (Insecure Direct Object Reference). Now as for what i did I manually brute forced the numbers , as i saw anything above 5 redirects to the homepage so it had to be between 0-5 (This is done by the authors to make it simple , if not Ffuf command to be used). and at **/0** we get some data . 

I decide to download the **0.pcap** file . You can use any reader but i use Wireshark. Now if youre aware of how a login request works , youll just be able to follow the sequence and reach there by yourself , if not if you look at Frame 40 , youll see the password. This happens due to the password not being encrypted .

```
ssh nathan@target_ip

```
Connect to SSH using the same password and it worked 
WE ARE IN!!!!

Now to see the files present in this directory 

```
ls
```
No **-la** command needed here
Now you have user.txt

```
cat user.txt
```
Use this command to see the flag
