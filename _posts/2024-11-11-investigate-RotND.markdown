---
layout: post
title:  "Investigate RotND"
categories: IT
---
![discord_rating_window1](/../images/2024-11-11-investigate-RotND/discord_rating_window1.png)

![discord_rating_window2](/../images/2024-11-11-investigate-RotND/discord_rating_window2.png)

![discord_rating_window3](/../images/2024-11-11-investigate-RotND/discord_rating_window3.png)

<br>
- - -
<br>

In order to derive the best build for a track, you must determine the hit windows for each rating first.<br> Although moderator Marukyu said the 'perfect' window is ±37.5, it would subject to change after full release. (he's saying is even ambiguous; is it the whole perfect window or a perfect window which is not attatched with 'E-' or 'L-'?) You also need thresholds for 'great', 'good', 'ok', and 'miss', so we need to find out the values from the provided game data.

Then where do the data reside? Because 'Rift of the NecroDancer Demo(RotND)' is a steam game, general path for game data is ```C:\Program Files (x86)\Steam\steamapps\common\Rift of the NecroDancer Demo``` (Windows 11). (By further search in file explorer, I also found that save, logs, etc. are in ```C:\Users\[USER_NAME]\AppData\LocalLow\Brace Yourself Games\Rift of the NecroDancer Demo```.) At the first path, you can easily see that the game is executed on Unity engine.


