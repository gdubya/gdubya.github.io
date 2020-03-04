---
layout: post
title:  "Azure Cognitive Services - Speech API Demo"
date:   2020-03-04 22:32:00 +0100
categories: azure cognitive speech
---

I've put together a little demo of the Speech API from [Azure Cognitive Services][azure-cognitive-services]. I'm surprised at how well it understood my Norwegian!

The repo can be [found on GitHub][gdubya-speech-demo].

## Example

### Input
I read the first couple of paragraphs from [this Aftenposten article][aftenposten-article]:
```
Søndag kveld hadde 19 nordmenn testet positivt for koronaviruset.

Kanskje ville viruset funnet veien til Norge uansett. Men at Oslo universitetssykehus skulle bidra så til de grader til spredning av det, er nærmest utilgivelig.
```

### Result

```
Recognized: Søndag kveld hadde 19 nordmenn testet positivt for kolonna viruset
Recognized: kanskje ville videre seg funnet veien til norge uansett
Recognized: men at oslo universitetssykehus skulle bidra så til de grader til spredning av det er nærmest utilgivelig
Recognized: stopp
```

It's a very small dataset, but the calculated [Word Error Rate (WER)][wer-link] in this example appears to be only 8.5%, which is pretty good!

|     |              |
|-----|--------------|
| S   | 2            |
| D   | 0            |
| I   | 1            |
| C   | 33           |
| N   | 35           |
| WER | 3/35% = 8.5% |




[azure-cognitive-services]: https://azure.microsoft.com/en-gb/try/cognitive-services/
[gdubya-speech-demo]: https://github.com/gdubya/azure-speech-demo/
[aftenposten-article]: https://www.aftenposten.no/meninger/kommentar/i/Jo4apm/norge-kan-stanse-koronaviruset-joacim-lund
[wer-link]: https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/how-to-custom-speech-evaluate-data