---
title: Welcome to my blog
---

# Outline
In you language of choice,
you likely have a function that outputs (psudo-)random numbers from 0 to 1.
This function either outputs on the interval $[0, 1)$ or $(0, 1)$.
In other words,
a range from 0 up to but not including 1 or that same range but excluding 0.
This post is about how to use that function without biasing your results.
There post has a demo with graphs at the end.
The demo is written in Octave,
so the function's name is `rand` throughout this post.

A 10 sided die was chosen to exemplify the math below.
But, you could imagine a 6 (or whatever) sided die instead if you like.
Just replace every instance of 10 with 6 in you mind.


# Derivation
If you want an evenly distributed random number from 1-10...
```
round(rand*10) #No.
```
... is a good place to start.
However, this gets you numbers in $[0, 9]$.
The lowest value the die should give is 1 and 10 is absent.
Lets try...
```
round(rand*10)+1 #No.
```
This fixed the problem but there is still a less obvious problem.
When you round, only the random numbers less than .05 will return 1.
Also, random numbers greater than or equal to .95 will return 11.
But, the other numbers are easier to get because they can be gotten from rounding up <b>OR</b> rounding down.

Therefore, that interval i gave earlier, $[0, 9]$, is actually $[0, 10]$
due to this troublesome rounding!
This lie was for illustrative purposes.
Did you fall for it?
Randomness is tricky.
Anyway, you could fix the rounding by doing modulo 10.
```
(round(rand*10)%10)+1 #Yes.
```
The modulo can return 0, so it is important to add 1 at the end.

But, instead of rounding you could truncate instead.
This simplifies the calculation.
```
int(rand*10)+1 #Yes, but moreso.
```

Great.
Remember, the +1 is to avoid getting 0 because a die doesn't have 0.
Which would give $[0, 9]$.
This shows how you could shift the window of results around.
So, in general this is how to do it:
```
sides = maximum - minimum + 1
int(rand*sides)+minimum #Generally.
```

${\Large HOWEVER}$ this is also very slightly biased, but not in a way that you would normally need to care about.
The problem arises from trying to redistribute numbers into a range that doesn't evenly fit them.
10 is nothing compared to the amount of options `rand` will output, so the bias is negligable.
My next post will be about that.

# What's Up With 1 In The Interval
Note that $\texttt{rand}*10<10$ because `rand` doesn't return 1.
If the interval did allow 1, then you may randomly get 10 before the addition.
This would result in 11.
It would be the only number that would do that.
It is best to exclude it for that reason.

# What's Up With 0 In The Interval
Feel free to skip to the [demo](https://github.com/Noventrice/Deriving-Randomness/edit/my-pages/index.md#demo) at the end if you don't understand this section.
Including 0 gives another option in the window of valid options.
So, why would it not be included?

Each individual number has a low probability of happening.
For 0, that is the machine epsilon of the floating point.
With 64-bit doubles, the chance of getting 0 is about 2.22e-16.
This is machine dependant but your machine is likely using 64-bit doubles.
Commercial 32-bit machines were phased out about a decade ago.
Anyway, 2.22e-16 is considered random enough.
The loss of granularity is negligible.
If you are curious you can view the machine epsilon in Octave with `eps`.
```
eps            #double precision
eps(single(1)) #single precision
```
Octave only has double and single float types.
If you want to see more, Wikipedia has a nice [chart](https://en.wikipedia.org/wiki/Machine_epsilon#Values_for_standard_hardware_arithmetics).

A probability of 2.22e-16 is hard to comprehend.
Matt Parker made a great video explaining extremely unlikely events.
Here is a [link](https://www.youtube.com/watch?v=8Ko3TdPy0TU&t=28m54s) to the relavant part of the video.
The luckiest thing he could find was a craps game with a likelihood of 1.4e-12.<sup>[timestamp](https://www.youtube.com/watch?v=8Ko3TdPy0TU&t=24m10s)</sup>
That's much more likely than 2.22e-16.
Matt Parker's probability unit of 3e-19 is smaller.
So, if <b>everyone</b> was doing an event every second with a probability of `eps`,
then it should happen at least once in a century.

Matt's unit is clearly not enough to cover this scenario.
Firstly, it isn't taking the speed of computers into account.
Secondly, if a program doesn't run for a century, the program that replaces it might cumulate to that.

#### How Likely Is It Then
Modern computers run at nanosecond precision.
A nanosecond is a billionth of a second AKA 1e-9 seconds.
My computer takes 36 nanoseconds on average to make a random 10 sided dice roll.
Here is some C code that I used to test that:
```
#include <stdio.h>
#include <stdlib.h> //rand
#include <time.h> //timespec_get
#include <limits.h> //LLONG_INT


int main(){
	struct timespec now, then;
	srand(time(NULL));
	volatile int x; //A random number [1, 10].
	(void)x;
	long diff; //Time diff in nanoseconds.
	long long acc=0; //numerator for average.
	long n = 1e7;

	for(int i=1; i<=n; ++i){
		timespec_get(&then, TIME_UTC);
		x = rand()%10+1; //rand returns integers in C, not floats. There are fewer outcomes.
		timespec_get(&now, TIME_UTC);
		diff = (1e9*now.tv_sec+now.tv_nsec) - (1e9*then.tv_sec+then.tv_nsec);
		if(acc + diff > LLONG_MAX){
			puts("Overflow.");
			return 1;
		}
		acc += diff;
	}

	printf("Took %ldns on average.\n", (long)acc/n);
}
```

36 nanoseconds means at a minimum a trial can take 3e-8 seconds (on my computer on average).
How many seconds would it take to get a 0?
$$3 \times 10^{-8} ~ \text{seconds} * 2.22\times10^{16} = 6.66\times10^8 ~ \text{seconds}$$
A year is only 3.1e7 seconds.
So, if you let your computer generate random numbers as fast as possible you are likely to have gotten a 0 within a year (where available).
The same applies to any other specific number.

It is still a one in two hundred quintillion chance, though.
You likely don't need that much granularity.
If you do you probably aren't using psudo-random number generation anyway.

For more on this subject check out
[this](https://stackoverflow.com/a/29815682)
Stack Overflow page.

# Demo
```
n=10; %The number of sides on the die.
v=zeros(1,n); %The array that holds rounded values.
u=zeros(1,n); %The array that holds truncated values.
for i=1:100_00%0 %Feel free to adjust the amount of events.
	++v(mod(round(rand(1)*n), n)+1);
	++u(floor(rand(1)*n)+1);
endfor

figure(), clf;
bar(v, 'histc');
figure(), clf;
bar(u, 'histc');
```

# Partial Credit
This post was inspired by [this video](https://www.youtube.com/watch?v=DJCp6k1ts3g&t=202). In the video the speaker says rounding doesn't work but the Matlab/Octave demo says otherwise. Perhaps now you can intuit what he did wrong.
