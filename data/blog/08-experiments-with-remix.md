---
title: 'Experiments with Remix'
summary: "I've recently been learning about Remix and have found it to be my new go-to framework. As I began building a sample recipe management application using Remix, I decided to document my experience through a series of blog posts. This article marks the beginning of that journey."
date: 2023-04-02 16:00
tags: ['TypeScript']
---

## Summary

I've recently been learning about Remix and have found it to be my new go-to framework. As I began building a sample recipe management application using Remix, I decided to document my experience through a series of blog posts. This article marks the beginning of that journey.

<img src="/static/images/experiments-with-remix/recipe-book-img.png" alt="recipe book demo"/>

I took this course, [Up and Running with Remix](https://egghead.io/courses/up-and-running-with-remix-b82b6bb6) by Kent C. Dodds, to gain a comprehensive understanding of Remix. The application I built served as a practical exercise to put my learnings from the course into practice, further solidifying my grasp of Remix. I highly recommend checking out this course for anyone interested in learning about Remix.

## My Usecase

As a starting point, I set out to construct a sample application that fulfills the following criteria:

- Scaffolds a starter template with authentication support
- Ensures data persistence in a database
- Familiarizes me with Remix's nested routing pattern
- Enables users to create, read, update, and delete recipes

Now, let's dive into how I tackled each of the requirements mentioned above.

## Scaffolding the app

To get things rolling, I began by scaffolding an app using the [Remix Indie Stack](https://github.com/remix-run/indie-stack). This choice proved advantageous, as it included Tailwind for styling, the Prisma ORM for connecting to a lightweight SQLite database, pre-built login and sign-up flows, and a ready-to-use setup for deploying my finished application to [Fly.io](https://fly.io/).

## Modeling the database schema

Next, I began crafting my database models. In the application, users can create and store their own recipes, with each recipe consisting of one or more ingredients and instructions. With this structure in mind, I developed the following schema model for my database.

You can view the prisma model [here](https://github.com/niki4810/recipe-book/blob/main/prisma/schema.prisma#L29-L70)

Furthermore, I needed to establish a [relationship between the User](https://github.com/niki4810/recipe-book/blob/main/prisma/schema.prisma#L18) and Recipes tables, as a user can create one or more recipes, and each recipe belongs to a single user.

## Adding routes

Undoubtedly, the most challenging aspect for me was grasping how nested routes function in Remix. It took me some time to fully comprehend the concept. I gained valuable insights by watching this [video](https://www.youtube.com/watch?v=ds_evK0jeHM) by Sam Selikoff.

While there's still much to learn and enhance regarding nested routes, I eventually established the following routes tailored to my specific use case:

| Route                                                  | Description                                          |
| ------------------------------------------------------ | ---------------------------------------------------- |
| /recipes                                               | An index route that displays a list of users recipes |
| /recipes/new                                           | A route to add a new recipe                          |
| /recipes/$recipeId/edit                                | Route to edit recipe by Id                           |
| /recipes/$recipeId/details                             | Layout route for recipe details                      |
| /recipes/$recipeId/details/ingredients                 | Nested route that displays recipe ingredients        |
| /recipes/$recipeId/details/ingredients/new             | Nested route to add a new ingredient to a recipe     |
| /recipes/$recipeId/details/ingredients/$ingredientId   | Nested route to edit an existing ingredient          |
| /recipes/$recipeId/details/instructions                | Nested route that displays recipe instructions       |
| /recipes/$recipeId/details/instructions/new            | Nested route to add a new instruction to a recipe    |
| /recipes/$recipeId/details/instructions/$instructionId | Nested route to edit an existing instruction         |
| /login                                                 | login route                                          |
| /join                                                  | sign up route                                        |

## Adding the CURD operation

Once I established my routes and database schema, the rest of the work proceeded smoothly. Loaders and actions are the two fundamental concepts in Remix that facilitated this process. With nested routing, each route segment was responsible for loading its respective content, which I found to be a convenient feature. Furthermore, I was pleasantly surprised by the amount of code I didn't have to write, particularly for form events and error handling, as Kent C. Dodds highlights in his course.

I was also impressed that all the features of my application are progressively enhanced and function seamlessly even when JavaScript is disabled in the user's browser. This impressed me greatly, as I didn't have to write any additional code to achieve this level of performance and accessibility.

## Final Thoughts

While this application is an MVP, I plan to incorporate additional features such as image uploading and instruction reordering using a drag-and-drop UI interaction. As I continue to learn more about advanced Remix features, I hope to implement them in this project. I'll be providing further details about these updates in my upcoming blog post.

<img src="/static/images/experiments-with-remix/recipe-book.gif" alt="recipe book demo"/>

> The completed code is available in the following repository: [recipe-book](https://github.com/niki4810/recipe-book)

## References

- [Up and Running with Remix by Kent C. Dodds](https://egghead.io/courses/up-and-running-with-remix-b82b6bb6)
- [Routing Patterns in Remix by Sam Selikoff](https://www.youtube.com/watch?v=ds_evK0jeHM)
