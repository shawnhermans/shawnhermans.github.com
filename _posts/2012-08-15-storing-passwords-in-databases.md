---
layout: post
title: "Storing Passwords in Databases"
author: Shawn Hermans
date: 2012-08-15 12:00:00 -0600
categories: [passwords, security, databases]
---
Originally posted as an answer to [Encryption: What is the best way to store username and password in a database?](https://www.quora.com/Encryption-What-is-the-best-way-to-store-username-and-password-in-a-database/answer/Shawn-Hermans?srid=hLq3)

The previous answers to this question were very good, but I want to make sure I stress two things.  When it comes to storing passwords using hashes, always use SALT (pepper is optional :). The reason is because if you only use a hash function, someone could use a rainbow table to break short or common passwords. The second thing I would recommend is never, ever, ever invent your own way of doing this.  Use something like bcrypt or some other library that has been fully tested and vetted.  Why? Cryptographic systems are hard to create.  I would suggest reading [http://www.schneier.com/blog/archives/2011/04/schneiers_law.html](http://www.schneier.com/blog/archives/2011/04/schneiers_law.html) to give some perspective.
