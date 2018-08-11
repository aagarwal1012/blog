---
layout: post
title: Animated Text Kit
description :  A flutter package to create cool and beautiful text animations. 
tags: [Flutter, Text-Animations, Cross-Platform-App-Development]
author : Ayush Agarwal
time : 8 minutes read
---

<div align="center"><img src="https://github.com/aagarwal1012/Animated-Text-Kit/blob/master/display/cover.gif?raw=true"/></div>

# Table of contents

  * [Installing](#installing)
  * [Usage](#usage)
    * [Rotate](#rotate)
    * [Fade](#fade)
    * [Typer](#typer)
    * [Typewriter](#typewriter)
    * [Scale](#scale)
    * [Colorize](#colorize)
  * [Bugs or Requests](#bugs-or-requests)
  * [License](#license)

# Installing

### 1. Depend on it
Add this to your package's `pubspec.yaml` file:

```yaml
dependencies:
  animated_text_kit: ^1.0.2
```

### 2. Install it

You can install packages from the command line:

with `pub`:

```css
$ pub get
```

with `Flutter`:

```css
$ flutter packages get
```

### 3. Import it

Now in your `Dart` code, you can use: 

```dart
import 'package:animated_text_kit/animated_text_kit.dart';
```


# Usage

You can override the `duration` of each animation by setting duration in each AnimatedTextKit class. For example:
```dart
FadeAnimatedTextKit(
  duration: Duration(milliseconds: 5000),
  text: ["do IT!", "do it RIGHT!!", "do it RIGHT NOW!!!"],
  textStyle: TextStyle(fontSize: 32.0, fontWeight: FontWeight.bold),
);
```

## Rotate

```dart
Row(
  mainAxisSize: MainAxisSize.min,
  children: <Widget>[
    SizedBox(width: 20.0, height: 100.0),
    Text(
      "Be",
      style: TextStyle(fontSize: 43.0),
    ),
    SizedBox(width: 20.0, height: 100.0),
    RotateAnimatedTextKit(
      text: ["AWESOME", "OPTIMISTIC", "DIFFERENT"],
      textStyle: TextStyle(fontSize: 40.0, fontFamily: "Horizon"),
    ),
  ],
);
```
**Note:** You can override transition height by setting the value of parameter `transitionHeight` for RotateAnimatedTextKit class.

## Fade

```dart
SizedBox(
  width: 250.0,
  child: FadeAnimatedTextKit(
    text: [
      "do IT!",
      "do it RIGHT!!",
      "do it RIGHT NOW!!!"
    ],
    textStyle: TextStyle(
        fontSize: 32.0, 
        fontWeight: FontWeight.bold
    ),
  ),
);
```

## Typer

```dart
SizedBox(
  width: 250.0,
  child: TyperAnimatedTextKit(
    text: [
      "It is not enough to do your best,",
      "you must know what to do,",
      "and then do your best",
      "- W.Edwards Deming",
    ],
    textStyle: TextStyle(
        fontSize: 30.0,
        fontFamily: "Bobbers"
    ),
  ),
);
```
## Typewriter

```dart
SizedBox(
  width: 250.0,
  child: TypewriterAnimatedTextKit(
    text: [
      "Discipline is the best tool",
      "Design first, then code",
      "Do not patch bugs out, rewrite them",
      "Do not test bugs out, design them out",
    ],
    textStyle: TextStyle(
        fontSize: 30.0,
        fontFamily: "Agne"
    ),
  ),
);
```

## Scale

```dart
SizedBox(
  width: 250.0,
  child: ScaleAnimatedTextKit(
    text: [
      "Think",
      "Build",
      "Ship"
      ],
    textStyle: TextStyle(
        fontSize: 70.0,
        fontFamily: "Canterbury"
    ),
  ),
);
```

## Colorize

```dart
SizedBox(
  width: 250.0,
  child: ColorizeAnimatedTextKit(
    text: [
      "Larry Page",
      "Bill Gates",
      "Steve Jobs",
    ],
    textStyle: TextStyle(
        fontSize: 50.0, 
        fontFamily: "Horizon"
    ),
    colors: [
      Colors.purple,
      Colors.blue,
      Colors.yellow,
      Colors.red,
    ],
  ),
);
```
**Note:** `colors` list should contains at least two values, also `ColorizeAnimationTextKit` can be used for flutter `>=0.5.7` which is available in `dev` channel. 

# Bugs or Requests

If you encounter any problems feel free to open an [issue](https://github.com/aagarwal1012/Animated-Text-Kit/issues/new?template=bug_report.md). If you feel the library is missing a feature, please raise a [ticket](https://github.com/aagarwal1012/Animated-Text-Kit/issues/new?template=feature_request.md) on GitHub and I'll look into it. Pull request are also welcome. 

See [Contributing.md](https://github.com/aagarwal1012/Animated-Text-Kit/blob/master/CONTRIBUTING.md).

# License
Animated-Text-Kit is licensed under `MIT license`. View [license](https://github.com/aagarwal1012/Animated-Text-Kit/blob/master/LICENSE).
<br><br>

If you find any problem leave a comment below.
