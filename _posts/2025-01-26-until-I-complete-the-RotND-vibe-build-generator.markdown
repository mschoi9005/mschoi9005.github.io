---
layout: post
title:  "Until I Complete the RotND Vibe Build Generator"
categories: IT
youtubeId: dN4QpQWJ_xY
---

## 0. Before Getting Started...

|![BYG_Terms_of_Use](/../images/20250126/BYG_Terms_of_Use.png)
|Terms of Use: decompiling is not allowed(<https://braceyourselfgames.com/terms-of-use/>{:target="_blank"})

Unfortunately, I found it after I've referenced code about the vibe duration mechanism, code about how the input ratings definition works, and several json files for beatmap and game mechanism(enemy, trap, input rating) from decompiling; I admit that my work already violated the BYG programmers' rights. But who knows how BYG will think about it?

Some examples of prohibited actions are currently ignored by BYG (e.g. uploading gameplay video) because they act as promotions of the game. If they ban me from the game, then I will accept it, buy another copy of the game, and then enjoy the game without a program support. That's it.

<br>

## 1. Introduction

Many of the contemporary rhythm games adopt 'fever' system. In those games, a player may activate fever after the player satisfies a predefined condition and get more score than usual. *Mush Dash* and *DJMAX RESPECT V*, for example, let notes give a fixed amount of fever gauge and the fever be activated itself after the gauge is full.

These games also support manual fever mode where a player should activate the fever manually by pressing a button. As you know, it is not easy to take care of both fever activation timing and accuracy. However, this 'manual fever' takes an important role in scoring. If there exists a fever activation timing that contains more notes than usual, it leads to higher score cap which is impossible to be reached with auto fever mode.

The game in our interest, *Rift of the NecroDancer(RotND)*, only support **manual fever** (vibe power). The vibe power increases score multiplier by 2, so it is inevitable to investigate and remember the vibe power activation timings if you are aiming for the first place. I knew I was talented at this game since *RotND Demo*, and I wanted to maximize my potential with optimal vibe paths. That's why I started to create the <strong id="vbg">RotND Vibe Build Generator(VBG)</strong>.

<br>

## 2. Project Progression

As a premise, I assumed that build optimization is not possible without explicit simulation, mainly due to dynamic traps and enemy collisions(zombies and headless skeletons)

At first, It seemed that reusing decompiled C# files to simulate the game and extract data was the easiest way to make a program, for there is no need to implement the game. In short, however, I failed; I couldn't get the very internal codes of `UnityEngine` library because they are just hidden.
Actually, original C# code and Unity C# code are not compatible; for example, C# doesn't provide the `Update()` API, which is called every single frame in Unity.

The next approach is to mimic the abstractions the game made use of. Although I cannot copy every line of the code, the high level codes the BYG programmers wrote are still present.
I was new to C# at that time and was only able to guess what a line of code would do, so it was not effective to keep using C#. I've made my mind to use Python because the interest was the ease of implementation, but not the performance.
In one sentence, with that approach, I successfully finished the program that automatically gives the possible vibe activation timings for the highest practical score.

<br>

## 3. Mechanism

The brief description of the program mechanism follows:<br>
(You can find the source code at <https://github.com/isocy/RotND_build>{:target="_blank"})

The major abstractions are the followings:

|`Map` |There is a map. The map has 3 columns and 9 rows. Each cell in the map is called a `Grid`. Then there are total 27 grids in the map. Each grid may have several nodes.
|`Node` |Each node has an object and a cooltime. An `Object` is either an enemy or a trap; that is, a node is either an enemy node or a trap node. The `cooltime` indicates how many beats are needed for the node to take a move.
|`Beat` |It stores information about what action row and when the player should press.

<p>
The given beatmap json file is first converted into a list of nodes. Then each node appears at the top of the map at each appropriate timing. For each timestep, either a new node is added to the map or some nodes take a move. If an enemy node arrive at row 0, it means the enemy should be hit now, so a beat instance is created. The simulation is continued until there is no more new nodes and the map is empty.
Finally, from the list of beat instances, the best build is derived, considering all the possible cases.
</p><br>

|Input |a json file for beatmap (variant) / enemy database, and input ratings definition (fixed)
|Output |practice start number for each vibe chain and possible 'builds'

Here, `Build` contains four elements: a partition of vibes, the total number of enemies defeated when the vibe is activated, the list of candidate `BeatCnt`s which corresponds to the partition, and the expected score.

- the `partition` of vibes: 2 vibe charges may be activated at once or seperately / while vibe is activated, additional vibe chains may be completed<br>
-> partition the use of vibe charges
- `BeatCnt`: has <code id="start_beat">start_beat</code>, <code id="cnt">cnt</code>, and `beat_diff`. It describes the situation that what if a vibe is just activated at `start_beat`.
    - `cnt` counts the number of enemies defeated when the vibe is activated.
    - (`beat_diff`) = (The last enemy's beat) - (`start_beat`)<br>
    -> implicitly indicates how hard it is to cover `cnt`

<br>

### 3-1. Build Manipulation

Of course, the game is simulated on Finite State Machine(FSM), so it seems it is safe to assume that the score remains the same if we provide the same keyboard input at the same time. But here's the thing; players have their own distinct resource(CPU, memory, etc.) status and this status do affect gameplay, thus possible build. Moreover, the ideal build sometimes requires extreme performance which is almost impossible for most players. That is, users should manipulate some parameters to find the build optimal for their circumstances.

[VBG](#vbg) provides three ways of customization.<br>
(<https://github.com/isocy/RotND_build/blob/main/RotND_build/Global/__init__.py>{:target="_blank"})

1. [`start_beat`](#start_beat) exception

    There are often several vibe paths which give you the same amount of score, and VBG recommends the vibe path which activates the vibe early. Occasionally, you may want to choose another path due to your skill issue or build stability.

2. [`cnt`](#cnt) reduction

    VBG was developed as generous as possible in order for it not to miss any builds, so it could give you a build that is impossible to carry out, and it becomes worse when the bpm of a beatmap gets higher due to shortened input range. Fortunately, you can "cut" some end notes of vibe by decreasing `cnt` value.

3. intended "great" strategy

    All "perfect" is the best way to maximize score, yet it is not always the case. Given that the current score multiplier as 4, the scores of "non-critical perfect" and "great" during vibe are 2220 and 2664, respectively. If we include one more note into vibe by sacrificing one "perfect", we will get 444 more. VBG supports the score calculation in which this kind of strategy is involved.

<br>

## 4. Result

<table>
<tr>
<td>
<pre>
RAW_BEATMAP_PATH = REACH_FOR_THE_SUMMIT_IMPOSSIBLE_PATH

ONE_VIBE_START_BEATS_EXCEPT: list[float] = []
TWO_VIBES_START_BEATS_EXCEPT: list[float] = []
THREE_VIBES_START_BEATS_EXCEPT: list[float] = []

ONE_VIBE_START_BEATS_LOOSE: list[tuple[float, int]] = [
    (146, 3),
    (146.5, 1),
    (178, 3),
    (178.5, 1),
    (708, 3),
    (708.5, 1),
    (710, 3),
    (710.5, 1),
    (712, 3),
    (712.5, 1),
]
TWO_VIBES_START_BEATS_LOOSE: list[tuple[float, int]] = []
THREE_VIBES_START_BEATS_LOOSE: list[tuple[float, int]] = []

TARGET_PARTITION: tuple[int, ...] = ()
GREAT_START_BEATS: list[tuple[float, int]] = []


Practice Start Numbers:
[65, 165, 230, 378, 485, 609]

(1, 1, 3, 1), 390
[146.5 61 21.0, 178.5 61 21.0, 457.5 202 64.0, 708.5 66 21.0], 3718322
</pre>
</td>
</tr>

<tr>
<td>
An example of a result text file which consists of input arguments and generated build
</td>
</tr>
</table>

<table>
<tr>
<td>
{% include youtubePlayer.html id=page.youtubeId %}
</td>
</tr>

<tr>
<td>
The execution of the build above
</td>
</tr>
</table>

This video is a gameplay of the hardest track among the RotND Vol.1 official tracks with a build from [VBG](#vbg). Thanks to it, I took the 1<sup>st</sup> place with big score difference from the previous 1<sup>st</sup>. You can also watch other gameplay videos that got help from VBG if you visit [my youtube channel](https://www.youtube.com/@isocy){:target="_blank"}.

By programming VBG, I was able to practice an abstraction of the given game mechanism. There was no need to know about the monster animations or sound effects for build search, so I dropped these and heightened the level of the abstraction. I believe that this problem solving/abstraction experience made me take a step forward for a good developer.
