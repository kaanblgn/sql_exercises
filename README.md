# sql_exercises

## StanfordOnline SOE.YDB-SQL0001 Databases: Relational Databases and SQL SQL Movie-Rating Query Exercises

You've started a new movie-rating website, and you've been collecting data on reviewers' ratings of various movies. There's not much data yet, but you can still try out some interesting queries. Here's the schema:

Movie ( mID, title, year, director )
English: There is a movie with ID number mID, a title, a release year, and a director.

Reviewer ( rID, name )
English: The reviewer with ID number rID has a certain name.

Rating ( rID, mID, stars, ratingDate )
English: The reviewer rID gave the movie mID a number of stars rating (1-5) on a certain ratingDate.

Your queries will run over a small data set conforming to the schema. [View the database] (https://courses.edx.org/asset-v1:StanfordOnline+SOE.YDB-SQL0001+2T2020+type@asset+block/moviedata.html)

### Questions

Q1) Find the titles of all movies directed by Steven Spielberg.

> select title
  from movie
  where director = 'Steven Spielberg'
