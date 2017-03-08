# Continuous Integration is a Practice

Continuous Integration (CI) is a Practice, not a tool. You do not automatically practice Continuous Integration if you use a build server like TeamCity. You can do CI without a build server, armed with a rubber chicken (more on that later). You do not practice CI if you utilize Feature Branches. Every blog post about CI emphasizes these facts. 

However, in every new project we always first have to clarify as a team what the words Continuous Integration mean and then realize that the team is not practicing CI. I find that extremely frustrating. I do not want to argue the defintion, especially when it is already clearly defined by much smarter people than me. 

Let's add to that pile of books and articles and hope one of these days it's going to stick. 

We'll first provide the CI defintion, then why it is probably often misunderstood and then why it's useful.

## CI Definition

According to Wikipedia the defintion is:

> In software engineering, continuous integration (CI) is the practice of merging all developer working copies to a shared mainline several times a day. Grady Booch first named and proposed CI in his 1991 method, although he did not advocate integrating several times a day. Extreme programming (XP) adopted the concept of CI and did advocate integrating more than once per day - perhaps as many as tens of times per day.

If you have a build server, that runs a build for every commit on every branch, you are doing continuous building. You are not integrating your code, so you are not doing CI.

Martin Fowler describes the key points in his [blog post](https://martinfowler.com/articles/continuousIntegration.html):

* Maintain a Single Source Repository
* Automate the Build and make it Self-Testing
* Everyone Commits To the Mainline Every Day
* Every Commit Should Build the Mainline on an Integration Machine
* Fix Broken Builds Immediately
* Keep the Build Fast
* Test in a clone of the production environment
* Make it easy for anyone to get the latest executable version
* Everyone can see what’s happening 
* Automate deployment
* Don’t go home after checking in until the system builds

It does not matter if you use a build server. By definition you cannot do feature branches if you do Continuous Integration. 

### Continuous Integration Certificate

Perhaps the definite book on achieving "Reliable Software Releases through Build, Test, and Deployment Automation" is [Continuous Delivery](https://www.amazon.com/Continuous-Delivery-Deployment-Automation-Addison-Wesley/dp/0321601912/ref=sr_1_2?ie=UTF8&qid=1488958912&sr=8-2&keywords=continuous+integration). The author, Jez Humble, provides a simple test to see if you are doing CI:

* Raise your hands if you do Continuous Integration.
* Keep your hands up if everyone on your team commits and pushes to a shared mainline (usually shared master in git) at least daily.
* Keep your hands up if each such commit causes an automated build and test.
* When the build fails, it’s usually back to green within ten minutes.

### CI with a Rubber Chicken and without Jenkins

You do not need a build server like Jenkins for that. James Shore describes a way to achieve [CI on a dollar a day](Continuous Integration on a Dollar a Day), armed with a rubber chicken and a crappy old build computer with access to your version control. 

> * Run the build/test script locally and make sure it passes 100%.
* Get the rubber chicken from its resting place. If it's not there, find the person who has it and annoy them until they're done with their check-in.
* Get the latest code from the repository and run the build script again just in case. If it doesn't pass, you know that there's some integration problem with the code you just got. Bitch and moan, put the chicken back, and get the person who last checked-in to help you out. Start over when you're ready.
* Check in your code.
* Walk over to the crappy build computer, get the latest code from the repository, and run the build script again. If it doesn't pass, revert (undo) your check-in. You installed some new software, or modified an environment variable, or set a registry setting, or forgot to add a file to the repository, or something. Anyway, you need to fix the problem on your computer and try again. You can hang on to the chicken for a moment, but give it back (and start over) if anybody needs it.
* Ring the bell. (Everyone else, that's your cue to applaud, or otherwise rejoice. "Yaaay.") Put the chicken back. You're done.

## Why everyone is initially always wrong about CI

I also belong to the category of people who misunderstood what CI is. I think there are two reasons for the confusion around Continuous Integration:

1. Every build server these days calls itself a CI server
2. Integration as a word is slightly overloaded for software

Gitlab-CI, Travis-CI, Bamboo CI and Thoughtworks Snap-CI. It's always in the name. Sadly the trend to keep the practice in the tool name is continuing. Thoughtworks has another build server called GO CD. Thoughtworks has done a lot to popularize CI but through their naming conventions they are obscuring the intent of CI. 

I also think that integration as a word is too overloaded. You do not need Integration Tests to practice Continuous Integration. You should have them to catch issues early though. The I in CI stands for integrating your changes.

## CI Benefits

To understand the benefits of CI, we'll start by contrasting it with another strategy, feature branching.

The basic idea of a feature branch is that when you start work on a feature (or UserStory if you prefer that term) you take a branch of the repository to work on that feature.

The advantage of feature branching is that each developer can work on their own feature and be isolated from changes going on elsewhere. It also makes code reviews easier to organize. And picking just the right stories for our release gets easiert as well. As a team we can basically cherry-pick what we deem ready for release.

Of course at some point the feature branch needs to be integrated. The first developer has it easy and the feature branch merge is a non-event. Master now suddenly has many changes. Each developer needs to pull and merge these new mainline changes to his feature branch, then push the feature branch changes to master and the cycle begins again. 

Each developer could have introduced semantic changes (method rename, extracting a class, removing an unreferenced class etc.) in code that another developer is using. These conflicts are very hard to detect with merge tools and usually require a developer to fix manually. The longer a branch lives and the more branches you have, the higher the chance for this to happen.

In my experience the last 1,5 days of a sprint are just for coordinating these scary feature branch merges and code reviews. In a big scrum team you could have 9 feature branches you need to push to mainline and 9 code reviews to conduct. As long as no one has refactored anything it should be fine. But that's your problem. 

> Once a team is afraid to refactor to keep their code healthy they are on downward spiral with no pretty end.

> -- [Martin Fowler](https://martinfowler.com/bliki/FeatureBranch.html).

Instead make integration a non-event. Implement really small features that take at most one hour to implement. If they do result in merge conflicts, talk to each other, then together improve the design. 

Continuous Integration is cheap. Not integrating continuously is expensive. If you don’t follow a continuous approach you won't know about merge issues until it is too late and the deadline is looming over our heads.

Because you’re integrating so frequently, there is significantly less back-tracking to discover where things went wrong, so you can spend more time building features.

With CI you can still cherry-pick features. Techniques such as Branch by Abstraction and Feature Toggles help with that. 

To summarize, with CI you 

* Avoid long and scary integrations
* Increase communication in your team
* Have the ability for large-scale refactoring
* Debug less and write more code

## CI Problems

In a blog post Yegor Bugayenko wrote that [Continuous Integration is dead](http://www.yegor256.com/2014/10/08/continuous-integration-is-dead.html). 

The question is simple

> Why the hell shouldn't continuous integration work, being such a brilliant and popular idea?

The problem as well

> Crucially, if the build fails, the development team stops whatever they are doing and fixes the problem immediately

That means you need a disciplined team that does not ignore a broken build.

You can do this using a read-only master branch. No human can commit to this master branch. Then create a script which merges, tests and commits. There are fancier words for this practice: pre-tested/gated/delayed commits. 

Gated checkin is really a pessimistic continuous integration (CI) build. Whereas traditional CI is optimistic and verifies a change after it is checked in, gated checkin performs validation on the change prior to it being committed to version control.

Instead of a script you can also use another tool like Gerrit (originally a code-review tool) or Verigreen which wraps your master branch. Changes are pushed to Gerrit which tests the change on master but only commits it when the test is green. 



It is very important that using gated commits does not result in feature branches or developer branches again. Then we'll lose all CI benefits again. 

 

Theoretically you can achieve this using two branches: integration and master. Every developer works on integration and pushes changes. The build server merges the changes to master if the integration build is green. That means that we still integrate our code with our teammates code. That also means we get his/her possibly broken changes and need to fix them before our push to integration can be green.

TeamCity has Pre-Tested Commits. Jenkins can do this with the [Git Plugin and pre-build branch merging](https://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin#GitPlugin-UsingGit,Jenkinsandprebuildbranchmerging). TFS supports this through branch policies. You can:

* Block pull requests completion unless there is a valid build
* Require a minimum number of reviewers per pull request

You can achive this benefit by giving every developer 

If the team is not disciplined you can prob

This can lead to an overall feeling in the team that breaking the build is wrong, and is to be avoided at any cost. This might not be a healthy attitude, since the original purpose of a build process was always to find out problems. You might say that if the build breaks, then the build did it’s job. This culture of “breaking the build is bad, and it’s your fault” can lead to developers hiding their efforts, in an effort not to look stupid in front of their peers. in essence, a private build can lead to moving from a group effort, into individual based comparison system “who breaks the build the most?”.

If developers always get the build failing privately, developers will feel they have to solve the build privately. when the build fails publicly, it can trigger help from the rest of the team to solve the problem faster since more brains are involved.

## CI Requirments

## TBD

Trunk Based Development is synonymous with Continuous Integration