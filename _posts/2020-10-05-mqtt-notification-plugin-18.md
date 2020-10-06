---
layout: post
title:  "Jenkins MQTT Notification Plugin v1.8 Released"
date:   2020-10-06 23:19:00 +0200
categories: jenkins plugin mqtt
---

Over 7 years has passed since I released [the first version of the MQTT Notification Plugin for Jenkins][mqtt-notification-plugin-10]. It's not particularly big or clever, but it serves a simple purpose and it was a good excuse to play around with MQTT and Jenkins Plugin development.

Recently<sup>*</sup> I noticed that users had reported a couple of issues with the plugin, including [a fairly critical memory leak][JENKINS-52767]. So I dusted off the repository to merge in the pull request, tidy up a few things, and try to remember how to create a release. Fortunately the Jenkins team has been hard at work improving the developer experience over the years, as it's now [much easier than I remember][plugin-release], boiling down to `mvn release:prepare release:perform`. The Jenkins wiki has also been deprecated for plugin documentation, [migrating over to Github pages][documentation-as-code] to follow the "documentation-as-code" approach.

So with that done and a few plugin dependency updates included for good measure I released v1.8 of the plugin. A short while later someone created an issue for [an actual feature request][feature-request], so hopefully it won't be too long until v1.9 is released!

<sup>*</sup> Actually the issues were raised several years ago but fell off my radar before I did anything about them! Whoops.

[mqtt-notification-plugin-10]: https://github.com/jenkinsci/mqtt-notification-plugin/releases/tag/mqtt-notification-plugin-1.0
[JENKINS-52767]: https://issues.jenkins-ci.org/browse/JENKINS-52767
[plugin-release]: https://www.jenkins.io/doc/developer/publishing/releasing/
[documentation-as-code]: https://www.jenkins.io/blog/2019/10/21/plugin-docs-on-github/
[mqtt-notification-plugin-18]: https://github.com/jenkinsci/mqtt-notification-plugin/releases/tag/mqtt-notification-plugin-1.8
[feature-request]: https://github.com/gdubya/mqtt-notification-plugin/issues/16