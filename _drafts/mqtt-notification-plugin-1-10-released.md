---
layout: post
title:  "Jenkins MQTT Notification Plugin v1.10.0 Released"
date:   2026-08-23 22:12:00 +0200
categories: [jenkins]
tags: [plugin, mqtt, cicd]
---

Back in 2020, when I wrote about [releasing v1.8 of the MQTT Notification Plugin for Jenkins][mqtt-plugin-18], I signed off with:

> *A short while later someone created an issue for [an actual feature request][feature-request], so hopefully it won't be too long until v1.9 is released!*

Well, true to the definition of maintaining a side project at a "leisurely pace", only nearly six years have passed. But good things come to those who wait: **v1.10.0 is now officially released!**

## What's New

### 🚀 Build Start Notifications
The headline addition is [Issue #16][feature-request]: you can now trigger MQTT messages when a build **starts**, not just upon completion. 

You can configure a dedicated `startTopic` and `startMessage` (or let it fall back to the default topic), complete with dynamic variable expansions like `${BUILD_DISPLAY_NAME}` and `${JOB_NAME}`.

### 🛠️ Enhanced Pipeline Support
The plugin provides full support for Jenkins Pipeline via the `mqttNotification` step. For example, in a Declarative Pipeline:

```groovy
post {
    always {
        mqttNotification(
            brokerUrl: 'tcp://localhost:1883',
            topic: 'jenkins/builds/${JOB_NAME}',
            message: '{ "job": "${JOB_NAME}", "build": "${BUILD_NUMBER}", "status": "${BUILD_RESULT}" }'
        )
    }
}
```

### ☕ Modernised Toolchain & JEP-229 Continuous Delivery
Behind the scenes, the plugin has undergone a massive modernization:
- Jenkins LTS baseline bumped to `2.568.2`.
- Java 21 build matrix and runtime support.
- Fully automated continuous delivery pipeline via JEP-229 and Release Drafter.

The release is now live on the [Jenkins Plugin Site][plugin-site] and [GitHub Releases][plugin-releases].

---

*Now, let's see how long it takes before v1.11.0...*

[mqtt-plugin-18]: {{ site.baseurl }}/posts/mqtt-notification-plugin-18/
[feature-request]: https://github.com/gdubya/mqtt-notification-plugin/issues/16
[plugin-site]: https://plugins.jenkins.io/mqtt-notification-plugin
[plugin-releases]: https://github.com/jenkinsci/mqtt-notification-plugin/releases
