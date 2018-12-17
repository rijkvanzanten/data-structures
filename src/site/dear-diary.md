---
layout: base.ejs
title: Dear Diary | Data Structures
---

# Dear Diary

_[GitHub](https://github.com/rijkvanzanten/ds-fa-2) • [Live Demo](https://ds-fa-2.rijks.website)_

## Brief

You will capture and store data for "Dear Diary": a source of semi-structured, qualitative data. The data should be stored in Amazon DynamoDB and queried using Node. You will design and create an interface to display relevant diary entries.

## The Verge

I decided to setup a [Pocket](https://getpocket.com) meets [The Verge](https://theverge.com). The data would be mini blog posts and other snippets to media online. This kind of information would be well suited for document based database as there are a lot of nested and unknown-length "columns", like the content itself, the keywords, and the possible existence of an image and an outgoing link.

I'm an avid reader of [The Verge](https://theverge.com) and I kinda dig what they were trying to do with the posts layout. I wanted to try if I could create a similar layout using the new CSS Grid system to improve on it.

![The Verge](/dd-verge.png)

## Posts

I wrote the individual content pieces [in Markdown files first](https://github.com/rijkvanzanten/ds-fa-2/tree/master/install/posts), after which I "shoot" them up to the DB using a single installer script I wrote. The installer will save the metadata in the "gray matter", convert the markdown to html, bundle the two together as an object, and save the whole thing to Amazon

## Working with DynamoDB

I don't like it. At all. I think I missed something, or I haven't seen the light yet, cause it was just pain all around:

* Documentation is very hard to navigate
* Terminology is hard to understand (`KeyConditionExpression`?, `ExpressionAttributeNames`?!)
* No way to "search"\*
* Very slow\*\*

\* I know that this isn't necessarily a thing that document based databases are made to do, but even then  
\*\* I'm assuming this is due to the fact that I'm on a free plan, but I was honestly surprised how long it takes to retrieve a single item by ID. I can't imagine this is supposed to be this way. Opening a super small piece of content feels like a WordPress installation sweating away.

## Interface

As mentioned, I tried replicating the Verge's homepage layout, as a challenge to learn CSS Grid. I figured that this would make a good homepage, as it's a big overview of all the available pieces of content. Clicking on the content previews will open up a detail page with a small exempt of the page and a big link to the original piece. From this detail view, the user can click through to the keywords as well, opening an archive of all other snippets that have the same keyword associated with it.

## Result

![Screenshot 1](/dd-1.png)

![Screenshot 2](/dd-2.png)

![Screenshot 3](/dd-3.png)

[Checkout the live demo!](https://ds-fa-2.rijks.website)

## Conclusions / final thoughts

* I'm probably going to steer clear of DynamoDB going forward.
* Saving (blog) posts in a document based database works really well. HOWEVER, I think it could be even nicer to use a hybrid approach, where you would save the post date and some metadata in a relational database (so you can search through it performantly) and the post content itself in the document based DB
* CSS Grid is awesome
