---
layout: post
title:  "From Scripter to Programmer - A learning journey"
date:   22019-08-22 10:21:09 +0100
author: James Eastham
categories: dotnet
---

Writing and understanding software is difficult, period. Whilst not quite akin to learning a new language the journey to being a good developer is certainly a long one. The syntax, the wording, the ‘best practices’, performance, extensibility… there’s plenty to think about. And that’s before you even get started on following good design principles.

For my first article, this is where I wanted to start. A bit of a deep dive into my own journey from how I got into code (one of the best decisions I’ve ever made) and the development path I’ve followed.

# In the beginning, there was support

My life in IT started out in a first-line support role, supporting a .NET windows based application running on a SQL back end. This was completely development free, and I went into this role with no development experience whatsoever.

I was actually accepted into university to study aeronautical engineering and before that wanted to be a pilot in the RAF. That dream died, but another passion soon reared its head.

I soon realized that the easiest way to debug 90% of the issues I encountered in support was straight in the database. With a bit of help from another tech-savvy support team member, MS SQL became my defacto choice for debugging. Now, how does a JOIN work again…

![Inner Join Diagram](https://thepracticaldev.s3.amazonaws.com/i/7vaybvxw5nf959w8kqa6.png)

An inner join returns records where there is a match in BOTH tables – If you’re interested
That lead me down the SSRS/SSIS path, and I moved to the companies data and reporting team. From there, my advanced SQL and performance tuning skills continued to develop and I also started to pick up a little bit of C# through SSIS scripting.

Looking back now, this was possibly one of the biggest downfalls of my career as a developer; writing code in SSIS was ordinarily really basic scripts used purely for manipulating data. It was all very procedural.

# Then came web forms
My journey took a turn towards web development when I moved to my next job after a brief, if you could count 12 months as brief, sabbatical. The company I joined implemented business automation solutions using ABBYY Flexicapture OCR solutions. With that, there was a lot of database administration (hello SQL) and building of tooling around the OCR software itself.

Web forms were fun, but MVC gave me a real buzz around web applications. Being able to write raw HTML alongside C# code was awesome, and the whole Model View Controller system made a lot of sense in my head.

That said, I still hated web DESIGN. HTML and CSS became the bane of my life.

Looking back at some of the code I wrote, it was still extremely procedural. I didn’t really follow any object-orientated programming practices aside from using objects to store data.

I had started to pick up layered application practices – front end, business logic, and data layers and tried to de-couple them as much as possible. Looking back at some of the code there was still a hell of a lot to be learned.

# Another sabbatical

After 6 months on a different kind of journey (it turns out this backpacking malarkey is actually kind of fun), I moved companies again. This time, to a startup. No existing code no pre-existing strategies, no old legacy code… a blank slate.

This was just over 12 months ago, and in that time I’ve improved more as a developer than in any of the last 6 years.

## But what changed?

I started to take much more interest in knowledge building. Previously, I used stack overflow A LOT, only looked up what I needed to know right at that time and then instantly forgot it. There was no concept of ‘how’ code should be written.

My journey has taken me through plenty of books. From basic programming principles (Practical Object-Oriented Design: An Agile Primer Using Ruby) all the way through to design best practices (Domain-Driven Design: Tackling Complexity in the Heart of Software).

I follow podcasts (No Dogma being my favorite), I follow .NET core release notes, I care about performance, I try to follow SOLID principles.

I’ve even started to expand myself out beyond the comfort blanking that is the .NET stack. Go being the current flavor of the month.

I care about unit testing. CI/CD and automated pipelines have removed almost all of my “oh shit I forgot to copy file X on deployment”. Add a fantastic branching and merging strategy on top of that and any releases I put out are now almost seamless.

And don’t even get me started on Docker, Kubernetes and microservice-based architecture. Those two tools plus building small microservices over giant monoliths has made me infinitely more productive. It also has the added benefit of making so many of our internal tools so much more repeatable.

# So what’s the takeaway?

I always like to write articles that give people takeaways and actionable tips.

Whilst this article is a deep dive into my personal journey, I think there is still one key takeaway.

Never settle! Always keep improving! Once you think you’re perfect, you’ve failed!

Technology moves at such a fast pace nowadays. In 12 months time who knows what languages/deployment strategies people will be using.

Span<T> being one of the biggest developments in recent months that, in my previous way of developing, I would have just completely missed. Now it’s becoming a huge part of my development toolkit.

Plus, knowing the features that are coming in C# 8 and being able to test them right now is wonderful. Nullable reference types being my absolute pick of the bunch on a feature front.

There’s so much to learn, so many best practices and so many different ways of achieving the same goal in software. What I’m going to try and chart in this blog is my own learning journey. A lot of my code may not be perfect, or the most performant (and please comment if you see any shit). But I’m still on a journey, a journey to becoming a better developer.

And on that note, I’m signing off for the day.

James

