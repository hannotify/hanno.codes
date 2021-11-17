---
layout: post
author: Hanno Embregts
title: Explaining software development to a class of 6-year olds
date: 17-11-2021 20:07:21 +0200
image: images/blog/bee-bot.jpg

tags: 
- teaching
- software-development
---

My daughter is 6 years old, and she and her classmates are currently learning about various professions. Her teacher asked the kids' parents whether they would want come into the classroom and explain their profession in 15 minutes or so. I really liked the idea of inspiring these bright young minds with some fun stories about software development, so I applied and prepared a lesson. Which turned out to be a lot harder than I thought it would be.

## Keep a primary school teacher around

"Keep it simple" I thought to myself. "These kids are 6 years old, they don't an in-depth explanation of how Java handles cryptography. Just enough content to spark their interest in software development will do." But in the same amount of time that it would have taken me to spell 'YAGNI', words like 'algorithm' and 'directive' had suddenly appeared in the lesson plan I was drafting.

Luckily, I keep a primary school teacher around to help me in cases like this. 🙂 When my wife saw the lesson plan, she first had a good laugh. Then she seemed to see it fit to insult my teaching skills. "These kids are 6 years old!" she said. "They don't need words like 'algorithm'! Your lesson plan should be WAY simpler!"

She was right, of course. I needed to replace the technical terms by simpler alternatives, so 'instructions' instead of 'algorithm'. On top of that I had to use examples that the children could relate to. So perhaps it wasn't the best idea to use 'blockchain consensus' as an example of an algorithm, and I probably had to use 'navigation instructions' instead - most 6-year olds have seen satnavs in action, right?

## What is a computer?

So armed with my lesson-plan-approved-by-the-wife I showed up in my daughter's classroom and delivered the lesson. "Who can tell me what a computer is?" I asked the class. "Screens! Games! Chromebooks!" the 6-year-olds shouted. I was satisfied with the answers; clearly some solid foundations of IT knowledge were already present.

So I continued by explaining that computers need *instructions* to perform their tasks, like an itinerary contains specific instructions to get you to your destination. "Tell me about it. My mum wouldn't get *anywhere* if it wasn't for her satnav" sighed a blonde-haired girl to my left. Clearly I had struck a nerve!

## What are instructions?

Then I showed the kids how a specific set of instructions can lead to the desired results, by using a Beebot. A Beebot is a bee-shaped robot, that can store a set of instructions and then execute them. Its available instructions are:

* Move 15 centimeters forward;
* Move 15 centimeters backward;
* Turn 90 degrees to the left;
* Turn 90 degrees to the right.

Then I set up a maze with 15-centimeter-sized squares, using cards with different kinds of fruit to serve as destinations. Then I asked the kids to program a route to the 'apple' card, or to the 'strawberry' card, for example. They picked it up really quickly.

![A Beebot](/images/blog/bee-bot.jpg)
> Image by <a href="https://pixabay.com/users/noratheone-7789308/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=4096410">noratheone</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=4096410">Pixabay</a>

## 1000 teacher photos!

"So what do you actually do at work - do you play with Beebots all day?" the teacher asked. "I wish!" I replied, and then I showed them an IDE (Visual Studio Code, in this case) and a Jekyll-based website that displayed the name of the teacher and a photo of her (she was very honoured, by the way). Then I asked the class: "What would happen if I changed this number '1' in the code to the number '10'"? They quickly found out that by changing it to '10' the website would contain ten identical teacher photos instead of just the one.

"Now do a hundred!" a boy shouted. "No, a thousand!" yelled another. Naturally I quickly implemented this very important requirement that held quite a bit of business value, at least according to this class of product owners. "A thousand teachers?" sighed a girl to my right. "And here I was, thinking that having a single teacher was already a bit much!"

## Did I actually inspire them?

After the lesson had finished the teacher and I evaluated the lesson. "Thanks a lot", she said. "It was a very fun lesson about computers, and the hands-on experience was really appreciated by the kids." "You're welcome", I replied. "So do you think some of them want to become software developers now?" "I'm not sure", she said. "But at the very least they'll probably tell their parents tonight that all you do at work is sending bee robots to get fruit and generating lots of photos of primary school teachers." 

Well, if I inspired the kids to tell their parents exactly *that*, then the parents probably had a good laugh about it. And that - frankly - is all I could have asked for.
