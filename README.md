# sql_exercises

## StanfordOnline SOE.YDB-SQL0001 Databases: Relational Databases and SQL

You've started a new movie-rating website, and you've been collecting data on reviewers' ratings of various movies. There's not much data yet, but you can still try out some interesting queries. Here's the schema:

Movie ( mID, title, year, director )
English: There is a movie with ID number mID, a title, a release year, and a director.

Reviewer ( rID, name )
English: The reviewer with ID number rID has a certain name.

Rating ( rID, mID, stars, ratingDate )
English: The reviewer rID gave the movie mID a number of stars rating (1-5) on a certain ratingDate.

Your queries will run over a small data set conforming to the schema. [View the database](https://courses.edx.org/asset-v1:StanfordOnline+SOE.YDB-SQL0001+2T2020+type@asset+block/moviedata.html)

###  SQL Movie-Rating Query Exercises

#### Q1) Find the titles of all movies directed by Steven Spielberg.

```
select title
from movie
where director = 'Steven Spielberg'
```
 
#### Q2) Find all years that have a movie that received a rating of 4 or 5, and sort them in increasing order.
```
select distinct year
from movie,rating
where movie.mid = rating.mid
and stars >= 4
order by year
```
#### Q3) Find the titles of all movies that have no ratings.
```
select title
from movie
left join rating r
on movie.mid = r.mid
where stars is null
order by movie.mid
```
#### Q4) Some reviewers didn't provide a date with their rating. Find the names of all reviewers who have ratings with a NULL value for the date.
```
select name
from reviewer
left join rating r on reviewer.rid = r.rid
where ratingdate is null
```
#### Q5) Write a query to return the ratings data in a more readable format: reviewer name, movie title, stars, and ratingDate. Also, sort the data, first by reviewer name, then by movie title, and lastly by number of stars.
```
select  name AS reviewer_name,
        title AS movie_title,
        stars,
        ratingdate
from rating
left join reviewer r on rating.rid = r.rid
left join movie m on rating.mid = m.mid
order by reviewer_name,
         movie_title,
         stars
```
#### Q6) For all cases where the same reviewer rated the same movie twice and gave it a higher rating the second time, return the reviewer's name and the title of the movie.
```
SELECT name,
       title
FROM ((rating r1
    join movie m on r1.mid = m.mid)
    join reviewer r on r1.rid = r.rid)
         INNER JOIN rating r2 on r2.rid = r1.rid and r2.mid = r1.mid
WHERE r1.rid = r2.rid
  AND r1.mid = r2.mid
  AND r2.stars > r1.stars
  AND r2.ratingdate > r1.ratingdate
GROUP BY name,title
```
#### Q7) For each movie that has at least one rating, find the highest number of stars that movie received. Return the movie title and number of stars. Sort by movie title.
```
select title,
       max(stars)
from rating join movie m on rating.mid = m.mid
where rating.ratingdate is not null
group by title
```
#### Q8) For each movie, return the title and the 'rating spread', that is, the difference between highest and lowest ratings given to that movie. Sort by rating spread from highest to lowest, then by movie title.
```
select title,
       (max(stars) - min(stars)) as rating_spread
from rating join movie m on rating.mid = m.mid
group by title
order by rating_spread desc,
         title
```
#### Q9) Find the difference between the average rating of movies released before 1980 and the average rating of movies released after 1980. (Make sure to calculate the average rating for each movie, then the average of those averages for movies before 1980 and movies after. Don't just calculate the overall average rating before and after 1980.)
```
SELECT (-AVG(after.avgafter80) + AVG(before.avgbefore80))
FROM
    (
        SELECT AVG(stars) as avgbefore80
        FROM Rating JOIN Movie USING(mID)
        WHERE year < '1980'
        GROUP BY title
    ) AS before
        ,
    (
        SELECT AVG(stars) as avgafter80
        FROM Rating JOIN Movie USING(mID)
        WHERE year > '1980'
        GROUP BY title
    ) AS after
			
-- additional solution
-- select abs(avg(before.average)-avg(after.average))
-- from
--     (select avg(stars) as average
--      from movie left join rating r on movie.mid = r.mid
--      where year < '1980'
--      group by movie.mid) as before,
--     (select avg(stars) as average
--      from movie left join rating r on movie.mid = r.mid
--      where year > '1980'
--      group by movie.mid) as after
```
###  SQL Movie-Rating Query Exercises Extras
#### Q1) Find the names of all reviewers who rated Gone with the Wind.
```
select distinct name
from rating
    join movie m on rating.mid = m.mid
    join reviewer r on rating.rid = r.rid
where m.mid = 101 and stars is not null
```
#### Q2) For any rating where the reviewer is the same as the director of the movie, return the reviewer name, movie title, and number of stars.
```
select m1.title,
       m1.director
from movie m1
where m1.director in (select m2.director
                     from movie m2
                     group by m2.director
                     having count(*)>1)
order by m1.director,
         m1.title
```
#### Q3) Return all reviewer names and movie names together in a single list, alphabetized. (Sorting by the first name of the reviewer and first word in the title is fine; no need for special processing on last names or removing "The".)
```
select name from reviewer
union
select title from movie
order by name
```
#### Q4) Find the titles of all movies not reviewed by Chris Jackson.
```
select title
from movie
where mid not in (select mid
                  from rating
                           inner join reviewer r on rating.rid = r.rid
                  where name = 'Chris Jackson');
```
#### Q5) For all pairs of reviewers such that both reviewers gave a rating to the same movie, return the names of both reviewers. Eliminate duplicates, don't pair reviewers with themselves, and include each pair only once. For each pair, return the names in the pair in alphabetical order.
```
select distinct v1.name, v2.name
from movie m
inner join rating r1 on m.mid = r1.mid
inner join rating r2 on m.mid = r2.mid
inner join reviewer v1 on r1.rid = v1.rid
inner join reviewer v2 on r2.rid = v2.rid
where v1.rid <> v2.rid and v1.name< v2.name
order by v1.name
```
#### Q6) For each rating that is the lowest (fewest stars) currently in the database, return the reviewer name, movie title, and number of stars.
```
select name,
       title,
       stars
from rating
    join reviewer r on rating.rid = r.rid
    join movie m on rating.mid = m.mid
where rating.stars = (select min(stars) from rating)
```
#### Q7) List movie titles and average ratings, from highest-rated to lowest-rated. If two or more movies have the same average rating, list them in alphabetical order.
```
select title,
       avg(stars) as average
from rating
    join movie m on rating.mid = m.mid
group by title
order by average desc,
         title
```
#### Q8) Find the names of all reviewers who have contributed three or more ratings. (As an extra challenge, try writing the query without HAVING or without COUNT.)
```
select name
from (select rid,
             count(mid) as total_cont
      from rating
      group by rid) as f
join reviewer on f.rid = reviewer.rid
where total_cont >= 3
```
#### Q9) Some directors directed more than one movie. For all such directors, return the titles of all movies directed by them, along with the director name. Sort by director name, then movie title. (As an extra challenge, try writing the query both with and without COUNT.)
```
select m1.title,
       m1.director
from movie m1
where m1.director in (select m2.director
                     from movie m2
                     group by m2.director
                     having count(*)>1)
order by m1.director,
         m1.title
```
#### Q10) Find the movie(s) with the highest average rating. Return the movie title(s) and average rating. (Hint: This query is more difficult to write in SQLite than other systems; you might think of it as finding the highest average rating and then choosing the movie(s) with that average rating.)
```
select m.title,
       avg(stars) as avg_stars
from rating
join movie m on rating.mid = m.mid
group by m.title
having avg(stars) =
       (select max(avg_stars)
        from
            (select avg(rating.stars) as avg_stars
             from rating
             group by rating.mid) as t
       )
```
#### Q11) Find the movie(s) with the lowest average rating. Return the movie title(s) and average rating. (Hint: This query may be more difficult to write in SQLite than other systems; you might think of it as finding the lowest average rating and then choosing the movie(s) with that average rating.)
```
select m.title,
       avg(stars) as avg_stars
from rating
join movie m on rating.mid = m.mid
group by m.title
having avg(stars) =
       (select min(avg_stars)
        from
            (select avg(rating.stars) as avg_stars
             from rating
             group by rating.mid) as t
       )
order by m.title
```
#### Q12) For each director, return the director's name together with the title(s) of the movie(s) they directed that received the highest rating among all of their movies, and the value of that rating. Ignore movies whose director is NULL.

This answer is out of the scope for these exercises because I used distinct on of postgresql dialect rather than using the sqlite because why not :)
```
select distinct on(m.director)m.director,
       m.title,
       r.max_review
from movie m
join (select r.mid,
             max(r.stars) as max_review
      from rating r
      group by r.mid
      order by r.mid) r on m.mid = r.mid
where director is not null
order by m.director,
         r.max_review desc
```



