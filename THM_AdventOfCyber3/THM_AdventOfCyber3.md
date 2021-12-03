---
title: TryHackMe - Advent of Cyber 3 (2021)
date: "2021-12-01"
---

TryHackMe - Advent of Cyber 3 (2021)

<!-- more -->

# [TryHackMe - Advent of Cyber 3 (2021)](https://tryhackme.com/room/adventofcyber3)



The TryHackMe - Advent of Cyber 3 just launched today and there are a lot of prizes to win.

The room consists of completing a new, beginner friendly security exercise every day leading up until Christmas;

I will be updating this walkthrough daily for each challenge.

##  [Day 1] **`Web Exploitation`** Save The Gifts

The room is based on Insecure Direct Object Reference or IDOR for short.

The explanations are all provided on the challenge page.

![](THM_AdventOfCyber3.assets/image-20211202004735054.png)

### **Challenge Walkthrough**

After finding Santa's account, what is their position in the company?

![](THM_AdventOfCyber3.assets/image-20211202005102738.png)

Santa’s `user_id` is `1`

>The Boss!

After finding McStocker's account, what is their position in the company?

![](THM_AdventOfCyber3.assets/image-20211202005343221.png)

McStocker's `user_id` is `2`

>Build Manager

After finding the account responsible for tampering, what is their position in the company?

![](THM_AdventOfCyber3.assets/image-20211202005454133.png)

The account responsible for tampering is `Grinch’s` and `user_id` is `9`

>Mischief Manager

What is the received flag when McSkidy fixes the Inventory Management System?

![](THM_AdventOfCyber3.assets/image-20211202010129552.png)

Once all the actions are revoked, the flag is displayed.

![](THM_AdventOfCyber3.assets/image-20211202010016166.png)

>THM{Fake_Flag}



