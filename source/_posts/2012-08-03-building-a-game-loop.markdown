---
layout: post
title: "Building a Game Loop"
date: 2012-08-03 17:54
comments: true
sharing: true
categories: [JavaScript, HTML5, Canvas, Games]
author: Sebastian Borrazas
description: How to build a simple Game loop from scratch using plain simple JavaScript?
---

Since this is my first codepath post, I wanted to start talking about something we all love: games and the web.

Lately I've been digging into HTML5 game development, with all the fun technologies it involved working with.
Often, I try to make my own simple versions of frameworks, toolkits or game engines, for fun, to learn and for simplicity matters.

The first big problem I ran into when trying to make a simple version of the old pacman game was the game loop.
How are the ghosts supposed to move by themselves? How and when should I notify the canvas that the game changed?

I finally came up with what I believe is an easy and simple way to run a game loop, which I've used ever since for making simplistic games.

<!-- more -->

This is its shortened version:

``` javascript Game Loop
(function (pacman) {

  var GAME_LOOP_MS = 30;

  var Game = function () {
    this.processing = false;
    this._interval = setInterval(function () { this.run(); }.bind(this), GAME_LOOP_MS);
  };

  Game.prototype.doLogic = function (interval) {
    // Check if player died, move ghosts, update score, etc
  };

  Game.prototype.draw = function (interval) {
    // Draw on canvas. (You probably want to delegate this)
  };

  Game.prototype.run = function () {
    var intervalTime;

    if (!this.processing) {
      intervalTime = GAME_LOOP_MS * (this.skippedIntervals + 1);

      this.processing = true;
      this.doLogic(intervalTime);
      this.draw(intervalTime);

      // Resetting data
      this.skippedIntervals = 0;
      this.processing = false;
    }
    else {
      this.skippedIntervals += 1;
    }
  };

  pacman.Game = Game;

})(GLOBAL_PACMAN);
```

This implementation won't work for hardcore games, but it does the job with games that need to be updated in really short intervals, and it's really simple to understand.
There are also many ways to improve efficiency by using the browsers RequestAnimationFrame function.
Thoughts?


