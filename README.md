# Rapid prototyping with postgres dependencies solved

#### The mental framework:
1. Anytime we define a new object, we store it's definition in a table in our database (this allows us to store `*` in our queries and propagate schema changes we make to dependencies).
2. When we want to update a schema/definition we: grab the definition from our table, drop cascade everything dependent on it, redefine it, resave the definition to our table, and restore _everything_ in the database. 
Restoring everything in the database might seem inelegant, but it's necessary.


I created a set of functions and table which I pushed to a repo for others to use. The functions are:
```
dep_store -- the table where all your view and function definitions are stored. Schema is (name text, definition text)
dep_commit(name, definition) -- save a definition
dep_restore() - restore everything saved (O(n^2))
dep_def(name) - quickly grab the definition of an object
dep_drop(name) - delete a definition
```


#### Installation:
Just copy the code in src.psql and run it in your database. Commit any existing views or function definition existing in your database with `dep_commit` applied to each of them. 



#### Here's the workflow I use:
1. `select * from dep_store order by name` - This shows me everything in my database in a glance and I can quickly see the beginnings of the query definitions for my objects.

2. `drop view myview cascade` - When I want to make a schema change to it

3. `create or replace view myview as <all the cool changes I want to make to the schema>` - Redefine the object

4. `select dep_commit('myview','create or replace view myview as...')` - We have to commit our changes to use them to restore in the future if the object is dropped via cascade

5. `select dep_restore()` Attempt to restore every object in the db. If we used * in our definitions, that will propagate through the newly restored objects. 

6. There may be an error at this step if a dependency needed a column you removed when making your change. In that case we look in the error message to see which object couldn't be restored and:

7. `select dep_def('brokenview')`

8. Copy the definition and paste it in the editor

9. Make edits needed

10. `select dep_commit(...)` - don't forget to commit all changes

11. `select dep_restore()`

Eventually we successfully restore everything!

When you want to drop something in your database, you just drop it and have to remember to call `dep_drop('name')` so it doesn't get restored.



* Note if you use strings in your queries and you need to commit them, you have to change ' to ''. One single quote becomes two single quotes. 
* If you already have views and functions in your db, you'll have to commit all of them to use this method.
* Up until now, I've used the supabase editor for creating tables.

**Again, anytime something is changed except tables in my `database,` I commit it with `dep_commit`.**

**I hope this helps people get started. I'm new to postgres too, so I'm definitely open to feedback from people with more experience!**


#### Star me
If I helped you, please consider giving me a star!
