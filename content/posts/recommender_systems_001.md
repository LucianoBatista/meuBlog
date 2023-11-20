---
layout: post
title: "Understanding recommender systems"
subtitle: "Chapter 01"
date: 2023-11-20
author: "Luciano"
authorLink: "https://www.linkedin.com/in/lucianobatistads/"
featuredImage: "img/recsys-001.png"
math:
  enable: true
tags:
  - AI
  - Recommender System
categories: [Tutorials]
draft: false
---

Welcome to the beginning of an adventure on recommendation systems! I'm diving into the book _Practical Recommender Systems_ (you can find the link at the end) and sharing my notes here as part of a practice exercise.

In this post, I'll focus on **Chapter 1**. It delves into the fundamental concept of what a recommender system is and introduces key ideas that will likely resurface throughout the book.

To save some typing, I'll use the abbreviation **RecSys** for Recommender Systems in the upcoming posts, considering we'll mention it frequently.

## Real-life recommendations

Recommendations have become integral to our digital experience. Consider these everyday scenarios:

- You grab your phone, open **Spotify**, and pick a track to play. The soundtrack, the song itself, or even the artist might have been suggested to you.
- Launch **Google Chrome** on your smartphone, and the homepage might display news aligned with your typical search patterns.
- Visit your **YouTube** account, and the suggested videos are based on your usual watch history.

However, not everything presented is a recommendation. These same apps often display advertisements, most of which aren't tailored specifically to you; they're essentially sponsored content on these platforms and are sometimes targeted at a broader audience segment.

The distinction between recommendations and commercials (advertisements) lies primarily in their content intent.

> _A recommendation is calculated based on what the active user like's, what others have liked in the past, and what's oftem requested by the receiver. A commercial is given for the benefit of the sender and is usually pushed on the receiver._

## Where recommender systems shiny?

RecSys are crucial for products with vast item catalogs. Their primary role lies in aiding users in discovering the best items suited to their preferences. For instance, imagine navigating through millions of Netflix series without personalized recommendations—relying solely on filters would make finding a suitable match incredibly challenging. Similarly, if your YouTube feed fails to display videos aligned with your interests, it becomes more arduous to locate desired content.

However, the significance of RecSys isn't limited to user convenience; it profoundly benefits the platforms hosting such content. By optimizing user experiences, platforms can achieve their objectives, whether it's increasing user engagement (leading to more ad views), prolonging subscriptions, or driving more sales.

{{< admonition type=warning title="Opinion" open=true >}}
From my perspective, these platforms often prioritize financial goals alongside user preferences. This could mean that recommendations might not always align solely with your taste, as they're influenced by the platform's objectives. Consequently, certain content may not be recommended to you despite being relevant to your interests.
{{< /admonition >}}

## Recommender system definitions

As some concepts will be frequently used throughout the chapters, the author provides some definitions:

- **Prediction:** A prediction denotes an estimation of how much the user would rate/like an item.
- **Relevancy:** Relevancy involves arranging items based on what's most pertinent to the user at a given moment. It's determined by context, demographics, and (predicted) ratings.
- **Recommendation:** Refers to the top N most relevant items.
- **Personalization:** Involves integrating relevancy into the presentation.
- **Taste profile:** Represents a list of defining characteristics.

These definitions can be validated within the following pipeline:

{{<image width=800 src="https://i.imgur.com/1pKjwIk.png">}}

It's crucial to emphasize that the system I'm referring to is already operational, which implies that we've gathered user data and constructed our models. Two very important steps that take a lot of our times.

Regarding the data itself, it encompasses a wide array of diverse information types. This includes:

- **Historical data:** Information gathered from past interactions or behaviors.
- **Content data:** Details about the items or content available within the system.
- **Context data:** Varied contextual information such as the device being used, whether the person is in motion or stationary, time of day, etc.
- **Demographics:** A comprehensive range of demographic details including age, gender, political inclinations, income levels, nationality, and more.

All these different categories of data contribute significantly to the functioning and accuracy of our models.

## Taxonomy of RecSys

This is for me, **the most important session of the chapter**. As RecSys are not just algorithms, there is a lot of pieces we need to pull out together in order to make it work. These seven points serve as a framework we can use at starting point of our RecSys.

### Domain

Absolutely, the domain of a recommendation system dictates the type of content it suggests. For instance:

- Netflix focuses on movies and TV series.
- Amazon recommends products.
- Spotify suggests music and podcasts.

Understanding the domain is crucial because it determines the impact of a poor recommendation:

- Recommending a bad music track might disappoint but won't cause significant harm.
- However, suggesting a bad foster parent for children in need could have severe consequences.

### Purpose

The purpose of a recommendation system benefits both the user and the system itself. Let's take Netflix as an example:

- **User:** Their goal is to find relevant content to watch and enjoy.
- **System:** Its goal is to maintain user subscriptions for as long as possible, ensuring continued revenue.

In some cases, a _proxy goal_ might be employed to indirectly measure the primary purpose. However, selecting a suitable _proxy goal_ is crucial; an inappropriate one can lead to undesired outcomes.

{{< admonition type=tip title="Note" open=true >}}
_The author suggest the book "Weapons of Math Destruction" provides further examples of proxy goals, offering valuable insights into how these indirect measures can sometimes have unintended consequences._
{{< /admonition >}}

Using a _proxy goal_ helps gauge or approximate the primary objective indirectly. In the case of Netflix, it might be measuring user engagement or time spent watching content as a means to assess whether users are finding the recommended content appealing enough to continue their subscription.

### Context

The context is also very important, because will set some variables regarding the environment in which the consumer receives a recommendation, for example:

- **current location**
- **time of the day**
- **what the consumer is doing**
- **weather conditions**
- **user's mood**

### Personalization level

We can think of RecSys using many personalization levels, like in the figure below:

{{<image width=800 src="https://i.imgur.com/rf2KlcB.png">}}
As the arrow goes up, we increase the complexity of our system.

### Privacy and trustworthiness

This is a very important "pauta" to address. Is here where the company that is implementing the RecSys needs to understanding what is allowed and whats not allowed to or expose from user's data.

Talk about manipulation and recommendations, this shake the trustness of the service by the user.

### Interface

Interface of a recommender system depicts the kind of input and output it produces.

#### Input

The system will allow for certain interactions with the platform, this data can be used by the RecSys.

#### Output

The way the information comes back to the user:

- predictions
- recommendations
- filtering

### Algorithms

The author split the algorithms presented in this book into to categories:

- **collaborative filtering**: employ usage data.
- **content-based filtering**: use content metadata and user profiles to calculate recommendations.
- **hybrid**: a mix of the two types.

#### Collaborative filtering

These types of algorithms attempt to identify preference segments. Each time a new user searches for an item, the algorithm places this user into relevant segments or "buckets." This allows for a comparison between the user's taste and that of others who share similar preferences.

{{<image width=800 src="https://i.imgur.com/2qDDWbb.png">}}

#### Content-based filtering

The system utilizes the metadata available for the offered items. For instance, if you give a 5-star rating to the movie "Interstellar" and a 2-star rating to "Game of Thrones," it uses this data to construct a profile that reflects your preferences in movies or TV Series.

#### Hybrid

Hybrid algorithms combine collaborative and content-based approaches to form a powerful strategy. It's crucial to note that the more complex your strategy becomes, the harder it is to explain its results.

For example, in the Netflix Prize, competition hosted on Kaggle, the wining solution was too complex and expansive to put into production. The costs with computational resources was not worthy. Besides, probably the interpretation level was also hurt by this solution.

## Summary

The most important takeaway from this chapter, for me, is the framework provided. It offers a structured approach to constructing a RecSys.

While algorithms are important, a successful RecSys requires assembling various pieces of the puzzle.

{{<image width=800 src="https://i.imgur.com/hnbBvSS.png">}}

The author also introduces the app that will be used throughout the book, but I'll provide a better description in the next post.

## Referencies

- book: Practical Recommender Systems
- course: Recommender Systems - Introduction (Coursera)
