# Project 2: Django-Twitter

## About
We are building a simple Twitter Clone using Django. This app will consist of just Users and Tweets (No hashtags, retweets, favourites, url-shortening or messages).

## What you have to learn

- Using `virtualenv`
- Using `postgres` and `psycopg2`
- Using the `django.contrib.auth` package
- Using `south` for schema-migrations
- Deploying Django apps on heroku
    + With the help of `django-toolbelt`

## Functional Requirements

### Users
- Users should be able to sign-up, login, logout and delete their accounts
- When their account is deleted, all their tweets are deleted too

### Tweets
- Tweets are made by a user and belong to that user
    + That user can delete their tweets
- Tweets contain mentions using the syntax `@username`
    + If `@username` is inculded in a tweet, the user mentioned is hyperlinked when the tweet is shown
- If a user changes his/her username from `old_username` to `new_username`:
    + The same user should still be hyperlinked on the tweets he/she was mentioned on using his old username
    + The content of the tweets must not be changed. So the tweet should still mention `@old_username` but link to the profile of `@username`

### Pages

#### Splash Page

- URL: `/`
- The welcome page of the website

#### Signup Page

- URL: `/user/register/`
- Users can signup with their email, password and username

#### Login Page

- URL: `/user/login/`
- Users can login with their email and password only
- On a successful long, redirect to the Feed Page

#### Profile Page [login-required]

- URL: `/user/<username>/`
- Shows the tweets made by the user
- Other logged in users have the option of following or unfollowing the user on this page

#### Followers and Following Page [login-required]

- URL: `/user/people`
- Show a list of users the user is following and is being followed by
    + All users are linked to their profiles

#### Mentions Page [login-required]

- URL: `/mentions`
- Shows the tweets that the user has been mentioned in

#### Feed Page [login-required]

- URL: `/`
- Shows the tweets made by the followers of this user

#### Settings Page [login-required]

- URL: `/user/settings`
- Users can edit their email, username, password, first name, last name and one-liner
- Users can also delete their account

#### All logged-in pages

- All logged-in pages should have a sidebar using which the users can create tweets

These are just the visible, interface pages that are present in the app. 
Feel free to add more URLs for logic purposes.

## Non-Functional Requirements

- Graphic Design doesn't matter, User Experience does
    + Using Bootstrap and HTML5-Boilerplate will be sufficient  
    + Make sure UX is simple and to the point
    + Speed matters, so no big images as backgrounds, keep things SIMPLE

- Code Structure and Quality matters
    + How you name packages, modules, funcitions, classes, variables
    + What you put in which file
    + How you name your URLs

- Website must be deployed on heroku

## Milestones

### Milestone 1 - User Management: Part 1 (3-4 days)
- Build the User Model
- Implement Signup and Login functionality
- Make sure you use a virtualenv and south by the end of this

### Milestone 2 - User Management: Part 2 (2-3 days)
- Implement the Delete and Settings functionality
- Implement Profile page
- Make sure you can deploy to Heroku by the end of this

### Milestone 3 - Tweets (2-3 days) 
- Implement the Tweet model
- Ability to create and delete tweets
- Ability to mention other people in tweets

### Milestone 4 - Feed Pages (1-2 days)
- Implement other pages by constructing the right queries

## Deadlines

### Checkpoint
- 2nd April, Wednesday
- Milestone 1 and 2 of the project must be complete

### Submission
- 9th April, Wednesday
- Full project should be complete