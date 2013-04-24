---
layout: post
title: "Explorations in Python and TeamCity"
description: "Experiments performing continuous integration testing with TeamCity"
category: 
tags: ["python", "teamcity", "testing"]
---
{% include JB/setup %}

##Experiments performing continuous integration testing in Python with TeamCity

At [Bazaarvoice](http://www.bazaarvoice.com), we use [TeamCity](http://www.jetbrains.com/teamcity/) for continuous integration testing. This has proved immensely useful for most of our Java projects, however as I am developing a Python application, I wanted to see if I could get TeamCity to run testing on my program. I already had a set of unit tests with pretty good code coverage, running fairly fast on my dev machine, so this was mainly an exercise to keep myself honest and track mistakes. No longer could I conveniently "forget" to run the tests before deploying, I want publicly fail within moments of pushing buggy code to master!

While proper unit testing is a topic covered elsewhere in far more detail and with far more expertise than I can provide, I want to share one tidbit that I have found to be the most important moment of "oh, now I get it..." as I've been learning how to properly design unit testing. Anytime something breaks, whether it's in production, QA, or on your dev machine, immediately make sure there's a corresponding unit test that breaks as well. If there isn't, write one before fixing the bug. That way you're sure it won't happen again. I won't overly evangelize unit testing, as I've been on teams who use it as a crutch, but with this one rule you can save yourself hours fixing bugs you've already fixed.

Anyway, as it turns out, JetBrains has already released a module called [teamcity-messages](https://github.com/JetBrains/teamcity-python), available via `easy_install` or `pip`. This module provides hooks for the `unittest`, `nose`, and `py.test` unit testing frameworks in Python. I'm simply using `unittest`, though I imagine working with the other two is similarly easy. The module is designed to be able to detect whether or not the test is being run from TeamCity, and if so, format the output in a way that TeamCity can immediately report and track tests as they run. When I started my project and unit tests took five seconds to run, there wasn't much point, but now that unit testing takes a minute or so it can be helpful. For some of our larger projects which take an hour or more to run the full test suite, it's critical to see failures immediately.

### Adding teamcity-messages to your tests

Using `unittest` with tests split into multiple modules can be fairly boilerplate - have one testing script that uses `unittest.defaultTestLoader`, then use `unittest.TextTestRunner` to run the loaded suite of tests. `teamcity-messages` plays perfectly into that, as all you need to adjust is what runner you use. Here's my current code:

	from teamcity import is_running_under_teamcity
	from teamcity.unittestpy import TeamcityTestRunner
	import unittest
	from tests import testApp, testService, (etc...)

	if __name__ == '__main__':
	    if is_running_under_teamcity():
	        runner = TeamcityTestRunner()
	    else:
	        runner = unittest.TextTestRunner()

	    suite = unittest.defaultTestLoader.loadTestsFromModule(testApp)
	    suite.addTests(unittest.defaultTestLoader.loadTestsFromModule(testService))
	    (etc...)
	    runner.run(suite)

The only change from when I originally wrote the test file was checking to see if we were running under TeamCity, and loading the appropriate test runner. Now, the testing script will run all my tests on both my development machine and TeamCity. No adjustments were needed to any of the tests themselves.

### Getting your agents running

Now that we've got our tests reporting to TeamCity when they are actually run there, lets get them running there. Turns out, this was actually the harder part of the operation. The first step was setting up the VCS root to read from the project on GitHub. This was simple to do from the build configuration settings, and we were able to leave most options at default. Then, in the 'Build Triggering' section, set up a trigger to build after every VCS change. That is, anytime a change to the branch TeamCity is listening to (default is master, but can be changed), it'll perform the build steps. This means you should probably have developers pushing code directly to their own branches, and only push to master when it's ready for public consumption - even if that doesn't mean a release. 

The build steps, too, were fairly simple. In addition to my Python `unittest` script, I also use [Grunt](http://gruntjs.com/) to lint my code. Adding these was as simple as adding two Command Line build runners: `python26 testing.py` for the Python tests, and `node_modules/grunt/bin/grunt` to run Grunt. Python26 was selected due to version conflicts on the build agents. I ran Grunt that way so it didn't have to actually be installed, it could be in the source code. 

However, this was pretty much the end of the simple tasks. Turns out, it isn't perfectly straightforward to install python requirements onto a build agent. For our purposes we ended up installing the requirements on all the agents manually, however to keep this working in the future we'd have to add this step to the build agent deployment script. Additionally, once I added some dependencies to AWS services, transferring the IAM credentials to the testing boxes proved to be a challenge we never ended up surmounting before my DevOps engineer working on the project shifted off, and I needed to continue myself. 

### In Conclusion

Unfortunately, I'm somewhat sad to say (given how much work went into it) that my TeamCity build configuration is paused with the solemn message "Discontinued until I figure out how to make it run without errors." I still run the same exact tests locally, and as I am the only engineer on the project I have no pressing need to ensure all code is tested as it is checked in. In the future, there's a couple things I'd like to do to fix things, however.

I believe I can install the Python modules to the agents as local modules, rather than global installations. I always use [virtualenv](http://www.virtualenv.org/en/latest/), so my `requirements.txt` stays up to date. Since the build runners don't have root permissions, I can't install packages globally. Neither can I easily set up virtual environments on the build agents. I believe I can install them to the source directory with some variation of `pip install --install-option="--prefix=$PREFIX_PATH" package_name` but I haven't gotten into testing this.

As for the AWS credentials, I'm not sure how I'll get them into the build agents. I'm sure other teams have done something so once I get the packages installing correctly, I will ask around. Another alternate is simply mock out all AWS services. For some functions like sending email this makes the most sense anyway. 

Hopefully soon, I'll be back up and running with TeamCity and will make a new post about the changes. Until then, I'll dutifully maintain my unit testing locally in the hopes it may become automated again in the future.
