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

  .. code:: perl

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

Me:
  *Thinking* maybe it would be useful to have something like:

  .. code:: perl

    my (@results) = $rs->search()->soft_where({ columnname => { '-in' => \@input }  })->all;

  Which would function like a big-assed glorious grep, but also with some scope to optimise/tweak
  the final query based on the software specification before executed.

  Maybe it could do multi-queries under the hood like I did, or maybe elide parameters from the query
  itself and just post-filter it, depending on which is more likely to be efficient.

  *thinking* Nah. Too many ways to fuck that up.


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

ResultSets by the book
----------------------

Somebody not me:

  https://metacpan.org/pod/DBIx::Class::ResultSet#ResultSet-subclassing-with-Moose-and-similar-constructor-providers

  .. code:: irclog

    <Getty> BUILDARGS did not return a hashref at (eval 742)[/home/torsten/usr/perl520/perl5/lib/perl5/Sub/Quote.pm:3] line 35.
    <Getty>         DBIx::Class::Getty::ResultSet::new(DBIx::Class::Getty::ResultSet=HASH(0x42b2a90), HASH(0x519d200)) called at t/load.t line 14


Me:

  ...

Not Me:

  .. code:: perl

    use Moo;

    extends 'DBIx::Class::ResultSet';

    sub BUILDARGS { $_[2] }

    __PACKAGE__->load_components(qw(
      Helper::ResultSet::Me
      Helper::ResultSet::OneRow
      Helper::ResultSet::Shortcut::Limit
      Helper::ResultSet::Shortcut::OrderBy
      Helper::ResultSet::Shortcut::Prefetch
      Helper::ResultSet::CorrelateRelationship
    ));

Perl:

  .. code::

    Expected parent constructor of DBIx::Class::Getty::Result to be DBIx::Class::Row, but found DBIx::Class::Helper::Row::StorageValues: changing the inheritance chain (@ISA) at runtime (after lib/DBIx/Class/Getty/Result.pm line 5) is unsupported at /home/torsten/usr/perl520/perl5/lib/perl5/Method/Generate/Constructor.pm line 89.

