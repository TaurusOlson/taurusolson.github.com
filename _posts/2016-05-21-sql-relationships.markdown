---
layout: post
title:  "SQL relationships"
date:   2016-05-21
categories: [note, data, sql]
---


## Introduction

This post is actually a note for myself about relationships in SQL. Most of the
time tables are related to others through relationships. The principle is to
avoid non-atomic values (first normal form) and data duplication (second normal
form). There are 3 relations ships: one-to-many, many-to-many and one-to-one.


## The data

We'll use fake data. Imagine we want to watch movies at the cinema. Let's
suppose I'm interested in adventure movies but I have no idea what movies are
worth watching. So I read the reviews of these movies, which is a score in 5
and chooses the one with the best score.

We have:

* a table for the movies called **movies** with the schema:

{% highlight sql %}
CREATE TABLE movies (
    id SERIAL PRIMARY KEY,
    title CHARACTER(50) NOT NULL,
    duration INTEGER CHECK (duration > 0),
    CONSTRAINT unique_title UNIQUE (title)
);
{% endhighlight %}

| id | title          | duration |
|----|----------------|----------|
| 1  | Don Juan       | 110      |
| 2  | Peter Pan      | 120      |
| 3  | The Lost World | 105      |
| 4  | Robin Hood     | 143      |


* a table for the genres called **genres** with the schema:

{% highlight sql %}
CREATE TABLE genres(
    id SERIAL PRIMARY KEY,
    genre CHARACTER(15) NOT NULL UNIQUE
);
{% endhighlight %}

| id | genre     |
|----|-----------|
| 1  | Romance   |
| 2  | Adventure |
| 3  | Fantasy   |


* table for the promotions called **reviews**:

{% highlight sql %}
CREATE TABLE reviews (
    id SERIAL PRIMARY KEY,
    movie_id INTEGER REFERENCES movies(id),
    review INTEGER
    FOREIGN KEY (movie_id) REFERENCES movies(id)
);
{% endhighlight %}


Note that we create the relationship by creating a foreign key. The foreign key
*movie_id* references the primary key *id* in the table **movies**.


| id | movie_id | review |
|----|----------|--------|
| 1  | 1        | 3      |
| 2  | 1        | 4      |
| 3  | 2        | 4      |
| 4  | 3        | 5      |
| 5  | 4        | 4      |
| 6  | 4        | 5      |



(Note: the examples used in this post are directly taken from a course on
CodeSchool, [The Sequel to SQL](https://www.codeschool.com/courses/the-sequel-to-sql))


## One-to-many 

One row in table A can relate to many rows in table B.
One row in table B relates to only one row in table A.

In our example:
one movie can have many reviews but a review adresses only one movie.

{% highlight sql %}
SELECT m.title, r.review 
FROM movies m JOIN reviews r 
ON m.id=r.movie_id;
{% endhighlight %}

| title          | review |
|----------------|--------|
| Don Juan       | 3      |
| Don Juan       | 4      |
| Peter Pan      | 4      |
| The Lost World | 5      |
| Robin Hood     | 4      |
| Robin Hood     | 5      |


## Many-to-many

One row in table A can relate to many rows in table B. One row in table B can
relate to many rows in table A. To define this relationship we need a table
C (the junction table) whose columns will be foreign keys referencing the
primary keys of tables A and B.

In this example:
A movie can have many genres and a genre can be associated to many movies.

{% highlight sql %}
CREATE TABLE movies_genres(
    movie_id INTEGER,
    genre_id INTEGER,
    FOREIGN KEY (movie_id) REFERENCES movies(id),
    FOREIGN KEY (genre_id) REFERENCES genres(id)
);
{% endhighlight %}


| movie_id | genre_id |
|----------|----------|
| 1        | 1        |
| 2        | 2        |
| 2        | 3        |
| 3        | 3        |
| 4        | 2        |


Now if we want to know the genres of the movies we have to use 2 joins:

{% highlight sql %}
SELECT m.id, m.title films, g.genre 
FROM movies m
INNER JOIN movies_genres mg
ON m.id=mg.movie_id
INNER JOIN genres g
ON g.id=mg.genre_id;
{% endhighlight %}


| id | films          | genre     |
|----|----------------|-----------|
| 1  | Don Juan       | Romance   |
| 2  | Peter Pan      | Adventure |
| 2  | Peter Pan      | Fantasy   |
| 3  | The Lost World | Fantasy   |
| 4  | Robin Hood     | Adventure |


## One-to-one

One row in table A relates to only one row in table B.
One row in table B relates to only one row in table A.
The primary key in B is the foreign key references the primary key in A.

In our example, we can imagine that each movie has a given budget and gross.
These information would be stored in the table **information**:

{% highlight sql %}
CREATE TABLE information (
    movie_id INTEGER PRIMARY KEY,
    budget INTEGER,
    gross INTEGER,
    FOREIGN KEY (movie_id) REFERENCES movies(id)
);
{% endhighlight %}


| movie_id | budget  | gross    |
|----------|---------|----------|
| 1        | 1000000 | 3000000  |
| 2        | 1500000 | 4000000  |
| 3        | 5000000 | 10000000 |
| 4        | 2000000 | 6000000  |


The important thing to notice here is that the primary key of this table
(*movie_id*) is also the foreign key references *id* in the table **movies**.

If we want to have the movies and these information we use a join on these
2 tables:

{% highlight sql %}
SELECT m.title, i.budget, i.gross 
FROM movies m JOIN information i
ON m.id = i.movie_id;
{% endhighlight %}


| title          | budget  | gross    |
|----------------|---------|----------|
| Don Juan       | 1000000 | 3000000  |
| Peter Pan      | 1500000 | 4000000  |
| The Lost World | 5000000 | 10000000 |
| Robin Hood     | 2000000 | 6000000  |


## What movie am I going to watch?

I want to watch the adventure movie with the best average review. First let's
create a view to determine the adventure movies:

{% highlight sql %}
CREATE VIEW adventure_movies_view AS
SELECT m.id, m.title, g.genre 
FROM movies m
INNER JOIN movies_genres mg
ON m.id=mg.movie_id
INNER JOIN genres g
ON g.id=mg.genre_id
where g.genre = 'Adventure';
{% endhighlight %}

Then we use join between the adventure movies and the reviews tables. We group by
movie and we average the score of the reviews:

{% highlight sql %}
SELECT amv.title, AVG(r.review) as avg_review
FROM adventure_movies_view amv JOIN reviews r 
ON amv.id=r.movie_id 
GROUP BY amv.title;
{% endhighlight %}

| title      | avg_review |
|------------|------------|
| Peter Pan  | 4.0        |
| Robin Hood | 4.5        |

OK. It seems I'm going to watch `Robin Hood`.
