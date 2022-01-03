---
layout: page
title: "Speaking"
---

Hi! I'm Robin. I enjoy speaking. I mostly give talks at local meetups, but have been lucky to have been invited to a couple of conferences. My usual speaking topics cover [testing](/talks/2016/06/property-based-testing/), [static typing](/talks/2016/07/things-you-cant-do-wdc16/), and [functional programming](/talks/2015/09/composition-and-pipelines/), but have previously switched it up with [ops](https://speakerdeck.com/damncabbage/the-testing-monitoring-continuum-devops-sydney-2015), [security](https://speakerdeck.com/damncabbage/xss-an-introduction-and-demonstration-2009) and [race conditions](https://speakerdeck.com/damncabbage/databases-and-race-conditions-sydphp-2014).

The full list is on [the front page](/#talks), but here are some of my more recent recorded talks:

<ul class="talks-showcase">
  <li>
    <div class="talk--thumbnail">
      <div class="embedded-iframe">
        <div class="inner">
          <iframe src="https://player.vimeo.com/video/194125263" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
        </div>
      </div>
    </div>
    <div class="talk--text">
      <h2><a href="/talks/2016/07/things-you-cant-do-wdc16/">
        The Things You Can't Do (Web Directions Code 2016)
      </a></h2>
      <p>Explores what we gain if we strive for less of "flexibility" and "power" in parts of our code. Walks through some different intentionally-limited things, like <code>forEach</code> instead of <code>for</code>, <code>let</code> vs <code>var</code>, immutable data, and then digs into static typing with <a href="http://flowtype.org">Flow</a> to show how limiting potential inputs and results from functions make our programs more predictable.</p>
    </div>
  </li>

  <li>
    <div class="talk--thumbnail">
      <div class="embedded-iframe">
        <div class="inner">
          <iframe src="https://www.youtube.com/embed/_OXrdIRmdJc" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
        </div>
      </div>
    </div>
    <div class="talk--text">
      <h2><a href="/talks/2016/06/property-based-testing/">
        Property-Based Testing (CampJS, 2016)
      </a></h2>
      <p>Introduces an approach to writing tests as "properties": each property is a rule that you write; you then generate random data of a particular shape, and run that property over and over with that data to make sure that rule still holds. This talk also includes some techniques for writing properties, as well as some real bugs caught by property-based testing tools.</p>
    </div>
  </li>

  <li>
    <div class="talk--thumbnail">
      <div class="embedded-iframe">
        <div class="inner">
          <iframe src="https://player.vimeo.com/video/138642247" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
        </div>
      </div>
    </div>
    <div class="talk--text">
      <h2><a href="/talks/2015/09/composition-and-pipelines/">
        Composition and Pipelines (Sydney Ruby Meetup, 2015)
      </a></h2>
      <p>Plays with function composition, using Ruby's <code>Proc</code>, and introduces some concepts like "Associativity" by example, with their practical benefits.</p>
    </div>
  </li>
</ul>
