---
layout: post
author: Paul Fitzpatrick
title: Diffing and patching data
username: paulfitz
---

A few years ago at the Eastern Conference for
Workplace Democracy in New Hampshire, a bunch of friends
chatting on a grassy knoll 
realized they were all working on overlapping directories of their 
communities, and decided to pool their efforts.
They tracked down some techies (I was one) and set them
to work building a directory website.  Someone should have warned
them about techies.

![From http://xkcd.com/974](http://imgs.xkcd.com/comics/the_general_problem.png "From http://xkcd.com/974")

Eight years later, okay there is a [directory website](http://find.coop), but the project has
morphed into something a lot more ambitious:

 * A full-blown co-op to deal with the cultural and legal side of data sharing.  This is the [Data Commons Co-op](http://datacommons.find.coop).
 * A growing toolbox to deal with the technological side of data sharing, specifically how to have fun (rather than get depressed) collaborating on data projects.  This is the [Coopy Toolbox](https://github.com/paulfitz/coopy).

We, like others in the Open Data world, have been asking: where's the
github for data?  More fundamentally, where's the `diff` and `patch`
programs for data?  Where's something like `diff3` for doing 3-way
merges?  Can we bring the whole free and open toolchain of diffing,
patching, merging, and version control to the world of data?

## The toolchain is there already, for some

Fun data collaboration is possible today by borrowing
existing tools designed with source code in mind, as
Rufus Pollock [has noted](http://blog.okfn.org/2013/07/02/git-and-github-for-data/).
Here's an example
of a pull request found in the wild, made to a repository on github
that tracks some bus routes in Iceland in regular CSV files:

> <https://github.com/gudmundur/straeto-data/pull/4>

> "Fixed stops on the wrong side"

![patching a bus schedule](/img/coopy-bus-stop.png)

I'm a bit embarrassed to remember how excited I was to
stumble across a
pull request to a bus schedule.  I ran around showing people, saying
"this!" "this!"   It is one of those moments when
you realize: you really are a nerd.  It is hard to explain to a
non-programmer why the social dynamics of this are going to be
so much better than a spreadsheet in google docs.

There are some technical drawbacks to working this way today though.  For example: what happens when things go wrong?  A poor merge
or a merge conflict in a text file still results in a text file, which
a user can edit as usual.  Text file in, text file out. But a poor
merge or a conflict in a CSV file can leave you with an invalid CSV
file with missing/surplus columns on some rows, or with conflict
markers inserted.  This bumps the user out of
Gnumeric/LibreOffice/Excel/Sqlite/... or whatever they were using to edit the
table, and leaves them staring at what may well be gobbledygook to
them. Another problem: column changes look awful in line-oriented diffs, which isn't
a deal-breaker but which is certainly a pity. We can do better.

## A diff for (tabular) data

There didn't seem to be any neutral format for comparing tables out
there when I went looking.  SQL could be abused to serve (DELETE
clauses to express rows removed, INSERTs rows added, UPDATEs for
modified cells, etc) but I couldn't see that leading anywhere happy.
My first idea was to express diffs as tables in CSV form, as a list
of operations.  [This was ugly](http://share.find.coop/doc/patch_format_csv_v_0_2.html).  Then Joe Panico of <http://www.diffkit.org> and I hammered out something we 
called [`tdiff`](http://share.find.coop/doc/patch_format_tdiff.html), 
tabular diff format, very much inspired by classic diffs, with added
awareness of columns.  This was better, but still felt a bit clunky. 
Finally, I arrived at what now seems obvious: 
a "[highlighter](http://share.find.coop/doc/spec_hilite.html)" format
that is just the original table with stylized editing marks to show
changes (and large chunks of unchanged material removed).

The highlighter format is tabular, and designed to be as simple as I
could make it without introducing ambiguity.  The first column in a
highlighter diff is called the "action" column, containing marks
meaning "inserted row", "deleted row", "modified row", etc.  Remaining
columns are drawn from either or both of the tables being compared.
If there are column differences to note, an extra row called the
"schema row" is inserted, which has marks for the inserted, deleted, 
or otherwise modified columns.
The whole diff can be transmitted
safely in CSV, then optionally formatted for prettiness using some
mechanical rules.

For example, here is an imagined Coopy highlighter diff against Jessica Lord's
[Hack Spots](http://jlord.github.io/hack-spots/)
[spreadsheet](https://docs.google.com/spreadsheet/ccc?key=0Ao5u1U6KYND7dFVkcnJRNUtHWUNKamxoRGg4ZzNiT3c#gid=0) at the time of writing (Hack Spots is a list
of hacking-friendly coffee shops demoing [sheetsee.js](http://jlord.github.io/sheetsee.js/)):

<table style="border-collapse:collapse">
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">@@</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Contributer's Twitter</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Name</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Address</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">City</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">State</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">long</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">lat</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Country</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Wifi Password</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Outlets</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Couch</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Large Table</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Brewing</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Outdoor Seating</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">hexcolor</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">cwmma</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Block 11</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">11 Bow St</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Somerville</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">MA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-71.096974</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">42.380881</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Intelligentsia</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#7f7fff" bgcolor="#7f7fff">&#8594;</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">thomaslevine</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7f7fff" bgcolor="#7f7fff">Bordelands Cafe&#8594;Borderlands Caf&#233;</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">870 Valencia St</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">San Francisco</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">CA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-122.42151</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">37.759031</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">open</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">?</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">?</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">coffee</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">lukekarrys</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Cartel Coffee Lab</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">225 W University Dr</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Tempe</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">AZ</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-111.942978</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">33.421907</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">espresso</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">In-house</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">thomaslevine</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">El Beit</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">158 Bedford Ave</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Brooklyn</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">NY</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-73.956847</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">40.718529</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">brooklyn</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">few</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">?</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">---</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">hij1nx</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Five Elephant</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Reichenberger Stra&#223;e 101, 10999</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Berlin</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Berlin</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">13.43829</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">52.493365</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">DE</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f; color:#888" bgcolor="#ff7f7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">5 Elephant</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">uhduh</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Gangplank</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">260 S Arizona Ave</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Chandler</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">AZ</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-111.841302</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">33.244008</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">walktheplank</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">sfrdmn</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Noisebridge</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">2169 Mission St</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">San Francisco</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">CA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-122.419161</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">37.762372</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Open</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">BYOC</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">possibly</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">+++</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">fitzyfitzyfitzy</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">Sandwich Theory</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">590 Valley Rd</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f"> Montclair</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f"> NJ</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">-74.208086</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">40.840497</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">sandwich</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">coffee</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">#B9FCFC</td>
</tr>
</table>

This was generated by taking the Hack Spot spreadsheet, editing it
in `gnumeric`, then comparing with the original using [coopyhx](https://npmjs.org/package/coopyhx).
I corrected a typo, added an entry for Sandwich Theory in my
neighborhood, and - completely accidentally - deleted the 
entry for Five Elephant.  For completeness, this would be the diff
in CSV format:

     @@,"Contributer's Twitter",Name,Address,City,State,long,lat,Country,"Wifi Password",Outlets,Couch,"Large Table",Brewing,"Outdoor Seating",hexcolor
     ...,...,...,...,...,...,...,...,...,...,...,...,...,...,...,...
     NULL,cwmma,"Block 11","11 Bow St",Somerville,MA,-71.096974,42.380881,USA,NULL,yes,yes,NULL,Intelligentsia,NULL,#B9FCFC
     ->,thomaslevine,"Bordelands Cafe->Borderlands Café","870 Valencia St","San Francisco",CA,-122.42151,37.759031,USA,open,yes,?,?,coffee,no,#B9FCFC
     NULL,lukekarrys,"Cartel Coffee Lab","225 W University Dr",Tempe,AZ,-111.942978,33.421907,USA,espresso,yes,yes,no,In-house,no,#B9FCFC
     NULL,thomaslevine,"El Beit","158 Bedford Ave",Brooklyn,NY,-73.956847,40.718529,USA,brooklyn,few,no,no,?,yes,#B9FCFC
     ---,hij1nx,"Five Elephant","Reichenberger Straße 101, 10999",Berlin,Berlin,13.43829,52.493365,DE,NULL,yes,no,no,"5 Elephant",yes,#B9FCFC
     NULL,uhduh,Gangplank,"260 S Arizona Ave",Chandler,AZ,-111.841302,33.244008,USA,walktheplank,yes,yes,yes,NULL,no,#B9FCFC
     ...,...,...,...,...,...,...,...,...,...,...,...,...,...,...,...
     NULL,sfrdmn,Noisebridge,"2169 Mission St","San Francisco",CA,-122.419161,37.762372,USA,Open,yes,yes,yes,BYOC,possibly,#B9FCFC
     +++,fitzyfitzyfitzy,"Sandwich Theory","590 Valley Rd"," Montclair"," NJ",-74.208086,40.840497,USA,sandwich,yes,no,coffee,no,yes,#B9FCFC

Rather than editing the Hack Spots spreadsheet directly in google 
docs, in my ideal world I'd send a pull-request (and someone would catch
my Five Elephant goof).

So far, the diff we have is just row-based.
Suppose I also added a column for the location's website, and deleted
the password column (rather missing the whole point) - the diff would now look like this:

<table style="border-collapse:collapse">
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#aaa" bgcolor="#aaa">!</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa" bgcolor="#aaa">+++</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa" bgcolor="#aaa">---</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaa; color:#888" bgcolor="#aaa"></td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">@@</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Contributer's Twitter</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Name</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Website</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Address</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">City</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">State</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">long</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">lat</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Country</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Wifi Password</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Outlets</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Couch</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Large Table</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Brewing</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Outdoor Seating</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">hexcolor</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">cwmma</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Block 11</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f; color:#888" bgcolor="#7fff7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">11 Bow St</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Somerville</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">MA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-71.096974</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">42.380881</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f; color:#888" bgcolor="#ff7f7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Intelligentsia</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#7f7fff" bgcolor="#7f7fff">&#8594;</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">thomaslevine</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7f7fff" bgcolor="#7f7fff">Bordelands Cafe&#8594;Borderlands Caf&#233;</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f; color:#888" bgcolor="#7fff7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">870 Valencia St</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">San Francisco</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">CA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-122.42151</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">37.759031</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">open</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">?</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">?</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">coffee</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">lukekarrys</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Cartel Coffee Lab</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f; color:#888" bgcolor="#7fff7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">225 W University Dr</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Tempe</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">AZ</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-111.942978</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">33.421907</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">espresso</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">In-house</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">thomaslevine</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">El Beit</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f; color:#888" bgcolor="#7fff7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">158 Bedford Ave</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Brooklyn</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">NY</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-73.956847</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">40.718529</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">brooklyn</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">few</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">?</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">---</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">hij1nx</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Five Elephant</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f; color:#888" bgcolor="#ff7f7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Reichenberger Stra&#223;e 101, 10999</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Berlin</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Berlin</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">13.43829</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">52.493365</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">DE</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f; color:#888" bgcolor="#ff7f7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">5 Elephant</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">uhduh</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Gangplank</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f; color:#888" bgcolor="#7fff7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">260 S Arizona Ave</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Chandler</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">AZ</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-111.841302</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">33.244008</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">walktheplank</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">sfrdmn</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Noisebridge</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f; color:#888" bgcolor="#7fff7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">2169 Mission St</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">San Francisco</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">CA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-122.419161</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">37.762372</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f" bgcolor="#ff7f7f">Open</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">BYOC</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">possibly</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">+++</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">fitzyfitzyfitzy</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">Sandwich Theory</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">www.sandwichtheory.com</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">590 Valley Rd</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">Montclair</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">NJ</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">-74.208086</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">40.840497</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#ff7f7f; color:#888" bgcolor="#ff7f7f"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">coffee</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#7fff7f" bgcolor="#7fff7f">#B9FCFC</td>
</tr>
</table>

Note the extra row inserted at the very beginning to identify
column operations.

There are plenty of other details, but that is the basic flavor of the
Coopy highlighter diff format today.  You can [read more about
it](http://share.find.coop/doc/spec_hilite.html) or 
try out two different implementations live, a
[Javascript implementation](http://paulfitz.github.io/coopyhx/)
and a
[C++ implementation](http://share.find.coop/).
You can also get a feel for using this kind of diff in a workflow 
at [GrowRows.com](http://growrows.com/).  Please send bug reports, or
ideas for better alternatives!

## Dealing with conflict

What happens if two people make conflicting changes to a table?
A regular text-based merge would stick in >>>>>>> ======== <<<<<<<
blocks, which would destroy our table's structure.  I've played
with a few ways to do better.  The method I'm happiest with
so far is to report on conflicts in an extension of the 
highlighter diff format that shows the alternate updates
possible.  Imagine if, as I fixed the spelling of "Bordelands Cafe"
to "Borderlands Caf&#233;",
someone else had already corrected it to
the slightly different "Borderlands Cafe".  Now rather than seeing:

 `->,thomaslevine,"Bordelands Cafe->Borderlands Café",...`

In the diff, I'll see:

    -!->,thomaslevine,"Bordelands Cafe-!->Borderlands Cafe;-!->Borderlands Café",...

Or with formatting rules:

<table style="border-collapse:collapse">
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">@@</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Contributer's Twitter</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Name</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Address</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">City</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">State</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">long</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">lat</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Country</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Wifi Password</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Outlets</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Couch</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Large Table</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Brewing</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">Outdoor Seating</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#aaf; font-weight:bold; padding-bottom:4px; padding-top:5px; text-align:left" bgcolor="#aaf" align="left">hexcolor</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">cwmma</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Block 11</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">11 Bow St</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Somerville</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">MA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-71.096974</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">42.380881</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Intelligentsia</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; background-color:#f00" bgcolor="#f00">&#8594;</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">thomaslevine</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show; background-color:#f00" bgcolor="#f00">Bordelands Cafe&#8594;Borderlands Cafe&#8594;Borderlands Caf&#233;</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">870 Valencia St</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">San Francisco</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">CA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-122.42151</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">37.759031</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">open</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">?</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">?</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">coffee</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show; color:#888"></td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">lukekarrys</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Cartel Coffee Lab</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">225 W University Dr</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">Tempe</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">AZ</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">-111.942978</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">33.421907</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">USA</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">espresso</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">yes</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">In-house</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">no</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">#B9FCFC</td>
</tr>
<tr style=":first-child td{border-top:1px solid #2D4068}">
<td style="border:1px solid #2D4068; padding:3px 7px 2px; border-left:1px solid #2D4068; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
<td style="border:1px solid #2D4068; padding:3px 7px 2px; empty-cells:show">...</td>
</tr>
</table>

 You can get a sense for how this works by testing on
<http://paulfitz.github.io/coopyhx/>. Be sure to select 
"Use 3-way comparison" option, which will set up two
versions of a table with a shared "common ancestor".
Double-click on cells in the diff to view their plain "CSV"
representation, and edit them.

## Full-blown revision control

I've tried two methods to build revision control with all this:

 * Modifying [`fossil`](http://www.fossil-scm.org), a distributed revision
   control system with beautifully compact and hackable source code,
   to use tabular diffs and merges natively.  The result is `ssfossil`
   ("spreadsheet fossil") in the Coopy toolbox.
 * Using custom diff and merge drivers with `git`, to achieve a similar 
   result.  A tutorial for doing this is at <http://share.find.coop/doc/tutorial_git.html>.

Both approaches share the same features:

 * No change to how the SCM stores data internally.  For example,
   `fossil` will continue using 
   its [delta encoding](http://www.fossil-scm.org/xfer/doc/trunk/www/delta_format.wiki), likewise `git` (yes, yes, in pack files only).
 * Visualization of diffs changes, and how merges happen.  This is good, since changes that would conflict in text-file world may well *not* conflict in tabular world, and we are guaranteed to always have valid tables.

Until making more radical changes to the SCM, it definitely makes
sense to store tables in a text format.  Formats I've experimented
with are:

 * CSV.  Simple, globally understood.  But just a table.
 * CSVS.  I made this up.  CSV extension with multiple tables,
   unambiguous column rows, and table names.  [Looks like this](https://github.com/paulfitz/coopy/blob/master/tests/fold/contacts.csvs).
 * Sqlitext, read "Sqlite Text".  I made this up.  This is a text dump
   of an Sqlite database, with consistent ordering of rows.  With
   careful use of `git`'s `clean` and `smudge` filters, a "live" Sqlite 
   database can be kept in version control, using this format as
   an intermediate.  This has the nice property of storing more
   meta-data (keys, references, etc).
 * SocialCalc.  A text format for representing spreadsheets used 
   by [SocialCalc](https://github.com/DanBricklin/socialcalc) and 
   inherited by <ethercalc.org>.  Stores table formatting, unlike
   the other formats I tried.

Personally, Sqlitext is working well for my needs.  However,
conspicuously absent from this list are the formats users typically
store their data in, for example Excel/LibreOffice/Gnumeric formats.
We need one more trick to deal with them.

## The last mile

Complicated spreadsheets are not great candidates for version control
as I've imagined it so far, since we don't have a way to diff/merge
non-data features.  So arbitrary spreadsheets in Gnumeric,
LibreOffice, and other programs (for simplicity I'm going
to call all these programs "Excel" from now on, forgive me)
aren't really in our scope.  But simpler ones, just storing data
without anything fancy, would be.  And they are certainly 
convenient, familiar editors for tables.

Putting an Excel file in a git/fossil repository won't lead anywhere good.
But what we can do is this:

 * We use git/fossil/... to do version control on data in a
   version-control-friendly format.
 * We keep that data in sync with an Excel (or Access, or other) file
   using merges that preserve formatting in the Excel file.  The Excel
   file should never be regenerated from scratch (except parhaps once,
   on initial cloning), but patched in a format-preserving way.

In principle a modified SCM could collapse these steps but I'm
definitely not there yet.  So what I've used is a program called
(confusingly enough) `Coopy` that handles the end-to-end work
of versioning Excel (and similar) files.  Here is Coopy cloning
a repository with a single table in it called "numbers" (the user
needed a URL for the repository in order to do this):

![x](/img/coopy-clone.png)

I won't win any awards for UI design, I know.  At this point,
under the hood, the repository is checked out on the user's machine,
with data in a neutral format.  The list of tables is shown, in this
example just a table called "numbers". When the user selects that 
table for the very first time, they are prompted to save it:

![x](/img/coopy-save-table.png)

They can choose the format to save the data, for example in
an Excel-compatible format.  The appropriate conversion 
happens, and the file opens in an appropriate editor (`gnumeric` for
me):

![x](/img/coopy-save-xls.png)

We can now go ahead and edit the table at will.  When we're ready,
in Coopy, we click "push out".  We'll be prompted for a commit
message describing the changes, and (the first time) where to actually
push to:

![x](/img/coopy-commit.png)

From then on, "pulling in" and "pushing out" will act as if they are
operating on the spreadsheet, with local formatting being preserved
even if no format information is in fact being stored in the neutral
repository format.  It is perhaps hard to see why that is important,
but imagine how annoying it would be if, for example, the column sizes
of a spreadsheet kept getting reset everytime you pulled in a
collaborator's changes.

There's a lot more to say, but a key point is that we could now have
one person editing a table in Excel, another in Gnumeric, another
tweaking it using Sqlite, and the whole thing being periodically
sync'd to a MySQL database on a webserver.  Fun!

For more details:

 * <http://share.find.coop/doc/CoopyGuide.pdf>, a manual for several tools
   in the Coopy toolbox.

## The power of patching

Stepping back from full-on revision control, I'd like to mention
something nice that popped out of this that I hadn't anticipated.
Once you have diff + patch, you can play games like this:

 * Store data in some form optimized for machine access, e.g. a MySQL database 
   with carefully chosen keys, indexes, cross-references etc.
 * Export part of data to some easy-to-edit form, e.g. a spreadsheet.
 * Make changes in the spreadsheet.
 * Generate a diff for that spreadsheet.
 * Apply that diff as a patch to the original data store e.g. in MySQL.

The export step here will usually blow away all sorts of meta-data
vital to the database.  It may also scramble stuff due to type mismatches
or other impedance mismatches.  But all that often just doesn't matter!
Remember, the patch will get applied with all the original meta-data
available.  So we can take edits in Excel and apply them in MySQL, or
edits in CSV and apply them in Excel, etc.  This is a small step 
towards reducing the irreversibility of data exports, where as 
soon as a format conversion happens, there's a lot more grit
in the away any fixes to the data are much
less likely to "swim upstream" to the original source.


