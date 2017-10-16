search vs find
--------------

Me:
  https://metacpan.org/pod/DBIx::Class::ResultSet#search

  Huh, weird, it looks like it says in the caveats that search doesn't do
  anything special, only find and friends does.

  I Guess that's gonna be a problem if I use anything fancy on my RS,
  surely, triggers and shit converting input types is what I want?

Answer:
  No, Find is dog shit, search is better because its dumb.

  Find sucks because it basically goes through the constraint list
  until it finds one that matches, and uses constraints for matching

  It won't even *consider* things in the query side that aren't constraints, so its basically russian-roulette
  with your database. Something every database needs. Why not add arbitrary UTF8 changes while we're there.

Too many SQL variables
----------------------

Me:
  Gosh, I have thousands of things I need to query where the given field must match
  some other field, like, almost like a INNER JOIN, except, instead of being able to
  just <existing> INNER JOIN <input>, well, <input> ain't in the database. What do?
 
    { columnname => { "=" => [ .... ] } } # Computer says no.

    { columnname => { "-in" => [ ... ] } } # Computer say no.

Answer:
  DBMS-agnostically, you're kinda fucked there.

Me:
  This approach works though to significantly reduce the number of queries:

  .. code:: perl

    my (@input) = ...;

    my (%seen) = map { $_ => 0 } @input;

    while ( @input ) {

        # beyond 10 or so the returns diminsh too slowly
        my (@subset) = splice @input, 0, 20, ();

        my (@results) = $rs->search({ columnname => { '-in' => \@subset }  })->all;

        for my $item (@results) {
            $seen{ $item->columnname }++;
            $item->update( stuff );
        }
    }

Hotspots?
---------

In a query that did 19794 calls to Row->update

Lost 2 seconds of 22 on this line:

  https://metacpan.org/source/RIBASUSHI/DBIx-Class-0.082840/lib/DBIx/Class/ResultSet.pm#L1329


SQL-A::update eats lots:

    # spent 5.82s (824ms+5.00) within SQL::Abstract::update which was called 19794 times, avg 294µs/call:

    # 19794 times (824ms+5.00s) by DBIx::Class::Storage::DBI::_gen_sql_bind at line 1679 of DBIx/Class/Storage/DBI.pm, avg 294µs/call

Lost another 1 here:

  https://metacpan.org/source/ILMARI/SQL-Abstract-1.84/lib/SQL/Abstract.pm#L365

Half a second here:

  https://metacpan.org/source/ILMARI/SQL-Abstract-1.84/lib/SQL/Abstract.pm#L378

2 seconds here:

  https://metacpan.org/source/ILMARI/SQL-Abstract-1.84/lib/SQL/Abstract.pm#L426



