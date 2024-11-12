---
layout: post
title:  "Investigate RotND - Rating Windows"
categories: IT
---
|![discord_rating_window1](/../images/20241112/discord_rating_window1.png)
|![discord_rating_window2](/../images/20241112/discord_rating_window2.png)
|![discord_rating_window3](/../images/20241112/discord_rating_window3.png)
|BYG discord server chat

## 1. Introduction

In order to derive the best build for a track, you must determine the hit windows for each rating first.<br> Although moderator Marukyu said the 'perfect' window is ±37.5, it would subject to change after full release. (he's saying is even ambiguous; is it the whole perfect window or a perfect window which is not attatched with 'E-' or 'L-'?) You also need thresholds for 'great', 'good', and 'ok', so we need to find out the values from the provided game data.

[For those who only want to get the results](#it-means)

## 2. inspect data

Then where do the data reside? Because *Rift of the NecroDancer Demo(RotND)* is a steam game, general path for game data is `C:\Program Files (x86)\Steam\steamapps\common\Rift of the NecroDancer Demo` (Windows 11). (By further search in file explorer, I also found that save, logs, etc. are in `C:\Users\[USER_NAME]\AppData\LocalLow\Brace Yourself Games\Rift of the NecroDancer Demo`.)<br>
With a first glance, you can easily see that the game is executed on Unity engine.

|![game_folder](/../images/20241112/game_folder.png)
|structure of 'C:\Program Files (x86)\Steam\steamapps\common\Rift of the NecroDancer Demo'

At first, because I've never developed with unity, I had no idea about the general unity game structure. So I asked to ChatGPT immediately😂, and its order was to look at **`RiftOfTheNecrodancer_Data`** folder.

|<img src="/../images/20241112/gpt_q1.png" alt="gpt_q1" width="650" align="right">
|gpt question

|<img src="/../images/20241112/gpt_a1-1.png" alt="gpt_a1-1" width="600">
|<img src="/../images/20241112/gpt_a1-2.png" alt="gpt_a1-2" width="600">
|<img src="/../images/20241112/gpt_a1-3.png" alt="gpt_a1-3" width="600">
|gpt answers

### 2-1. data.unity3d

Let's look at `data.unity3d` first. AI suggested *AssetStudio* or *UABE*, so I downloaded the latest(v0.16.53) *AssetStudio.net6*.<br>
When you execute `AssetStudioGUI.exe`, the first thing you should do is to **specify the unity version**. But how do I know what version of unity is used to build the game? According to ChatGPT, one of the easiest way to obtain it is to open `data.unity3d` with text editor.

|![unity_version_check](/../images/20241112/unity_version_check.png)
|`data.unity3d` opened with notepad++

The file starts with "UnityFS" and there is a version in the same line. Go to `Options-Specify Unity version` and enter the version you found. (I didn't know that it is mandatory, so I end up several reinstallations of AssetStudio and UABE..😢)

|![specify_unity_version](/../images/20241112/specify_unity_version.png)
|enter the unity version (2021.3.34f1 in my case)

Then load `data.unity3d`. You can view some data of assets in 'Asset List' section.<br>
The most valuable data was beatmap json files. They may be used to program "build suggestion system" later.<br>
However, there was no useful data from 'MonoBehaviour' assets for my purpose and they seemed insufficient to cover all data constituting the game. I had to find another source.

### 2-2. Assembly_CSharp.dll

Next target is `Managed` folder. There are lots of .dll files, but our interest is **`Assembly-CSharp.dll`**, which contains the game logics. AI's suggestion was *dnSpy*, *ILSpy*, or *dotPeek*, and I used *dotPeek*. I've never used other programs except that, but it seems that *dnSpy* also provides a way to modify assembly file. Nevertheless, I thought editing game data is quite dangerous because I don't know what kind of security system is present behind the game, and if present, there would be a way to detect such file corruption and punish me.

Open `Assembly-CSharp.dll` file with *dotPeek*. I want to find thresholds for ratings like 'great' or 'good', so search those kind of keywords with 'Assembly Explorer'. It is not hard to find Enum `InputRating`.

|![InputRating](/../images/20241112/InputRating.png)
|Enum `InputRating`

You can find its usages by ctrl+clicking `InputRating` - 'Show Usages' - 'Show in Find Results'.

|![find_results](/../images/20241112/find_results.png)
|search usages for InputRating

It's time to become a half-human, half-compiler. Read all c# files which look like relevant source to process keyboard inputs with threshold data. They are all **disassembled** c# files, so most of the variables have auto-generated names like `num`. I recommend you to copy the relevant source files to an editor and rename those variables appropriately for your understanding.

In conclusion, the process of deriving input rating is as follows:
1. There is a corresponding file for `InputRatingsDefinition` class which contains data for defining rating thresholds.
2. In order to get the input rating with respect to a given time difference between input and note, we need two methods in `InputRatingsDefinition`; `GetRatingPercent()` and `GetInputRating()`.
    1. `GetRatingPercent()`: calculate rating percent defined as<br>
    (int)((1f - (ratio of the time difference over one-sided hit window)) * 100f)
    2. `GetInputRating()`: determine the input rating based on the rating percent and the given rating thresholds

So the higher rating percentage is, the better rating you would get.

### 2-3. bundles

Our next task is clear; Find the appropriate file for `InputRatingsDefinition`.

The remaining data was located at `StreamingAssets\aa\StandaloneWindows64`. There are lots of 'bundle' files which contain sets of assets. It's *AssetStudio*'s time again!

First of all, go to `Options-Export options` and set 'Group exported assets by' as 'container path' for later exploration of assets.

|![export_option](/../images/20241112/export_option.png)
|export options

Next, select `File-Extract folder` and choose `...\StandaloneWindows64` and a destination folder respectively.

Looking at the assets is quite fun. You can find game constituents such as VFX images, a hidden cinematic video, animation images, etc. Among them, what I want to find is the files for `InputRatingsDefinition`. Because this class has property `_ratings`, I search for the keyword. There were 5 candidates;

- `Assets\RhythmRift\Prefabs\Enemies\WyrmHoldInputRatings.json`
- `Assets\Shared\Prefabs\ScoreResults\RhythmRift_InputRatingsDefinition.json`
- `Assets\Shared\ScriptableObjects\RatingsDefinitions\Default_BossBattleInputRatingsDefinition.json`
- `Assets\Shared\ScriptableObjects\RatingsDefinitions\Default_InputRatingsDefinition.json`
- `Assets\Shared\ScriptableObjects\RatingsDefinitions\Default_MinigameRatingsDefinition.json`

Second or fourth one would be our interest. But the fourth one has its property `_perfectBonusScore` with value `0`, not `1`. Thus, data in the second json file could eventually give us the answer.

|<img src="/../images/20241112/input_ratings_def_2-1.png" alt="input_ratings_def_2-1" width="600">|<img src="/../images/20241112/input_ratings_def_4-1.png" alt="input_ratings_def_4-1">
|<img src="/../images/20241112/input_ratings_def_2-2.png" alt="input_ratings_def_2-2">|<img src="/../images/20241112/input_ratings_def_4-2.png" alt="input_ratings_def_4-2">
|`RhythmRift_InputRatingsDefinition.json`, second one|`Default_InputRatingsDefinition.json`, fourth one

I'd like to explain the json file(`RhythmRift_InputRatingsDefinition.json`) in detail.<br>
1. Hit window is divided into beforeBeatHitWindow and afterBeatHitWindow, and they are the same as 175ms.
2. 'Miss', 'Ok', 'Good', 'Great', and 'Perfect' have minimum rating percentage requirements as `minimumValue`s, and if a rating percent satisfies several requirements, then the rating with highest `minimumValue` is chosen.
3. When the rating is 'Perfect', there is a bonus score +2 if the rating percent is greater or equal to `_truePerfectBonusMinimumValue`, else +1 if greater or equal to `_onBeatMinimumValue`.

#### It means,
- Hit window is ±175ms. If input is out of range, then the note is regarded as 'Miss'.

- if input is in the range of **±3.5ms**, it is **true 'Perfect'** with score 557<br>
else if input is in the range of **±17.5ms**, **flawless 'Perfect'**/556<br>
else if input is in the range of **±35ms**, **'Perfect'**/555<br>
else if input is in the range of **±87.5ms**, **'Great'**/333<br>
else if input is in the range of **±122.5ms**, **'Good'**/222<br>
else(**±175ms**) **'Ok'**/111

(values subject to change after game release;<br>
I'm not gonna modify this consistently)

## 3. conclusion

So, our honor Marukyu was lying. Actually, that ±37.5ms is come from another json file(`Default_InputRatingsDefinition.json`), and I think BYG team may have more experimental InputRatingsDefinition json files than us.

For the further goal, to build a program that suggests the vibe activation timings, I need to somehow find and replicate the source code from *AssetStudio* which extracts data from bundles. That would be the next topic of the next post. See ya!
