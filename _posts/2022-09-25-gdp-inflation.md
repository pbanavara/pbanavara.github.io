---
layout: post
title: GDP and Inflation
---

### Understanding GDP

On Friday, I was lucky enough to listen to Prof.Prasanna Tantri from ISB. While I didn't expect a great deal of learning, given the predisposition us technologists have against economics, business and the business school as such, I had to swallow my thoughts and pride and listen to this man for 3 hours without even looking at my watch.

I am going to distill some of those learnings here and I hope I haven't lost a great deal of content in the last couple of days.

GDP - Let's try to understand GDP, which is defined as Gross Domestic Product. If we break this down, gross means total, domestic is obvious confined to a geography, and product is well, just the output of that geography.

Where it gets interesting is, GDP is not just products, it includes services. So to summarize

* GDP = Sum ( Goods and Services ) produced in a country in a given year.

How do we arrive at goods and services, what does this actually mean in real terms.

Goods and Services = Let's say everything the population produces, in very simplistic layman terms. How do you track production and is production a good metric.
Well, no really - what gets produced has to get consumed otherwise it's useless, so consumption is a better metric. 

There was recent news that India's GDP grew at 16% in the April - June Quarter and this was a very positive and welcome news. At the same time, one of the outside think tanks pegged India's GDP growth at a paltry 3%. How could this be possible ? It's possible because of 2 aspects of GDP

* Nominal GDP - Total value of goods and services in today's monetary value, without taking inflation into account
* Real GDP - Total value of goods and services taking inflation into account.

India measures GDP growth in nominal GDP from previous year to this like any company does. So compared to Apri-June 2021 we grew 16%.
The outside think tank measures the growth from previous quarter. So from 2022 Jan - Mar. 

Both these reports are correct and we should just pay attention to the details without getting taken away by sensationalism. 

Now, how to calculate GDP. Conusmption is obviously a contributing factor.

So we have
``` GDP = Consumption ```

That's just not it. You need capital to produce and also capital to consume. Let's call this investment. So now we have
``` GDP = consumption + investment ```

Well then there is government expenditure. Salaries for government staff. Interestingly, subsidies don't come under government expenditure because the subsidy
doesn't produce any output. It's just taking someone's produce and handing it off to someone else, well in very simplistic terms. 

Then there is exports which you deduct and add an additional error correction from the government side. That gives

``` GDP = consumption + investment + Govt Expenditure - exports + error"```

The real breakdown is something like this
``` GDP(100%) = 59%(C) + 30%(I) + 11%(GE) - 5%(E) + 5%(Error)```

What's interesting is that we save 30% of what we produce. This is super important because this investment serves the growth next year. Growth comes from 2 aspects
* Investment
* Productivity

This brings up an interesting aspect of developed v/s developing countries. A developed country consumes 90% of what they produce and invest something like 9-10% or even less.
With just 10% available for investment in the next year they need to fuel growth purely by productivity. 

So how do you measure productivity, just by measuring efficiency. [ICOR](https://www.investopedia.com/terms/i/icor.asp#:~:text=The%20incremental%20capital%20output%20ratio%20(ICOR)%20explains%20the%20relationship%20between,the%20next%20unit%20of%20production.) is the metric used to measure productivity.

Developed countries have an ICOR of around 3 which means they use far less resources and investment to produce a unit of output as compared to a country like India where the ICOR is somewhere around 5.

There is a floor for ICOR, you can't really get better than 2 or 2.5 and now you have to fuel growth purely by producitivity. My view here is that - this would have been fine with a growing young population, but developed countries are facing a declining working population. It's not that the population growth is declining, but the number of job seekers are also declining. 

India, is poised to grow at 6-7% even with no improvements in ICOR simply because of the amount we save and reinvest. Now if ICOR improves to 4 or 3 ( this is brought about by improvements in infrastructure, logistics, ways of doing business etc ) we can easily take the growth to 9-11% consistently, whereas the developed countries will grow at best by 2-4%.

It's not hyperbole, when policy makers postulate that India will be a $5T economy soon
#### Summary:

While GDP seems like a number that is not representative of the ground realities, and there are these confusions around nominal v/s real GDP, there isn't a better alternative to measure economic growth. So we gotta live with what we have but pay close attention to real GDP and not get carried away by nominal GDP growth.


### Understanding Inflation

Inflation is the measure of rate of increase of goods and services, mostly goods. We all know this. How is inflation measured. When we say inflation is 8% this month what exactly does it mean.

Rate of increase means that you need a baseline rate to measure the increase against. The baseline is of course chosen by the country officials Fed, govt etc. Some countries like India choose a fixed baseline. For us it is the FY 2011-12, as of now. US has a different baseline - a moving average based baseline. So they penalize older years as compared to newer ones.

What matters is that there be a baseline and there be a correct baseline. Why can't we have a baseline in India that is at 1947 ? Any guesses ? Well, most of the goods we use today were not present in 1947 and most of the goods that were prevalent in 1947 are obsolete now. I believe the correct measure is
``` current year - 7```. Well for now it is what it is.

So once you have a baseline, you need some way of arriving at one number for all goods. This is super tricky to obtain. So they resort to what is know as common basket of goods and create an index based on that.

``` baseline index = Normalized(basket of goods) = 100 ```

You then use this scale to compare against current value of the index. Then depending on whether you are reporting month on month inflation, year or year or whatever you use the appropriate values w.r.t this index.

More than the inflation formula, what's important is to understand what causes inflation.

The crypto enthusiasts have been saying this for eons, yes it's the government printing money. This is the sole cause of inflation because, if the government doesn't print extra money whatever price increase happened will balance out with the equivalent price decrease in other goods.

For instance, let's say wheat supply is down, wheat prices go up, there's no more money in the system, naturally the price of something else has to come down, in this case it may be milk or who knows what. This is perhaps why there was a limited supply of bitcoin. In the utopian world of only bitcoin, there would be no inflation. 

WHhen there is high inflation, government prints more money so inflation reduces and we are all good.
When there is low inflation, government sucks up money, things become more expensive and at some point we are all good.

Not really, what's important here is to understand what is known as expected and unexpected inflation.

Expected inflation is what people are used to, for instance in India we are used to 8-10% inflation. It's baked into our psyche. In the US it is around 2-3% that's baked into the payche. As long as inflation stays at these levels, world will run smoothly.

Problem arises when inflation changes unexpectedly beyond these levels, which is what is happening in the developed world now. This causes somewhat of a compounding effect combined with the chicken and egg problem.

If the inflation increases abruptly and government/Fed doen't reduce money supply proportionally,
People start losing faith in Fed/government and they reset the expectation to current inflation levels.

Now Fed/Government has to play catch up and if they don't catch up in one iteration, this cycle will continue
and lead to extreme inflation as seen in countries like Venezuela etc.

#### What is increasing / decreasing money supply ?
This is also known as raising/reducing rates, printing money and other layman terms. This is what happens.

Central banks say 'Our target rate this quarter is x%'
Depending on whether x is greater or lower than the current rate the following will happen

The traders at central banks immediately hit the market on the trading instrument. Imagine the fire sale scene in Margin Call.  

``` if new target > current rate:
	    Fed traders will Buy bonds at the target rate
```

It's that simple. But who the hell will sell the bonds - Everyone and their mother. Banks, insurance companies, mutual funds, debt funds everyone.

``` if new target < current rate:
       Fed traders will start unloading bonds 
```

Who will buy - again the same folks who sold when Fed was buying.

This is only possible because Fed is the big bully in town. There is no greater authority. So when they sell, you better buy and when they buy you better sell.

#### Is inflation necessarily good or bad ?
Depends on who you are asking. For the citizens, it's terrible. For the government it's great. It's breat because they can now basically give the illusion of life becoming better while in reality life is getting worse. This happens because we are all fixated on money and not the real goods.

For simplicity sake, 

``` inflation = 5%
	salary = $1000
	expected hike = 5%
	new salary = $1050.
	Employee is happy
	
	Cost of a house before inflation - $1000
	new cost of the house - $2000
```
This is how governments can actually give the illusion of good. Of course this can't go on forever as I've depicted earlier.

It is really important to understand inflation because the only power central banks have is to reduce/increase the money supply. That's all that matters for managing inflation.Granted wars can cause a shortage in supply but that will largely be maintained, except in the case of oil. If the oil supply stops, literally the world will come to a stand still. 

Inflation and GDP are very much correlated since nominal GDP is a function of inflation and it's the psychological benchmark of economic growth of a country.

#### Conclusion

I hope I was able to distill this without forgetting much. I personally became more bullish on India after listening to this lecture, simply because we have 3 ingredients for
growth that no other country has

* A growing working population - 500M people under the age of 27
* Entrepreneurial spirit - (This is subjective, but it's there nonetheless)
* The 30% savings investment rate

The fourth one which is also probably going to work in our favour

* Strategic efforts by the government to reduce ICOR. If we hit an ICOR of 3 - then we are literally unstoppable. 


















