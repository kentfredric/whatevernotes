search vs find
--------------

- Me: ::

  https://metacpan.org/pod/DBIx::Class::ResultSet#search

  Huh, weird, it looks like it says in the caveats that search doesn't do
  anything special, only find and friends does.

  I Guess that's gonna be a problem if I use anything fancy on my RS,
  surely, triggers and shit converting input types is what I want?

- Answer ::

  No, Find is dog shit, search is better because its dumb.

  Find sucks because it basically goes through the constraint list
  until it finds one that matches, and uses constraints for matching

