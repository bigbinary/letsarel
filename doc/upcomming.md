# Left Outer Join ActiveRecord

includes on scope eager loads the given associations

````
Person.by_name("brad").includes(:movies)
SELECT "people".* FROM "people" 
WHERE (("people"."first_name" LIKE '%brad%' OR "people"."last_name" LIKE '%brad%'))
SELECT "collaborations".* FROM "collaborations" 
WHERE "collaborations"."person_id" IN (289834744)
SELECT "movies".* FROM "movies" WHERE "movies"."id" IN (552238307)
````

but if any of the eager loaded association is used in the query join
used in the query changed from inner join to left outer join

````
Person.by_name("brad").includes(:movies).merge(Movie.drama)
SELECT "people"."id" AS t0_r0, "people"."first_name" AS t0_r1, 
"people"."last_name" AS t0_r2, "people"."created_at" AS t0_r3, 
"people"."updated_at" AS t0_r4, "movies"."id" AS t1_r0, 
"movies"."title" AS t1_r1, "movies"."budget" AS t1_r2, 
"movies"."revenue" AS t1_r3, "movies"."released_on" AS t1_r4, 
"movies"."genre" AS t1_r5, "movies"."distributor_id" AS t1_r6, 
"movies"."created_at" AS t1_r7, "movies"."updated_at" AS t1_r8 
FROM "people" LEFT OUTER JOIN 
"collaborations" ON "collaborations"."person_id" = "people"."id" 
LEFT OUTER JOIN "movies" ON "movies"."id" = "collaborations"."movie_id" 
WHERE "movies"."genre" = 'drama' 
AND (("people"."first_name" LIKE '%brad%' OR "people"."last_name" LIKE '%brad%'))
````

if you use order on the including association it will trigger LEFT OUTER
JOIN

````
Person.includes(:movies).order("movies.title")
or 
Person.includes(:movies).order(Movie.arel_table[:title])
````
result below sql
````
SELECT "people"."id" AS t0_r0, "people"."first_name" AS t0_r1, "people"."last_name" AS t0_r2,
"people"."created_at" AS t0_r3, "people"."updated_at" AS t0_r4, "movies"."id" AS t1_r0, 
"movies"."title" AS t1_r1, "movies"."budget" AS t1_r2, "movies"."revenue" AS t1_r3, 
"movies"."released_on" AS t1_r4, "movies"."genre" AS t1_r5, "movies"."distributor_id" AS t1_r6,
"movies"."created_at" AS t1_r7, "movies"."updated_at" AS t1_r8 FROM "people" 
LEFT OUTER JOIN "collaborations" ON "collaborations"."person_id" = "people"."id" 
LEFT OUTER JOIN "movies" ON "movies"."id" = "collaborations"."movie_id" 
ORDER BY movies.title
````
but query will fail if order query element does not have a table prefix

````
Person.includes(:movies).order("title")
SELECT "people".* FROM "people" ORDER BY title
SQLite3::SQLException: no such column: title: SELECT "people".* FROM "people"  ORDER BY title
````

### selecting from different tables

imagine i need select two columns from different tables

````
Person.joins(collaborations: :movie)
````

I need to select collaboration role and movie title from the above scope

````
Person.joins(collaborations: :movie).select("collaborations.role,
movies.title")
````

If you want to this in arel 

````
proxy = Person.joins(collaborations: :movie)
proxy.arel.ast.cores.first.projections = [ Collaboration.arel_table[:role], Movie.arel_table[:title] ]
proxy.to_sql

"SELECT "collaborations"."role", "movies"."title" FROM "people" 
INNER JOIN "collaborations" ON "collaborations"."person_id" = "people"."id" 
INNER JOIN "movies" ON "movies"."id" = "collaborations"."movie_id""
````
### order using arel attribute

if you want to specify order asc/desc in your query

````
Person.order(:first_name)
SELECT "people".* FROM "people" ORDER BY first_name
````

you want to desc on first_name
````
Person.order("first_name desc")
SELECT "people".* FROM "people" ORDER BY first_name desc
````
you can do this same using arel attribute

````
Person.order(Person.arel_table[:first_name].desc)
SELECT "people".* FROM "people" ORDER BY "people"."first_name" DESC
````
if you use arel attribute then constructed query has the table prefix as
opposed to first case(Person.order(:first_name))
