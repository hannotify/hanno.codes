---
layout: post
co-authors: 
- Julien Lengrand-Lambert
- Jan-Hendrik Kuperus
- Jan Ouwens
title: "Exploring sustainability in tech (without the guilt-trips)"
date: 14-11-2022 20:00:00 +0200
header:
  teaser: /assets/images/blog/2022-11-14-exploring-sustainability-in-tech/IMG_1142.JPG
tags: 
- sustainability
---

Climate change is a thing that affects us all. As developers, we are in a unique position to help do something about this. After all, the whole world runs on computers these days, and computers consume energy. We, developers, program these computers, and the decisions that we make have consequences beyond the business domain we code for. Also, as employed people living in Europe or North-America, we belong to the top 10% of richest people on the planet.

![Panel at Devoxx](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/IMG_1142.JPG) 

(source: Joey Bakker)

But what _can_ we do specifically? It's not often clear. After a discussion on Twitter, we ([Hanno Embregts](https://twitter.com/hannotify), [Julien Lengrand-Lambert](https://twitter.com/jlengrand), [Jan-Hendrik Kuperus](https://twitter.com/jhkuperus) and [Jan Ouwens](https://twitter.com/jqno)) discovered we all have ideas about what we can do. Julien even has a [conference talk](https://www.youtube.com/watch?v=umQby6H50xI) on the subject. We want to know how we can make the most impact, and we know that we can't do it alone.

We figured it was a good idea to host a panel about it: both to get more people involved in the conversation and to learn from them ourselves. The most important rule of such a panel? **No guilt-tripping**! We all do our best, and ultimately, that's all we can do. Participating in a panel like this one can serve as an inspiration to do even better.

![Panel at J-Fall](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/img_1198.jpg) 

(source: Joey Bakker)

We've been fortunate to host this panel twice so far: once as a Birds-of-a-Feather session at Devoxx Belgium, and once at J-Fall in the Netherlands, both in 2022. Both times, we first set the stage by presenting some information about climate change, to get all attendees on the same page. Then, we opened up the floor for discussion, based around three main themes: developer sustainability, infra sustainability, and personal sustainability. Both times, a lively discussion followed in which many great ideas were shared, and the hour just flew by. We were even able to take some learnings from Devoxx with us to the J-Fall panel.

![Panel at Devoxx](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/IMG_1144.JPG) 

(source: Joey Bakker)

Here are a few things that resonated most with the attendees at both conferences, grouped by theme.

## Setting the stage

![Climate change: a timeline](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/semi_rad.jpg) (source: [@semi_rad on Twitter](https://twitter.com/semi_rad/status/1055192820124856320))

- We started with this image and did a show of hands to see where everybody thinks we stand right now. Most were at 'Oops', though several were at 'F°ck' as well. Nobody thought we were in the first two categories.
- Xkcd's [Earth Temperature Timeline](https://xkcd.com/1732/) made an impact. It also happens to be a nice example of what developers can do that no-one else can: Randall Munroe, the comic's creator, has modified his server's configuration so that anyone visiting from a US Government IP is redirected to this particular comic, regardless of which of his 2600+ other comics was requested.

## Developer sustainability

- The most important choice a developer can make, is where they work. Would you want to work for oil companies, airlines, or banks with dubious investment portfolios?
![Green programming languages](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/greenlanguages.png) (source: [Energy Efficiency across Programming Languages: How does Energy, Time and Memory Relate?](https://sites.google.com/view/energy-efficiency-languages))

- Choosing the right programming language matters. However, while the research behind this table is interesting, we found several factors outside the scope of this research that would contribute. For instance, how does the developer workflow of compiling and running tests factor into this? Also, while Python is near the bottom of this list, the really resource-intensive machine learning stuff it's currently popular for, is delegated to heavily optimized C-libraries; Python is merely the glue. Ultimately, no-one was convinced to switch to C or Rust.
- Someone suggested spending more time on design up-front, since spending more time on design might mean less time spent in energy-intensive compile-test-debug loops.
- Turn things off after office hours: the amount of energy wasted on idling test environments is probably huge. One attendee told a story about a school saving €12k a year in power bills by running a script that automatically turns off all computers and peripherals outside office hours.

## Infra sustainability

- It's hard to measure the carbon footprint of an app. The closest approximation seems to be the monthly bill at the cloud provider: the lower it is, the lower your footprint will be as well. However, one attendee suggested [Cloud Carbon Footprint](https://www.cloudcarbonfootprint.org/), which calculates the footprint of your cloud app and suggests ways to improve it.

![A christian news website closed on Sunday](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/sunday.jpg) 
(source: [Reformatorisch Dagblad](https://www.rd.nl), screenshots by Jan Ouwens)

- In addition to shutting down our personal equipment after office hours, we can think about shutting down apps. Does your corporate app really need to run 24/7, or have many [nines of uptime](https://en.wikipedia.org/wiki/High_availability#%22Nines%22)? Some Christian newspapers and web shops in The Netherlands turn their websites off on Sundays. While they do it for religious reasons, we can do it for other reasons, such as reducing energy consumption and reducing infrastructure complexity. When applied where it makes sense, and when communicated properly, your customers and users will probably understand.
- With the prevalence of solar power nowadays, maybe we should rethink our nightly builds, and run them during the day instead. Or as someone pointed out: it's always afternoon _somewhere_; just run the build in a region where the sun is out!
- Either way, make sure to cache your builds. If you let Maven download all of your projects dependencies on every build, you'll undo a lot of effort to make the build more sustainable.
- If possible, take footprint into account when choosing your cloud provider. If you go for one of the Big Three, try to find out what sustainability measures they take in the data center closest to your users. Or you can look at smaller providers that are more sustainable. For instance, [LeafCloud](https://www.leaf.cloud/) in The Netherlands adopted a decentralized approach by placing its servers in various buildings (such as hotels, nursing homes and swimming pools) scattered across the country. The heat generated by those servers is then used directly to heat the building where the servers are located.

## Personal sustainability

![CO2 emissions from passenger transport graph](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/co2-emissions-from-passenger-transport.jpg) 
(source: [European Environment Agency](https://www.eea.europa.eu/media/infographics/co2-emissions-from-passenger-transport/view))

- The way we travel makes a big impact. One attendee at Devoxx received applause when he casually mentioned he had travelled to Antwerp from Switzerland by bicyle! One J-Fall attendee mentioned he traveled by train from another country, but was disappointed how it not only takes longer, but is also about 4 times more expensive, which doesn't make sense.
- There was a discussion about air travel at J-Fall. We, the panelists, don't want to tell people not to do it, and an attendee asked why not, since it's one of the largest contributors to CO2 emission an individual can have. We decided that it's a personal choice that everybody needs to make. If anything, it's something for politicians to discourage and disincentivize. If politicians don't move fast enough, it's easy to call a political party and ask them questions, or even to join them and try to make a difference.
- 80% of CO2 emissions for a device such as a phone, tablet or laptop are spent during production. The best way to scale down is by buying less things, and keeping them longer. Of course it's ok to buy these things, but take quality into account: if you spend a bit more up front, you can make it last longer, which reduces emissions.

## Other

![Panel at J-Fall](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/img_1200.jpg) 

(source: Joey Bakker)

- We need to motivate other people to help, but how can we do this? We could think of only four ways: giving financial incentives, which is something for governments to do; providing a convenient way to change behavior, which is something companies (or individuals within them) can do; or ... guilt-tripping people, despite the main rule of the session. The best way, however, seems to be to inspire.
- For instance, one of the attendees at J-Fall represented a large company that was also a sponsor of the event. She felt inspired to choose only sustainable events, from now on.
- Some people complained that J-Fall itself could make some improvements to become more sustainable. The lunch contained a lot of packaging and foods that not everybody wanted to eat, leading to a lot of waste. It turns out that the lunch break, like every other session, could be rated and commented on, which we all subsequently did. Hopefully this will lead to some improvements in this area next year!

## Conclusion

![Panel at Devoxx](/assets/images/blog/2022-11-14-exploring-sustainability-in-tech/IMG_1149.JPG) 

(source: Joey Bakker)

All these ideas are great, but it's still interesting to see some numbers: what's the impact of working for an oil company versus not running builds at night, for instance? For now, we leave those questions unanswered, as these sessions were mainly intended to raise awareness and get people involved. We'll try to work on these numbers in the future.

After the sessions, most of the attendees had a takeaway to look into or implement right away. I, for one, have been taking the train instead of the car more often. In that sense, the sessions have been great at inspiring us to do better, rather than guilt-tripping us into it!

> Cross-posted with permission. The original post is at <https://blog.yoink.nl/posts/2022/11/14/exploring-sustainability-in-tech.html>