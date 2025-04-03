---
title: "How I Reported a Vulnerability to My School's Management Service Provider"
date: 2022-06-25T15:34:30-04:00
categories:
  - blog
tags:
  - Brute-Force
  - Authentication-Response
  - Authentication-Bypass
  - Hacking
  - School-Management-System
---

> This blog demonstrates how a weak authentication response and a lack of complexity in passwords can result in **Authentication Bypass**.



### Disclaimer
_This blog is for educational purposes only, you should not test the security of devices that you do not own or do not have permission to test._

---

> #### Let the name of my School's Management Service Provider be XYZ.

---

# What is XYZ?
> XYZ is a School Management Service provider. It enables parents to perform certain tasks including paying fees, viewing their children's personal information, and viewing their kids' grades and attendance records, etc.


---

# Authentication Response
> This service is utilised by the school where I study. Well, there isn't really a flaw in the authentication, but the Authentication Response isn't up to par. If you enter an invalid username during account login, a message stating that the user does not exist is displayed. Here, the display message is considered to be a weak point which allows attcker to **Brute Force**. Another problem is that a username assigned to an account is quite obvious. It is assigned in a way which can be brute forced.

> What OWASP advices: [OWASP Cheatsheet](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Authentication_Cheat_Sheet.md#authentication-and-error-messages)

![Authentication Response](/assets/images/XYZ%20-%20Brute%20Force/XYZ-authentication.png)

---

# Guessing Username
> If you are a student and, for instance, your father's name is _John Doe_.
**john%%%%** will be your username (% represents four digits). After looking up other students' usernames, it is also possible to guess the range of four digits, thus reducing the number of unsuccessful brute force attempts.

---

# What I did :-

> ### Using Burp Suite to brute force username :-
- Intercepted the login request.
- Sent the login request to intruder - an in-built tool in burpsuite for authentication attacks.
- Removed all of the selected options, then drew attention to the username field.

![brute forcing username](/assets/images/XYZ%20-%20Brute%20Force/XYZ-intruder-username.png)

- Generated and loaded a list of usernames in the format I talked about previously - _john%%%%_.

### A simple bash script to generate usernames:-
> ```
for ip in {5000..6000}; 
do echo john$ip >> username.txt; 
done
```
This script will append numbers from 5000 to 6000 to _john_ and save it in _username.txt_ file.

---

- A useful website that lets you paste a whole bash script and have it explain - what it does and what each command in the script does.

[Explain Shell](https://explainshell.com/)

---

![loading-usernames](/assets/images/XYZ%20-%20Brute%20Force/burp-suite-load-username.png)

- Added **User doesn't exists** as grep match. 

![grep match](/assets/images/XYZ%20-%20Brute%20Force/burp-suite-user-grep.png)

> Since we have defined a grep match, Burp Suite alerts us if the authentication response differs from this one. The differ in response will mean that the username is correct, then we can try brute-forcing the **password**.

#### Invalid username:-
![invalid user](/assets/images/XYZ%20-%20Brute%20Force/intruder-attack-invalid-user.png)

#### Valid username:-
![valid user](/assets/images/XYZ%20-%20Brute%20Force/intruder-attack-valid-user.png)
- The message "Password is invalid" reveals that the username is accurate.

#### How we spotted correct username:-
The invalid flag helps us to find if your grep string was matched.

![correct usernsme](/assets/images/XYZ%20-%20Brute%20Force/how-we-knew-username.png)

---

# Brute Forcing Password
> Since we already know the username of our target, all that is left to do is successfully brute force the password.

- Let's assume our target's username is _john6969_. 

> **Every user account has a default password that consists of their username followed by four random numbers. And, believe me, every single user still uses the default password since they don't see the need to change it.**

### What I did:-
- Added the username of the target (which I brute forced earlier) and highlighted **password** instead of username.

![highlighting password](/assets/images/XYZ%20-%20Brute%20Force/highlighting-password-field.png)

### Using modified version of the previous script to generate password list:-
```
for ip in {0000..9999}; 
do echo john6969$ip >> password.txt; 
done
```

This script will append four digit random numbers to _john6969_ and save it in _password.txt_ file.

- Loaded the password list in **Payload Options**.
![loading pass list](/assets/images/XYZ%20-%20Brute%20Force/intruder-loading-password.png)

- Set the grep match to "Password is invalid" because that is what Authentication Response is when you enter a valid username but wrong password.

![password is invalid](/assets/images/XYZ%20-%20Brute%20Force/password%20is%20invalid.png)

- Initiated the intruder attack in Burp Suite.

#### Response with correct password:-
![correct password](/assets/images/XYZ%20-%20Brute%20Force/correct%20password.png)

#### Response with wrong password
![wrong password](/assets/images/XYZ%20-%20Brute%20Force/password%20wrong.png)

- Burp suite would detect it as soon as we requested the web server with the proper password because we had previously configured the grep match.

![password found](/assets/images/XYZ%20-%20Brute%20Force/password%20found.png)

**Now we have successfully compromised an account**

---

# Mitigations:-
> 1. Use Strong Passwords.
2. Limit Login Attempts. (Block IPs after numerous wrong attempts in a time limit)
3. Use CAPTCHAs.
4. By not diclosing if a user is valid.
"User or Password is invalid" - this Authentication Response is more secure. 

---