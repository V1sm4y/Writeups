Alright, so here’s what I found first while poking around the machine.

First off, we hit it with the usual combo—Nmap and Gobuster. Just to open up the field and see what’s up.

### Step 0: Reconnaissance with Nmap

Here’s the command I ran:
```bash
sudo nmap -Pn -sS -sV -sC ctf.thm
```
Classic aggressive scan with service/version detection and default scripts. The scan took a while but here’s the output that stood out:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
```

HTTP and SSH—solid. Noted the Apache version and jotted it down for later in case there are known exploits.

Then I checked the scripts output for HTTP and saw something juicy:
```
| http-robots.txt: 1 disallowed entry 
|_/VlNCcElFSWdTQ0JKSUVZZ1dTQm5JR1VnYVNCQ0lGUWdTU0JFSUVrZ1p5QldJR2tnUWlCNklFa2dSaUJuSUdjZ1RTQjVJRUlnVHlCSklFY2dkeUJuSUZjZ1V5QkJJSG9nU1NCRklHOGdaeUJpSUVNZ1FpQnJJRWtnUlNCWklHY2dUeUJUSUVJZ2NDQkpJRVlnYXlCbklGY2dReUJDSUU4Z1NTQkhJSGNnUFElM0QlM0Q=
```
Yep. We’re gonna come back to that.

### Step 1: Gobuster Time
You already know we’re gonna comb that web server. So I let `gobuster` loose with the good ol' common.txt wordlist:
```bash
gobuster dir -u http://ctf.thm -w /usr/share/wordlists/dirb/common.txt
```
Here’s what popped up:
```
/button               (Status: 200)
/cat                  (Status: 200)
/index.php            (Status: 200)
/iphone               (Status: 200)
/login                (Status: 301) [--> http://ctf.thm/login/]
/robots.txt           (Status: 200)
/small                (Status: 200)
/static               (Status: 200)
```
We’re seeing a bunch of valid responses—`/button`, `/cat`, `/iphone`, `/small`, and of course `/robots.txt`.

So I hit up `http://10.10.194.27/robots.txt` just to check if the devs were hiding anything juicy. Classic move, right? And boom—there it was:

```
User-agent: * (I don't think this is entirely true, DesKel just wanna to play himself)
Disallow: /VlNCcElFSWdTQ0JKSUVZZ1dTQm5JR1VnYVNCQ0lGUWdTU0JFSUVrZ1p5QldJR2tnUWlCNklFa2dSaUJuSUdjZ1RTQjVJRUlnVHlCSklFY2dkeUJuSUZjZ1V5QkJJSG9nU1NCRklHOGdaeUJpSUVNZ1FpQnJJRWtnUlNCWklHY2dUeUJUSUVJZ2NDQkpJRVlnYXlCbklGY2dReUJDSUU4Z1NTQkhJSGNnUFElM0QlM0Q=
```

At the very bottom of the file, I noticed something else too—just a string of hex values:

```
45 61 73 74 65 72 20 31 3a 20 54 48 4d 7b 34 75 37 30 62 30 37 5f 72 30 6c 6c 5f 30 75 37 7d
```

So I was like, okay, let’s see what this is hiding.

### Step 2: Checked that Base64 string in the `Disallow:` field
At first glance, that looked like Base64, so I threw it into a decoder. It had **multiple layers of encoding**. Here’s the process I followed:

1. Decoded once (base64):
```
VSBpIEIgSCBJIEYgWSBnIGUgaSBCIFQgSSBEIEkgZyBWIGkgQiB6IEkgRiBnIGcgTSB5IEIgTyBJIEcgdyBnIFcgUyBBIHogSSBFIG8gZyBiIEMgQiBrIEkgRSBZIGcgTyBTIEIgcCBJIEYgayBnIFcgQyBCIE8gSSBHIHcgPQ==
```
2. Decoded again (base64):
```
U i B H I F Y g e i B T I D I g V i B z I F g g M y B O I G w g W S A z I E o g b C B k I E Y g O S B p I F k g W C B O I G w
```
3. Another round gave:
```
R G V z S 2 V s X 3 N l Y 3 J l d F 9 i Y X N l
```
4. Final base64 decode:
```
DesKel_secret_base
```

Looks like a hidden directory!

### Step 3: Navigated to `/DesKel_secret_base`
Went to the directory in the browser and saw this HTML page with a photo and a cheeky message:
```html
<html>
	<head>
		<title> A slow clap for you</title>
		<h1 style="text-align:center;">A slow clap for you</h1>
	</head>
	
	<body>
	<p style="text-align:center;"><img src="kim.png"/></p>
	<p style="text-align:center;">Not bad, not bad.... papa give you a clap</p>
	<p style="text-align:center;color:white;">Easter 2: THM{f4ll3n_b453}</p>
	</body>
</html>
```
The flag was hidden in the white-colored text. A little sneaky but nothing we couldn’t handle.

### Step 4: Exploring `/login`
Then I checked out the `/login` page. It was a very basic form with just username and password fields. But here’s the kicker—the source code had a hidden message:

```html
<head>
	<meta content="text/html;charset=utf-8" http-equiv="Content-Type">
	<meta content="utf-8" http-equiv="encoding">
	<p hidden>Seriously! You think the php script inside the source code? Pfff.. take this easter 3: THM{y0u_c4n'7_533_m3}</p> 
	<title>Can you find the egg?</title>
	<h1>Just an ordinary login form!</h1>
</head>

<body>
	<p>You don't need to register yourself</p><br><br>
	<form method='POST'>
		Username:<br>
		<input type="text" name="username" required>
		<br><br>
		Password:<br>
		<input type="text" name="password" required>
		<br><br>
		<button name="submit" value="submit">Login</button>
	</form>
</body>
```

Boom! Another Easter egg. Hidden in a `<p hidden>` tag. Love to see it.

### Step 5: Revisiting the Homepage (Troll City)

So I circled back to the homepage—and oh boy, DesKel’s got jokes. Right off the bat, I saw:

```html
<h3>I want to play a game</h3>
<img src="saw.gif"/>
<a href="/game1"><h2>GAME 1</h2></a>
<a href="/game2"><h2>GAME 2</h2></a>
```

Two clickable game links. A classic SAW reference? This is getting spicy.

And further down, a rainbow cat apocalypse:

```html
<h3>EXPLODING RAINBOW.......</h3>
<img src="cat.gif"/><img src="cat.gif"/><img src="cat.gif"/><img src="cat.gif"/><img src="cat.gif"/>
<!--! Easter 17-->
<button onclick="nyan()">Mulfunction button</button><br>
<p id="nyan"></p>
```

That "Mulfunction" button might be linked to some JS mischief (maybe an inline script or obfuscated logic). I haven't clicked yet but I'm noting this down for deeper inspection.

There’s also a comment with `<!--! Easter 17-->`, which means another flag might be nearby, possibly hidden behind the button logic.

Also found something labeled **Easter 14** with a **ridiculously long hash**, but I decided to skip documenting that one for now—it’s just noise until I see context or can crack it later.

### ✅ Flags:
- Easter 1: `THM{4u70b07_r0ll_0u7}`
- Easter 2: `THM{f4ll3n_b453}`
- Easter 3: `THM{y0u_c4n'7_533_m3}`

Three eggs cracked. Let’s keep the hunt going 🥚🔍
