---
layout: post
title:  "Investigate RotND"
categories: IT
---
|![discord_rating_window1](/../images/2024-11-11-investigate-RotND/discord_rating_window1.png)
|![discord_rating_window2](/../images/2024-11-11-investigate-RotND/discord_rating_window2.png)
|![discord_rating_window3](/../images/2024-11-11-investigate-RotND/discord_rating_window3.png)
|BYG Discord Server chat

- - -
<br>

In order to derive the best build for a track, you must determine the hit windows for each rating first.<br> Although moderator Marukyu said the 'perfect' window is ±37.5, it would subject to change after full release. (he's saying is even ambiguous; is it the whole perfect window or a perfect window which is not attatched with 'E-' or 'L-'?) You also need thresholds for 'great', 'good', 'ok', and 'miss', so we need to find out the values from the provided game data.

Then where do the data reside? Because 'Rift of the NecroDancer Demo(RotND)' is a steam game, general path for game data is ```C:\Program Files (x86)\Steam\steamapps\common\Rift of the NecroDancer Demo``` (Windows 11). (By further search in file explorer, I also found that save, logs, etc. are in ```C:\Users\[USER_NAME]\AppData\LocalLow\Brace Yourself Games\Rift of the NecroDancer Demo```.) In the first path with a first glance, you can easily see that the game is executed on Unity engine.

|![game_folder](/../images/2024-11-11-investigate-RotND/game_folder.png)
|structure of 'C:\Program Files (x86)\Steam\steamapps\common\Rift of the NecroDancer Demo'

At first, because I've never developed a simple game with unity, I had no idea about the general unity game structure. So I asked to ChatGPT immediately😂, and its order was to look at **```RiftOfTheNecrodancer_Data```** folder.

|![gpt_q1](/../images/2024-11-11-investigate-RotND/gpt_q1.png)
|gpt question

|![gpt_a1-1](/../images/2024-11-11-investigate-RotND/gpt_a1-1.png)
|![gpt_a1-2](/../images/2024-11-11-investigate-RotND/gpt_a1-2.png)
|![gpt_a1-3](/../images/2024-11-11-investigate-RotND/gpt_a1-3.png)
|gpt answers

Let's look at ```data.unity3d``` first. AI suggests AssetStudio or UABE, so I downloaded the latest(v0.16.53) AssetStudio.net6.<br>
When you execute AssetStudioGUI.exe, the first thing you should do is to **specify the unity version**. But how do I know what version of unity is used to build the game? According to ChatGPT, one of the easiest way to obtain it is to open ```data.unity3d``` with text editor.

|![unity_version_check](/../images/2024-11-11-investigate-RotND/unity_version_check.png)
|```data.unity3d``` opened with notepad++

The file starts with "UnityFS" and there is a version in the same line. Go to ```Options-Specify Unity version``` and enter the version you found. (I didn't know that it is mandatory, so I end up several reinstallation of AssetStudio and UABE..😢)

|![specify_unity_version](/../images/2024-11-11-investigate-RotND/specify_unity_version.png)
|enter the unity version (2021.3.34f1 in my case)

Then load ```data.unity3d```. You can view some data of assets in ```Asset List``` section.
