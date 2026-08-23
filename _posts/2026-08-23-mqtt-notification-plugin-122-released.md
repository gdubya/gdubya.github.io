---
layout: post
title:  "Jenkins MQTT Notification Plugin: Release 122.vf67b_9da_d6f70"
date:   2026-08-23 22:12:00 +0200
categories: [jenkins]
tags: [plugin, mqtt, cicd, antigravity]
redirect_from: /posts/2026/08/23/mqtt-notification-plugin-122-released/
---

Back in 2020, when I wrote about [releasing v1.8 of the MQTT Notification Plugin for Jenkins][mqtt-plugin-18], I signed off with:

> *A short while later someone created an issue for [an actual feature request][feature-request], so hopefully it won't be too long until v1.9 is released!*

Well, true to the definition of maintaining a side project at a "leisurely pace", nearly six years have passed. And if you were expecting a simple `v1.9.0` or `v1.10.0`, the Jenkins ecosystem has moved on too! Under Jenkins' modern [JEP-229 Continuous Delivery][jep-229] model, releases are now automated with changelist-based version numbers—making **`122.vf67b_9da_d6f70`** the latest official release.

Credit where it's due: getting this release over the finish line—tackling the open issues, modernising the test suite, and overhauling the CI/CD pipelines—was in large part thanks to pair programming with [Google Antigravity][antigravity].

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

### ☕ Modernised Toolchain & Continuous Delivery
Behind the scenes, the plugin has caught up with modern Jenkins standards:
- Adopting JEP-229 continuous delivery with automated changelog generation via Release Drafter.
- Jenkins LTS baseline bumped to `2.568.2`.
- Java 21 build matrix and runtime support.

The new release is live on the [Jenkins Plugin Site][plugin-site] and [GitHub Releases][plugin-releases].

---

*Now, let's see if anyone's actually still using the plugin...*

[mqtt-plugin-18]: {{ site.baseurl }}/posts/mqtt-notification-plugin-18/
[feature-request]: https://github.com/gdubya/mqtt-notification-plugin/issues/16
[plugin-site]: https://plugins.jenkins.io/mqtt-notification-plugin
[plugin-releases]: https://github.com/jenkinsci/mqtt-notification-plugin/releases
[jep-229]: https://github.com/jenkinsci/jep/tree/master/jep/229
[antigravity]: https://antigravity.google
