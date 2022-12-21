Coding fast vs fast code
========================

Code that executes quickly is part of coding efficiently, but there's a trade-off: Writing fast code is typically time-consuming.

<a href="https://xkcd.com/1205">
<img src="https://imgs.xkcd.com/comics/is_it_worth_the_time_2x.png" title="Don't forget the time you spend finding the chart to look up what you save. And the time spent reading this reminder about the time spent. And the time trying to figure out if either of those actually make sense. Remember, every second counts toward your life total, including these right now." alt="Is It Worth the Time?" />
</a>

The comic ([xkcd 1205 <img src="https://upload.wikimedia.org/wikipedia/commons/6/64/Icon_External_Link.png" width="13px"/>](https://xkcd.com/1205)) shows the maximum amount of time you can spend optimizng a task before it becomes inefficient by how much time you are saving and how often the task is run, over a 5-year horizon.

- If you do a task once a year and you will only save one minute, then the most time you should spend on making it faster is 5 minutes.
- But if you do it several times a day, say 5/day, then it might make sense to spend up to 6 days making it faster.

In practical coding situations the trade-off is seldom so clear-cut, but the exact idea does apply. It makes little sense to spend a long time optimizing code you will only run a handful of times. By contrast, if you expect you will have to run it often then making it run faster might well be worth the investment.
