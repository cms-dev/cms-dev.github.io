## How to contribute

If you want to help translate CMS to your language, you can do so on
[Weblate](https://hosted.weblate.org/engage/cms/).

If you are interested in implementing major features, please [contact the core
team](./contact.md) before starting the design and implementation, so that the
effort can be better organized and we can make sure there is no duplication of
work.

For every change, you should ensure that:

<!-- TODO: update this, maybe use Flake8 instead of pep8 command? -->
- no PEP8 errors are introduced (check with the pep8 command)
- the Pylint score does not decrease too much (use pylint)
- you add your copyright line in the header of all files you changed substantially;
- if it is your first contribution, your name is in AUTHORS.txt.

The process to send changes is as follows:

- fork cms-dev/cms on github;
- create a new branch and commit there your change(s);
- create a pull request against the main branch of cms-dev/cms

The following are some ideas for self-contained projects to expand CMS’s
features. All of these require some knowledge of Python, SQLAlchemy and
relational databases. Experience with the organization of programming contests
and training camps is a plus.

## :material-head-lightbulb: Contribution ideas

If you are interested in developing one or more of these ideas, please
[contact us](./contact.md), and we will arrange for a member of the core team to
provide guidance.

### Representative submissions

Allow admins to store, for each task a set of representative submissions,
intended to have a precise result (for example: a wrong submission which should
score zero points, a correct one which should score full points, and an
intermediate one which should score half of the points. Admins should be able to
test at any moment that the representative submissions score exactly as
expected.

More precisely, there should be a way to associate to tasks a set of tuples
(submission, (name), expected result); moreover, AWS should be extended to allow
the recomputation of the scores associated to the representative submissions and
to warn the admins when the score is different from the expected.

#### Expected changes

1. Adding the new fields for the representative submissions to the task DB
   class, together with their metadata (name and expected results); writing the
   updater.
1. Adding hooks to the YamlImporter for easy testing.
1. Adding AWS UI to show and edit the representative solutions and their
   metadata (including the actual results), and to trigger a re-evaluation.
1. Letting ES and SS know that there is a new “type” of submission and make sure
   that the path taken by these submissions is correct and do not spill over the
   paths taken by the contestants’ submissions.

**Additional requirements:** none.

**Difficulty:** medium.

**Size:** medium.

### Expand test coverage

CMS currently has a functional test that runs all the services (apart from RWS)
and simulating the creation of contests and tasks by the admins, and some
submissions from the user. There are some unit tests but very limited. The task
is to increase the coverage of the unit tests (possibly reorganizing code to
make it more testable), extend the functional tests to other aspects (like
importing of tasks) and to speed up the execution of functional tests.

**Additional requirements:** testing experience (especially in Python).

**Difficulty:** high.

**Size:** big.

### In-browser code editing

Many online services for programmers (e.g., github) are starting to offer
in-browsing editing of source files. It would be very cool to add these
capabilities to CMS, allowing editing of submissions directly from the browser.
Existing open source projects could be used, like
[Monaco](https://microsoft.github.io/monaco-editor/). There is a lot of
potential for follow ups, like for example branching and auto-snapshotting.

#### Expected changes

1. Experimenting with the existing in-browser plugins to decide which one to
   use, using as metrics: browser compatibility, performances, user experience,
   offline capabilities (many contests don’t allow Internet access).
1. Modifying CWS to include a new tab for each task to allow editing and
   submitting of solutions from scratch.
1. Adding a canvas to the new tab to display the existing submissions (with some
   basic info, like timestamp and score) to allow contestants to edit and submit
   previously submitted solutions (that will be displayed as branching out from
   the starting solution); allowing contestants to branch multiple times from a
   single submission.
1. Implementing auto-snapshotting after x minutes and/or y chars of delta;
   deciding how to store the snapshot (probably as a submission with a special
   flag); adding the new field to the submission DB object; writing the updater.

**Additional requirements:** JavaScript.

**Difficulty:** medium to high, depending on the follow ups.

**Size:** big.

### Contest statistics and graphs

After an official contest is often nice to have some easily retrievable
statistics about scores distributions, language usage, and so on.

#### Expected changes

1. Add a new page to AWS to contain graphs and downloadable data.
1. Think of the useful aggregated data.
1. Actually implement the graphs and let admins download CSV data.

**Additional requirements:** JavaScript.

**Difficulty:** low.

**Size:** small.

### Exposing complexity guess

There is already an experimental code in cmscontrib that tries to compute the
complexity of the contestants’ solutions based on the size of the input. A
similar computation was suggested as a more sane way of scoring at some IOI
conference, though it has many problems in reliability and is subject to
cheating. Still, it would be interesting for admins to see this information, for
example to have an idea of the cleverness of the solutions of two contestants.

#### Expected changes

1. Adding a new appropriate field to the submission DB object to store the
   complexity of the submission; writing the updater.
1. Writing a new service that “subscribes” to the event “submission scored” and
   computes the complexity of the submission.
1. Exposing the complexity in an appropriate UI in AWS.

**Additional requirements:** statistical regression.

**Difficulty:** low.

**Size:** medium.

### Ease the usage of CMS in a training settings

At least, make sure that all interfaces are usable with a high number of
contestants, tasks (and of contests for RWS). This may mean to have adaptive UIs
after a certain threshold (for example, not showing a single page in RWS if you
have 10k contestants).

In the [related](./related.md) page there is a 

**Additional requirements:** training camps experience.

**Difficulty:** medium to high, depending on how thorough is the investigation.

**Size:** big.

### Team competitions

Many online contests are in the form of team competitions: multiple contestants
cooperate to solve the same set of problems. The easiest way to implement this
without changing a lot of code is to put the team information in the user table
and add a new table for team participants (“sub-users"?). As extras, with
in-browser code editing it would be nice to allow real-time cooperation, and
also for admins it would be nice to be able to see who wrote a specific piece of
code.

#### Expected changes

1. Adding the new table to the DB; writing the updater.
1. Modifying YamlImporter and similar to accept the new information.
1. Modifying CMS to understand sub-users and allow the login with the sub-user
   credential but contributing to the team submissions.

**Additional requirements:** none.

**Difficulty:** low.

**Size:** medium.

### Visual solutions in RWS

(An old attempt at this can be found in
<https://github.com/cms-dev/cms/pull/434>)

Allow spectators to be more involved with the contest by giving them the
possibility of looking at the internal functioning of the participants’
submission. During the evaluation process, CMS will record in the submission one
or more opaque pieces of data, that will be passed to RWS; on his side, RWS will
have a plugin able to translate back each piece of data into the solution given
by the contestant’s submission to a simple input, and to show this solution to
the spectators. Bonus point for implementing this also for interactive tasks and
showing the spectators an animation.

#### Expected changes

1. Adding new opaque fields to the submission DB class; adding a boolean to the
   inputs marking those whose solution will be exported; writing the updater.
1. Designing the system that will fill the opaque fields starting from the
   submission; decide to do that at the evaluation time, or to create a new job
   type; implementing the proposed solution.
1. Modifying SS and RWS to understand the new data being passed.
1. Implementing the plugin system for RWS, mimicking what CMS has for task types and score types.
1. Modifying RWS to let the plugins draw the UI of the solution.
1. Implementing some examples task to showcase the new feature.

**Additional requirements:** contest organization experience, JavaScript.

**Difficulty:** high.

**Size:** big.

### Mobile-ready RWS (or an app?)

**NEW:** a mobile client is now available! **Ranky** [:simple-android:](https://play.google.com/store/apps/details?id=org.ioinformatics.ranky) [:simple-github:](https://github.com/wil93/ranky)

The RankingWebServer works awfully on mobile devices (mostly phones) due to
their small screens and to the mouse-driven interaction model. It’s advisable to
redesign the client to handle these use-cases better, or to develop dedicated
applications for mobile (Android, iOS, etc.). The latter ones should get the
data from the same server that provides the current in-browser scoreboard (i.e.
communicate over the existing HTTP API). Notifications could be interesting too.

**Additional requirements:** JavaScript, accessible web development; possibly
app development.

**Difficulty:** high (there will be no guidance available from the core team for
native app development).

**Size:** big.

### Non-interactive scoreboard for public screening

Important competitions may have [on-site large screens, projectors or
“totems”](./assets/totem-ioi2012.jpg) to display the live scoreboard to the
audience (team leaders, guests) so they don’t have to use their laptops or
phones. This use-case is most likely best addressed by adding some kind of
"totem" or "demo" mode to the RankingWebServer frontend, which can be toggled
on/off.

**Additional requirements:** JavaScript.

**Difficulty:** medium.

**Size:** small.
