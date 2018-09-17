---
layout: post
title:  "Brown Dust : Understanding Attack Priority"
image: ''
date:   2018-09-16 00:12:00
tags:
- Brown Dust (Intermediate)
description: 'Getting to know the Brown Dust Attack Priority in detail'
categories:
- Brown Dust (Mechanics)
---

Most of my posts aren't going to actually introduce Brown Dust. You will have to be somewhat familiar with the game to understand what I'm posting about (if the tags include 'Basic', that might be a post that can be understood without even knowing what Brown Dust is, otherwise, you'd have to know the game at least a little)

---

## Introduction

Unlike most other games, Brown Dust has a 3x6 matrix like board (see image below). Not all cells are used, which can lead to strategic formations. In this post, I will only elaborate on how ***attack priority*** works on the board.

<img src="../uploads/browndust-attack-priority-sample-board.jpg">

'So which row/cell does the unit attack?' will be the main topic elaborated. In order to ease the explanation, I'm going to simplify the board into just rows for now.

<img src="../uploads/browndust-attack-priority-simplified-rows.jpg">

---

## Attack Priority (w/ Rows)

If your current unit is on the Middle Row and if there is a unit on the enemy's Middle Row, then it's pretty intuitive that it attacks the unit on the enemy's Middle Row. However, what happens if there are no units on the enemy's Middle Row, but on Upper Row and Lower Row? Well, in Brown Dust, the priority goes **down**. For example,

<img src="../uploads/browndust-attack-priority-row-priority-example.jpg">

It will attack the row that's colored <span style="color:red">red</span>. The only way to forcefully change the row it's naturally going to attack is by using the skills :

* <span style="color:red">Focus Fire</span>
* <span style="color:blue">Ignore Taunt</span>
* <span style="color:green">Taunt</span>

---

## Focus Fire / Ignore Taunt / Taunt

(because I use the KR client, it's in korean, but I think you guys can figure it out with the icon image)

<img src="../uploads/browndust-attack-priority-priority-changing-skills.jpg">

**IMPORTANT**

How I ordered these skills are related to priority. In a sense, if there is a <span style="color:red">***Focus Fire***</span> active on the board, <span style="color:red">***it doesn't matter if there are taunts or ignore taunts ability active, because it will target the unit that has the Focus Fire ability active***</span>. If there are no <span style="color:red">***Focus Fire***</span> ability active on the board, then <span style="color:blue">***Ignore Taunt***</span> is up next. Even if there is taunt on board, it will attack the same cell(s) on the board even if there is a <span style="color:green">***Taunt***</span> active on board. If there are no <span style="color:red">***Focus Fire***</span> and <span style="color:blue">***Ignore Taunt***</span> available on the board, then <span style="color:green">***Taunt***</span> takes highest priority and any unit that attack will attack the unit that has <span style="color:green">***Taunt***</span> active.

For the sake of simplicity say there are only <span style="color:red">***Focus Fire***</span> or only <span style="color:green">***Taunt***</span> on the board. Everything seems fine if there is only 1 <span style="color:red">***Focus Fire***</span> or <span style="color:green">***Taunt***</span> on the map, but what happens if there are 2? For this we can no longer use only rows, but we will have to consider columns as well. Here is the <span style="color:red">***Focus Fire***</span> and <span style="color:green">***Taunt***</span> priority shown with the board.

<img src="../uploads/browndust-attack-priority-focusfire-taunt-priority.jpg">
(Lower the number, the higher the priority)

The question I had when I first saw this focus fire and taunt priority board was : ***Does the position (cell) of the unit that is about to attack matter?*** The answer is **NO**. The only factor that's taken into account is the position of the unit that currently has <span style="color:red">***Focus Fire***</span> or <span style="color:green">***Taunt***</span> active.

Perhaps, examples will help...

* X = the unit that is going to attack
* A,B,C = the unit that is going to receive the attack (i.e. enemy unit)
* () = no skills active
* (FF) = Focus Fire active
* (IT) = Ignore Taunt active
* (T) = Taunt active

**Example 1**

<img src="../uploads/browndust-attack-priority-sample-1.jpg">

Pretty straight forward. It follows the priority as shown beforehand.

**Example 2**

<img src="../uploads/browndust-attack-priority-sample-2.jpg">

Due to <span style="color:blue">***Ignore Taunt***</span> by unit X, it will attack B even though A as <span style="color:green">***Taunt***</span> active.

**Example 3**

<img src="../uploads/browndust-attack-priority-sample-3.jpg">

If A didn't have <span style="color:red">***Focus Fire***</span>, it will attack C. However, A's <span style="color:red">***Focus Fire***</span> takes higher priority, so it will attack A.

**Example 4**

<img src="../uploads/browndust-attack-priority-sample-4.jpg">

Now C has <span style="color:red">***Focus Fire***</span> and A still has <span style="color:red">***Focus Fire***</span>. When A and C both don't have <span style="color:red">***Focus Fire***</span>, <span style="color:blue">***Ignore Taunt***</span> takes over and attacks C, but since both has <span style="color:red">***Focus Fire***</span> active it follows the priority shown previously, so it will attack A.

**Example 5**

<img src="../uploads/browndust-attack-priority-sample-5.jpg">

Even without <span style="color:blue">***Ignore Taunt***</span>, it will attack <span style="color:red">***Focus Fire***</span> active unit.

---

Before concluding, I'd like to say that <span style="color:red">***Focus Fire***</span> ability isn't activated by the enemy. It's a debuff given to an enemy unit by an ally unit. This is why it has higher priority. Since there's no point of having a <span style="color:red">***Focus Fire***</span> ability without a <span style="color:blue">***Ignore Taunt***</span> (what's the point of having it when you are gonna give <span style="color:red">***Focus Fire***</span> to an enemy unit ***Taunt*** unit that already has the highest priority?), you will find that every single unit with <span style="color:red">***Focus Fire***</span> ability have <span style="color:blue">***Ignore Taunt***</span> ability as well.

If there are any confusing parts or any parts you'd like more elaboration, leave a comment or contact me in any ways possible :^)
