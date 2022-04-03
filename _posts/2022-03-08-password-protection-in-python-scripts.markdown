---
layout: post
title:  "Password protection in python scripts"
date:   2022-03-08 19:50
author: Arun
tags: [programming]
---


I came across an interesting problem recently. I was running a python script that would periodically update a remote database. It would connect to the database with a shared login credential and update a table. The script would run locally on multiple machines. Initially, the hashed db password was stored in the script, and extracted before being used to connect to the database.

It was not too long before I realized that anyone with access to the script can straight up read the password without so much as a how do you do. I looked up password protection on the internet, and there are some good approaches out there, like using hashing with hashlib, storing the password in env values, or using Keyring to store and fetch the password. You can find a good tutorial on using them [here](https://martinheinz.dev/blog/59)

However, these approaches face the same issue. Anyone with access to the machine the script is running on, can recover the password quite easily. I think only some form of secure handshake/authentication protocol can provide user authentication without storing the password in the script. The problem is, I do not know the first thing about setting up such a system. Especially with a DBMS with which I am unfamiliar. Also, I’m pretty sure the shared login is pretty much an immutable object (pun intended), and I cannot avoid using it. So I thought about it some more, and it occurred to me, that the only reason to keep the password in the script in the first place was because it used a connect_to_db module in the same script to connect to the database. 

![](/assets/images/db_conn_mod.png)


I realized that if I moved the entire connection module to a different file, and imported that file as a library in my script, my script would still have the db connection, but would not need the password in the script in any form. I could then use Keyring in the special connection library to retrieve a password that I store on the system separately, again using Keyring. But this just moves the problem to a different python file. To ensure that the password cannot be extracted from the connection lib file either, I hit upon the idea of compiling the lib file, and including only the byte file along with the main script. And that did it! The script works, and has at least somewhat better security than earlier.

![](/assets/images/conn.png)


Now, this is still not very secure. It is very easy to de-compile the pyc file and extract the password from it. Still, some security is better than no security.
