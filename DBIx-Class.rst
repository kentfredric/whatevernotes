search vs find
--------------

Me
^^

https://metacpan.org/pod/DBIx::Class::ResultSet#search

Huh, weird, it looks like it says in the caveats that search doesn't do
anything special, only find and friends does.

I Guess that's gonna be a problem if I use anything fancy on my RS,
surely, triggers and shit converting input types is what I want?

Answer
^^^^^^

No, Find is dog shit, search is better because its dumb.

Find sucks because it basically goes through the constraint list
until it finds one that matches, and uses constraints for matching

It won't even *consider* things in the query side that aren't constraints, so its basically russian-roulette
with your database. Something every database needs. Why not add arbitrary UTF8 changes while we're there.

Too many SQL variables
----------------------

Me
^^

Gosh, I have thousands of things I need to query where the given field must match
some other field, like, almost like a INNER JOIN, except, instead of being able to
just <existing> INNER JOIN <input>, well, <input> ain't in the database. What do?

code::
  { columnname => { "=" => [ .... ] } } # Computer says no.
  { columnname => { "-in" => [ ... ] } } # Computer say no.

Answer
^^^^^^

DBMS-agnostically, you're kinda fucked there.

