---
layout: post
title:  'BuyMeACoffee silently dropped support for many countries, and nobody cares'
date:   2024-08-08
description: "Silent changes in payment methods on big creator funding platforms raise some unpleasant questions."
categories: war rant
comments: true
image: https://zverok.space/img/2024-08-08-bmac/cover.png
---

**Silent changes in payment methods on big creator funding platforms raise some unpleasant questions.**

## What happened

Recently, many Ukrainian creators have reported problems with payouts from [BuyMeACoffee](https://buymeacoffee.com/), a creator funding/crowdfunding platform.

At first, the reported support answers were typical corporative “we are sorry that we don’t care,” citing “compliance” and “policy updates.”

<blockquote class="twitter-tweet" data-conversation="none"><p lang="uk" dir="ltr">В оригіналі, щоб вестерни читали які там є нікчеми <a href="https://t.co/JxoGgamYLq">pic.twitter.com/JxoGgamYLq</a></p>&mdash; Karå ꑭ🇺🇦 (@BzickOff) <a href="https://twitter.com/BzickOff/status/1819341225024983310?ref_src=twsrc%5Etfw">August 2, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Those reports were widely shared, and our predominantly feeling was “shocked but not surprised”: it is not the first time when a big platform has decided that they are too busy to try to distinguish non-occupied parts of a large country with a 30+m population from occupied parts, and decided they don’t care. Just recently, for example, the Mercury payment system **[just blacklisted the entirety of Ukraine](https://x.com/United24media/status/1815707715307479244)** alongside Iran and North Korea.

But in a few days, it turned out (again, from support messages shared on Twitter) that BuyMeACoffee **just dropped support for Payoneer** (which works in Ukraine), leaving Stripe (which doesn’t) as the **only payout method**.

<blockquote class="twitter-tweet" data-conversation="none"><p lang="uk" dir="ltr">4. Бо бай мі кохве мала в гузно своїх споживачів та закрила можливість виводити кошти напряму на карту через Wise, а також через Payoneer. Єдино можливий метод виводу стає сервіс Stripe який не доступний в Україні та до 14 серпня всі автори зможуть ввостаннє вивести свої кошти. <a href="https://t.co/XIQdNHVllu">pic.twitter.com/XIQdNHVllu</a></p>&mdash; AdrianZP (@AdrianZPcity) <a href="https://twitter.com/AdrianZPcity/status/1821252119875436595?ref_src=twsrc%5Etfw">August 7, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Did they, though?

Currently BuyMeACoffee’s [support page](https://help.buymeacoffee.com/en/articles/6258038-supported-countries-for-payouts-on-buy-me-a-coffee) simply states:

> Buy Me a Coffee supports payouts to creators in several countries, facilitated through our payment provider, Stripe.

According to the Internet Archive, in [February](https://web.archive.org/web/20240229183241/https://help.buymeacoffee.com/en/collections/1907909-payments), there was another link to a [page](https://web.archive.org/web/20231028113814/https://help.buymeacoffee.com/en/articles/6258038-payment-supported-countries) that listed Stripe and Payoneer, while [May’s snapshot](https://web.archive.org/web/20240521195730/https://help.buymeacoffee.com/en/collections/1907909-payments) already has only Stripe-related links.

So, the change _in the documentation_ happened somewhere between February (that’s when I personally used it for the last time) and May: Internet Archive doesn’t provide other snapshots between those two dates.

So, at least the support’s private answers were aligned with the documentation.

**At the same time, there are a couple of funny facts about this deprecation.**

**First,**  I know for a fact that some Ukrainian creators were receiving payouts via Payoneer somewhere around ~1 month ago; I even saw a credible claim about last week! (I just sent mine, which trickled from a moderately popular blog, will see whether it will come through; it is funny if it will. At least my configured Payoneer payout method is currently active in the BMaC dashboard.)

But more importantly, **there was NO communication about the change**. You wouldn’t find anything on [their Twitter](https://x.com/buymeacoffee) or in the [public changelog](https://building.buymeacoffee.com/changelog/) (screenshotted at 2024-08-08 11:05:30 GMT+0300—just in case something will magically appear there retroactively):

![](/img/2024-08-08-bmac/image00.png)
![](/img/2024-08-08-bmac/image01.png)
![](/img/2024-08-08-bmac/image02.png)
![](/img/2024-08-08-bmac/image03.png)

As you can see, **there is nothing about payout updates** after the [Nov'23 post](https://building.buymeacoffee.com/changelog/improved-payout-experience), which mentioned Payoneer and Wise as available options alongside Stripe.

I assure you that **there was no email communication to the creators either**.

## How bad is this?

The difference is public lists of supported countries between [Payoneer](https://www.payoneer.com/resources/global-payment-capabilities/) and [Stripe](https://help.buymeacoffee.com/en/articles/6258038-supported-countries-for-payouts-on-buy-me-a-coffee) are **95 (ninety-five)** countries and territories. I am not sure that all 95 of those don’t have working Stripe (say, maybe the Faroe Islands just listed on Payoneer separately while being actually served by Stripe via Danish banks?), but it seems to be _a lot_ anyway.

I’d like to know what the experience of BMaC users from those countries is like. But what I know for sure is that in Ukraine, a lot of people use the platform as a source of income (sometimes even a primary one). My subscriptions, for example, include but are not limited to:

* A singer-songwriter turned paramedic who uses BMaC donations to record her songs on short vacations between duties;
* A writer and culturology scholar turned soldier who funds his studies about Ukrainian culture and history;
* A girl whose two brothers have fallen on the frontlines, and she funds her absolutely unique book club to connect to people and share reading experience with them;
* A small business owner from Kharkiv who uses a “build in public” approach for a small coffee shop (and recently got mobilized into Armed Forces);
* ...and so on.

None of them have BMaC as their primary income, and still, it is a sign of support from their peers and the possibility to continue doing what’s important to them.

**There is no way for any of us to receive the money accumulated on BMaC** anymore. What looks like “technical” change for some American is realistically **prohibitive** for those not privileged enough to live in Stripe-supported countries. The money is still “there”, so probably no lawyer can say BMaC has “stolen them”—you just can’t neither receive “your” money nor, at least, give them back to those who sent them.

Note that some BMaC accounts have a lot of supporters, and many of those use “yearly” payments as a sign of their support—and all of it is currently in some technological limbo.

Now, I am just a mere developer, and I don’t know much about payout regulations and legal facts. I _might_ charitably assume there was a good reason for dropping Payoneer support (maybe a business one, or maybe it is actually related to some government regulations). The fact itself is extremely distressing for many, but hey, that’s their business, what do we know.

**But the way this policy was implemented—no prior notice, no choice, no explicit communication of the reasons and possibilities—is absolutely fascinating, to say the least.** And, honestly, support’s evasive behavior (and inconsistent reports about “some Payoneer payouts might still work”) just adds insult to injury.

I am not sure that any service that handles people’s money as their _primary_ activity, and that does it this way, can be ever trusted. Just sayin’.

Have a good day, and have a screenshot of the latest message from BuyMeACoffee’s ever-positive Twitter!

![](/img/2024-08-08-bmac/image04.png)

Such a nice vibe! _(Neither their primary account nor [the founder’s personal one](https://x.com/jijosunny), which both have a history of replying to all relevant tweets, are currently giving any signs they are aware of the situation.)_

PS: As a side note, [here is a story](https://en.ain.ua/2023/04/10/what-is-wrong-with-patreon-and-why-ukrainians-urge-canceling-it/) about why Patreon is not an option for Ukrainians. And [here](https://www.patreon.com/wargonzoo) is russian war correspondent’s Patreon account, mentioned in the story, still proudly present on the platform (though seemingly dormant).
