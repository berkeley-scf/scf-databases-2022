Solutions to challenges in the SCF databases tutorial.

Note that many of the answers below return the full result of the query which may be much more data than you want to transfer into an R dataframe (and the transfer itself might take the bulk of the time used).

The answers are posed in terms of a query from R, but the SQL syntax can used without modification in Python.

## Answers to challenges in the SQL page:

## 1.1 Getting started

***Challenge***: Return a few rows from the users, questions, answers, and tags tables so you can get a sense for what the entries in the tables are like.

```
dbGetQuery(db, "select * from users limit 5")
dbGetQuery(db, "select * from questions limit 5")
```

***Challenge***: Find the oldest users in the database.

```
## this is a computationally expensive approach because it sorts the entire dataset
dbGetQuery(db, "select * from users sort by age desc limit 10")
## this is quicker but might involve some trial and error
dbGetQuery(db, "select * from users where age > 80")
```

## 1.3 Grouping / stratifying


***Challenge***: What does this query do? Describe the table that would be returned.
```
result <- dbGetQuery(db, "select tag, count(*) as n from questions_tags group by tag order by n desc limit 100")
```
This groups by the unique tags and then counts the number of times each tag occurs is used in a question, returning the 100 most-used tags. The `count(*)` counts the number of rows within each group (i.e., for each tag). The resulting table should have two fields: `tag` and `n` and 100 rows. Without `limit 100` it would have as many rows as the number of unique tags.

***Challenge***: Write a query that will count the number of answers for each question, returning the IDs of the most answered questions.

```
result <- dbGetQuery(db, "select questionid, count(*) as n
               from answers
               group by questionid order by n desc limit 5")
```

### 1.4.1 Simple joins

***Challenge***: What does this three-way join do? 
```
result1 <- dbGetQuery(db, "select * from questions Q
        join questions_tags T on Q.questionid = T.questionid
        join users U on Q.ownerid = U.userid
        where tag = 'python' and age > 70")
```

This matches tags and user information to all the questions, selecting only questions about Python asked by users over 70 years old.


***Challenge***: Write a query that would return all the answers to questions with the Python tag.

```
result <- dbGetQuery(db, "select * from answers A join questions_tags T
               on A.questionid = T.questionid
               where T.tag = 'python'")
```

***Challenge***: Write a query that will count the number of answers for each question, returning information about the  most answered questions.

```
result <- dbGetQuery(db, "select questions.questionid, questions.creationdate,
               questions.ownerid, count(*) as n
               from questions join answers on
               questions.questionid = answers.questionid
               group by questions.questionid order by n desc limit 5")
```

***Challenge***: Write a query that would return the users who have answered a question with the Python tag.

```
result <- dbGetQuery(db, "select * from answers A join users U
               on A.ownerid = U.userid
               join questions_tags T on A.questionid = T.questionid
               where T.tag = 'python'")
```

That would return the answers as well, and will have duplicates in terms of the user information, so this next query more exactly answers the question using the DISTINCT keyword (DISTINCT).

```
result <- dbGetQuery(db, "select distinct U.userid, U.location, U.displayname,
               U.age, U.upvotes, U.downvotes
               from users U join answers A
               on A.ownerid = U.userid
               join questions_tags T on A.questionid = T.questionid
               where T.tag = 'python'")
```


### 1.4.2 More on joins

***Challenge***: Create a view with one row for every question-tag pair, including questions without any tags.

```
result <- dbGetQuery(db, "create view tagged_questions as
               select *
               from questions Q left outer join questions_tags T
               on Q.questionid = T.questionid")

***Challenge***: Write a query that would return the displaynames of all of the users who have *never* posted a question. The NULL keyword will come in handy -- it's like `NA` in R. Hint: NULLs should be produced if you do an outer join.

```
result <- dbGetQuery(db, "select distinct displayname 
               from questions Q right outer join users U
               on Q.ownerid = U.userid       
               where Q.ownerid is NULL")

## no right outer join in SQLite
result <- dbGetQuery(db, "select distinct displayname 
               from users U left outer join questions Q
               on Q.ownerid = U.userid       
               where Q.ownerid is NULL")

```

***Challenge***: How many questions tagged with 'random-forest' were unanswered? (You should need two different kinds of joins to answer this.)

```
result <- dbGetQuery(db, "select count(*) from questions Q join questions_tags T
               on Q.questionid = T.questionid
               left outer join answers A on Q.questionid = A.questionid
               where T.tag = 'random-forest' and
               A.answerid is NULL")
```


### 1.4.3 Joining a table with itself (self join)

***Challenge***: What is the problem with using this query to get all pairs of questions asked by the same user?

```
result <- dbGetQuery(db, "create view question_contrasts as
               select * from questions Q1 join questions Q2
               on Q1.ownerid = Q2.ownerid")
```

This will return each question paired with itself as well as the pairings we want.

***Challenge***: There's actually a further similar problem. What is the problem and how can we fix it by changing two characters in the query above? Hint, even as character strings, the creationdate column has an ordering.

Furthermore, it will return two copies of each pair of questions, one with one question listed first and the other with the other question listed first.

We can fix the latter problem with:

```
result <- dbGetQuery(db, "create view question_contrasts as
               select * from questions Q1 join questions Q2
               on Q1.ownerid = Q2.ownerid
               where Q1.creationdate < Q2.creationdate")
```

## 2.2 String processing and creating new fields

***Challenge***: Select the questions that have "java" but not "javascript" in their titles using regular expression syntax.

```{r}
dbGetQuery(db, "select * from questions_tags where tag SIMILAR TO '%java[^s]%' limit 10")
dbGetQuery(db, "select * from questions_tags where tag SIMILAR TO '%java%'
               except
               select * from questions_tags where tag SIMILAR to '%javascript%'
               limit 10")
```


***Challenge***: Figure out how to calculate the length (in characters) of the title of each question.

```
dbGetQuery(db, "select title, length(title) as nchar from questions limit 5")
```

***Challenge***: Process the creationdate field to create year, day, and month fields.


```
dbExecute(db, "create view questions_dated as
               select questionid, ownerid, score, viewcount, 
               substring(creationdate from '#\"[[:digit:]]{4}#\"%' for '#') as year,
               substring(creationdate from '[[:digit:]]{4}-#\"[[:digit:]]{2}#\"%' for '#') as month,
               substring(creationdate from '[[:digit:]]{4}-[[:digit:]]{2}-#\"[[:digit:]]{2}#\"%' for '#') as day 
               from questions")
               
```

That used regular expressions for illustration but could be done with fixed length substrings, which means one could it with SUBSTR in SQLite.

```
dbExecute(db, "create view questions_dated as
               select questionid, ownerid, score, viewcount, 
               substr(creationdate, 1, 4) as year,
               substr(creationdate, 6, 2) as month,
               substr(creationdate, 9, 2) as day
               from questions")
```

## 3.1 Set operations: UNION, INTERSECT, EXCEPT

***Challenge***: Those two queries return equivalent information, but the results are not exactly the same. What causes the difference? How can we modify the second query to get the exact same results as the first?

Note that the second query will return duplicates where we have a person asking multiple R or Python queries. But we know how to solve that by including DISTINCT in the second query:

```
select distinct displayname, userid from ...
``` 

Even after we do that, the results from the two queries will probably not be in the same order, so one would need to do some sorting.

***Challenge***: Find the users who have asked an R question or a Python question.

```
result <- dbGetQuery(db, "select displayname, userid from
               questions Q join users U on U.userid = Q.ownerid
               join questions_tags T on Q.questionid = T.questionid
               where tag = 'r'
               union
               select displayname, userid from
               questions Q join users U on U.userid = Q.ownerid
               join questions_tags T on Q.questionid = T.questionid
               where tag = 'python'")
```

***Challenge***: Find the users who have asked only an R question and not a Python question.

```
result <- dbGetQuery(db, "select displayname, userid from
               questions Q join users U on U.userid = Q.ownerid
               join questions_tags T on Q.questionid = T.questionid
               where tag = 'r'
               except
               select displayname, userid from
               questions Q join users U on U.userid = Q.ownerid
               join questions_tags T on Q.questionid = T.questionid
               where tag = 'python'")
```


## 3.2 Subqueries

### 3.2.1 Subqueries in the FROM statement

***Challenge***: What does the following do?

```{r, eval=FALSE}
dbGetQuery(db, "select * from questions join answers A on questions.questionid = A.questionid 
                join 
                (select ownerid, count(*) as n_answered from answers 
	            group by ownerid order by n_answered desc limit 1000) most_responsive on
                A.ownerid = most_responsive.userid")
```

It finds all the answers (and the questions they go with) from users who are the top 1000 users in terms of the number of questions they answer.

***Challenge***: Write a query that, for each question, will return the question title, number of answers, and the answer to that question written by the user with the highest reputation.

```
result <- dbGetQuery(db, "select * from questions Q join
                          (select *, max(reputation) as maxRep, count(*) as n
                          from answers A join users U
                          on A.ownerid = U.userid group by A.questionid) maxRepAnswers
                          on Q.questionid = maxRepAnswers.questionid limit 5")
```

### 3.2.2 Subqueries in the WHERE statement

***Challenge***: Write a query that would return the users who have answered a question with the Python tag. We've seen this challenge before, but do it now based on a subquery.

```
result <- dbGetQuery(db, "select displayname, userid from users where userid in
                          (select distinct ownerid from
                          questions join questions_tags on questions.questionid = questions_tags.questionid
                          where tag = 'python')")
```

***Challenge***: How would you find all the answers associated with the user with the most upvotes?

```
result <- dbGetQuery(db, "select * from answers join users on answers.ownerid = users.userid
                          where upvotes =
                          (select max(upvotes) from users)")
```

***Challenge***: Create a frequency list of the tags used in the top 100 most answered questions. This is a hard one - try to work it out in pieces, starting with the query that finds the most answered questions and joining that to other tables as needed. Note there is a way to do this with a JOIN and a way without a JOIN.

Here's one solution that joins a table to the result of a subquery.

```
result1 <- dbGetQuery(db, "select T.tag, count(*) as tagCnt from
               questions_tags T 
               join 
               (select questionid from answers
               group by questionid order by count(*) desc limit 100) most_answered
               on T.questionid = most_answered.questionid
               group by T.tag order by tagCnt desc")
```

Here's another solution that uses the subquery in a WHERE:

```
result2 <- dbGetQuery(db, "select T.tag, count(*) as tagCnt from
               questions_tags T where questionid in
               (select questionid from answers
               group by questionid order by count(*) desc limit 100)
               group by T.tag order by tagCnt desc")
```


## 3.3 Window functions

***Challenge***: Use a window function to compute the average viewcount for each ownerid for the 10 questions preceding each question. 


```
result <- dbGetQuery(db, "select *,
                          avg(viewcount) over (partition by ownerid order by creationdate
                          rows between 10 preceding and 1 preceding) as avg_view
                           
                          from questions limit 50")
```

***Challenge (hard)***: Find the users who have asked one question that is highly-viewed (viewcount > 1000) with their remaining questions not highly-viewed (viewcount < 20).

As a first step, this finds the viewcount, rank of each question and number of questions asked by the user

```{r}
result <- dbGetQuery(db, "select *,
                          rank() over w as rank,
                          max(viewcount) over w as maxcount
                          from questions where ownerid is not null
                          window w as (partition by ownerid order by viewcount desc)")
```

Now let's use that result as a subquery where we filter based on the conditions we need to satisfy:

```{r}
result <- dbGetQuery(db, "select * from
                          (select *,
                          rank() over w as rank,
                          max(viewcount) over w as maxcount
                          from questions where ownerid is not null
                          window w as (partition by ownerid order by viewcount desc))
                          where rank = 2 and viewcount < 100 and maxcount > 1000")
```                         


## 3.4 Putting it all together to do complicated queries

Rough sketch pseudo code solutions.

1) Given a table of user sessions with the format
```
date | session_id | user_id | session_time
```
calculate the distribution of the average daily total session time in the last month. I.e., you want to get each user's daily average and then find the distribution over users. The output should be something like:

```
minutes_per_day | number_of_users
```

Pseudo-code answer:

a) filter to last month (use datetime functionality to extract month or by comparing to the day that was one month ago)
b) sum session_time grouped by user
c) divide by number of days in month and round
d) count grouped by minutes_per_day 


2) Consider a table of messages of the form
```
sender_id | receiver_id | message_id
```
For each user, find the three users they message the most.

Pseudo-code answer:

a) count grouped by sender and receiver
b) window function to rank for each sender in descending order on the count
c) filter to rank <= 3


3) Suppose you have are running an online experiment and have a table on
the experimental design
```
user_id | test_group | date_first_exposed
```
Suppose you also have a messages table that indicates if each message
was sent on web or mobile:
```
date | sender_id | receive_id | message_id | interface (web or mobile)
```
What is the average (over users) of the average number of messages sent per day for each test group
if you look at the users who have sent messages only on mobile in the last month.

Pseudo-code answer:

a) filter to messages in last month
a) subquery to get mobile-only users - find all mobile users and use EXCEPT to remove all web users
b) join design, messages tables and count grouped by users, filtering to mobile users
c) divide by number of days in month
d) average grouped by test_group

(Note the statistical issue of selection bias in who decides to only send mobile messages or send messages at all...)
