# EDDY'S CYBER GARAGE

With love, for my son Mars Z. Dong.

<br><br>

### Table of Content

[APOLLO MUSIC IMPROVISING](#apollo-music-improvising)

[LEAST SQURE (FOUNDATION FOR AI) FROM SCRATCH](#least-square-from-scratch)

[SHANGHAI PANDEMIC SIMULATION](#shanghai-pandemic-simulation)

[SOLAR SYSTEM SIMULATION](#solar-simulation-for-my-son-mars)

[GREEDY WORMS](#greedy-worms-for-my-son-mars)

[REVERSI](#reversi)

<br><br>

### APOLLO MUSIC IMPROVISING

A project using rule based plus data driven methods to help composers to study and improvise music, including scales, chords, rhythms and more. It features a very easy to use yet very powerful browser-based pianoroll, with some built-in AI tools to assist music analysis and creation. 

The project is 100% in pure Javascript.

It's a long-term one and currently still in early stage.

Some screenshots showing the recent progress:

![img](img/apollo/2.png)
![img](img/apollo/1.png)

[Listen to the endless Bach Chorales! (Press Enter to start after loading)](https://eddydong.github.io/apollo)

![img](img/apollo/3.png)


### LEAST SQURE (FOUNDATION FOR AI) FROM SCRATCH

![img](img/neuron_research/leastsq.png)

Implementation of the Gradient Descend algorithm for estimating a simple classical linear model, the Least Square, with pure python & numpy. Can be upgraded to a fully functional neuron network framework.

[View The Source Code](https://github.com/eddydong/PycharmProjects/blob/main/pythonProject/leastsq_test.py)


[GO BACK TO TOP](#eddys-cyber-garage)

### SHANGHAI PANDEMIC SIMULATION

Hey guys this is Eddy. I’m been working from home in Shanghai for 6 weeks in a row and now trapped in the lockdown. Like everyone in this city, I want to know more about what’s going on, and what it could be moving forward. So I devoted my weekend to write [this simulator](https://eddydong.github.io/shanghai).

![img](img/shanghai/0.jpeg)

First, my goals here are simple: trying to estimate the duration of this pandemic, the death toll of the BA2 infection and of other causes related to the lockdown, and the accumulated cases when it’s all over. And I set up 3 imaginary scenarios to analysis the impact of different interference methodologies.

Then I thought of quite a few hypotheses for the simulation, and when you translate them into mathematical language, you get the pandemic models. And when you assemble the models with computer programming, you will get the simulator, and then you can play with it and test your ideas. 

- You got a large city with 26M people living in 20K communities.
- People live in a certain place but they travels around on daily basis, following certain pattern like skewing to their residence etc.
- One has a chance to get contracted when visiting a community in which positive cases were reported within a certain number of days.
- A positive case will be quarantined, but sometimes with a lag of days.
- Vaccination can lower the transmission & the death rate.
- Virus can survive outside its host, human for instance, for certain days.
- Lockdown will be throttled in 3-phase, decided by recency of the latest positive case. Different phases have different level of mobility limitation.
- People have certain chance to die from COVID infection.
- People who have contracted the virus got immunity for years.
- People have certain chance to die from severe disease other than COVID due to limitation of transportation/logistics, especially in the top phase lockdown.
- Imported cases from outside Shanghai not considered in the simulation.

From open sources I collected some data to describe the behavior of the virus, like how long it can survive in and outside its host, infection rate and death rate etc, and the behavior of people, like how frequently they will travel around in the city under different scenarios and where are they going etc, and some nature of the city, like the population and the amount of communities etc, and the lockdown measures, like how much proportion of the cases will be quarantined and how swiftly this process is, how strict the restrictions are for transportation & logistics for each of the 3-phase lockdown levels under each of the 3 scenarios. One thing to point out is the death rates. From what we’ve observed in the first 40 days of the Shanghai pandemic, actual death rate could be somewhere near 1 death out of 200 thousand. But for safety redundancy here I deliberately put an exaggerated number: 1 death out of 10 thousand. So we can be more confident to say that that the reality will be no worse than the simulation. For the non-covid lockdown death rate, we assume there will be 1 person die from lockdown related restrictions out of 1 million people in home confinement only. For we’ve seen so far, this should be a conservative estimation. We don’t want to underestimate the COVID death and overestimate the lockdown death.

```markdown
const
	virusLongevityInHost=21, // BA2 survival days inside host
	virusLongevityInEnv=7,	// BA2 survival days outside host
	mobilitySkewness=0.7, // proportion of traveling within adjacent communities
	population= 260000, // population index of Shanghai
	scale=100, // times which we get the real population number
	seedCases= 3, // positive case proportion in total population at the begining
	infectionRate= 0.00252, // chance for a positive case to spread the virus to surrounding negative individuals in a day
	vaccinationRate = 0.776, // vaccination rate : vaccine can ONLY reduce death rate and won't affect spreading rate
	vaccinationEffectiveRate = 0.8; // chance for vaccinated people to resist BA2

	otherDeathRatio= 0.000001, // severe disease hit rate per person per day
	deathRateBA2=0.0016; // 1 death out of 10,000 positive cases
```

```markdown
var
  // [Quarantine Rate, Mobility Throttle[防范区, 管控区, 封控区], 
  //	Quarantine Lag, Scenario Name]
  scenarios=[
  // Scenario #0: No Interference
    [0.00, [1.00,1.00,1.00], 0, "No Intervention"],  	
  // Scenario #1: Shanghai Strict Mode
    [0.99, [0.10,0.05,0.01], 3, "Shanghai Strict Mode"],
  // Scenario #2: Shanghai Loose Mode
    [0.90, [0.40,0.05,0.01], 3, "Shanghai Loose Mode"],  
  // Scenario #3: Global Coexist Mode
    [0.30, [1.00,1.00,1.00], 7, "Global Coexist Mode"]   
  ],
  // intervention approach applied since interventionStartDay
  interference=3, 
```

And here is how I want the simulation to be visualized. As you can see we have many green boxes here. Each represents a residence community. There are 2600 people live in each of these communities. If we put a matrix of 100 by 100 communities, we get a city with 26 million population. 
Let’s zoom out. You may notice the 3 red boxes out there. These are the communities in which our seed positive cases were first detected. In fact the boxes were color coded according to the Shanghai 3-phase lockdown system where Red stands for Complete Lockdown, Amber for Partial Lockdown and Green for Prevention. 
Then on the right we will have all the constants, parameters and variables for the simulation. 
On the bottom left, we have 5 charts to show key indicators like the daily new cases, death toll etc.

![img](img/shanghai/2.jpeg)

Then I stitched all the hypotheses, scenarios, models and data together with computer programming, and realized the visualization concept I’ve just shown you. Some critical codes are like the following:

- Virus spreading in communities:
```markdown
if (p.status==0 && !p.immune){
	var x=p.x, y=p.y;
	if (Math.random() < infectionRate)
		if (moved==0 && tiles[y][x].lastPositive+virusLongevityInEnv>=day ||
			(moved==2 && tiles[yy][xx].lastPositive+virusLongevityInEnv>=day) || (moved==1 &&
		(  (x<tileW-1) && (tiles[y][x+1].lastPositive+virusLongevityInEnv>=day)
		|| (x>0) && (tiles[y][x-1].lastPositive+virusLongevityInEnv>=day)
		|| (y<tileH-1) && (tiles[y+1][x].lastPositive+virusLongevityInEnv>=day)
		|| (y<tileH-1) && (x<tileW-1) && (tiles[y+1][x+1].lastPositive+virusLongevityInEnv>=day)
		|| (y<tileH-1) && (x>0) && (tiles[y+1][x-1].lastPositive+virusLongevityInEnv>=day)
		|| (y>0) && (tiles[y-1][x].lastPositive+virusLongevityInEnv>=day)
		|| (y>0) && (x<tileW-1) && (tiles[y-1][x+1].lastPositive+virusLongevityInEnv>=day)
		|| (y>0) && (x>0) && (tiles[y-1][x-1].lastPositive+virusLongevityInEnv>=day) ))) {
			p.status=1;
			p.testedPositive=day;
			tiles[p.y][p.x].status=2; 
			tiles[p.y][p.x].lastPositive=day;
		};
};
```

- Quarantining:
```markdown
if (p.status==1 && !p.quarantined && (day-p.testedPositive)>=scenarios[scenario][2])
	if (Math.random()<scenarios[scenario][0]) 
		p.quarantined=1;
```

- Individual recovering, getting immuned or dead from Covid:
```markdown
if (p.status==1 && day-p.testedPositive>durationPositive){
	p.status=0;
	if (p.quarantined) p.quarantined=0;
	p.immune=1;
	var vaccineWorks= 0;
	if (p.vaccinated && Math.random()<vaccinationEffectiveRate)
		vaccineWorks=1;			
	if (Math.random()<deathRateBA2 && !vaccineWorks) p.dead=1;
};
```

- Individual dead from other causes related to the lockdown:
```markdown
if (Math.random()<otherDeathRatio && 
	Math.random()>scenarios[scenario][1][tiles[p.y][p.x].status]){
	p.dead=2;
};
```

Now it’s time to play with your creation. First is the Shanghai Strict Mode, in which you will have the highest quarantine rate for positive cases, and most people will be required to stay at home. For each of the 3 phase respectively: 15 percent mobility freedom for communities under Prevention, 5 percent for those under Partial Lockdown, and 1 percent for Complete lockdown. 

![img](img/shanghai/3.jpeg)

Then the Loose Mode, in which you will have lower but still high quarantine rate for positive cases, and people still be required to stay home but the restrictions are less strictly followed. Again, for each of the 3 phase: 40 percent mobility freedom for communities under Prevention, 20 percent for those under Partial Lockdown, and 5 percent for Complete lockdown. 

![img](img/shanghai/4.jpeg)

Then the third scenario, the so called coexist mode which has been widely adopted by most countries in the world, in which you will have limited quarantine rate for positive cases, and besides the positive cases there is no mobility restrictions. You may ask, if the pandemic will end at all? Yes because people get immuned for a while after contract the virus, and when infection rate climbs to a certain degree, the virus will run out of food and die out - this is also called Herd Immunity.

![img](img/shanghai/5.jpeg)

Now we’ve done all simulations for the 3 scenarios. Let’s conclude a little bit. Here you can see the total death toll for the 3 scenarios, or the 3 different interference methods. The Loose Mode has the highest total death toll of nearly 4 thousand. Strict Mode caused more than 2 thousand deaths. For both of these 2 lockdown scenarios, there is no death reported from contracting the virus - all deaths were caused by the restrictions on mobility. Total infections are 0.4M and 1.4M respectively. In Strict mode, the pandemic ended in 4 months, and in the Loose mode, it lasted more than 10 months. In the coexist mode, total infection reached 15m when a 4 year long pandemic is over. Total death toll from the BA2 infection was less than 1 thousand. And obvious there was no death from lockdown measures because there was no lockdown at all.

![img](img/shanghai/6.jpeg)

Finally, I wish the pandemic will end and we will go back to our normal life soon. Stay safe until then!

Thank you.

#### DISCLAIMER

ANY THEORIES, DATA & MODELS HEREIN ARE IMAGINARY AND NOT REFLECTING THE REAL WORLD. ANY PREDICTIVE RESULTS HEREIN ARE JUST FOR THE PURPOSE OF ACADEMIC RESEARCH. NO PERSON OR ENTITY SHOULD MAKE DECISIONS ON THESE PREDICTIONS. BY NO MEANS THE AUTHOR WILL BE HELD ACCOUNTABLE FOR ANY DAMAGE IT MAY CAUSE. 

All Right Reserved (C) Eddy K. Dong 2022

[GO BACK TO TOP](#eddys-cyber-garage)



### SOLAR SIMULATION (FOR MY SON MARS)

Floating in the solitude of the deep darkness of the universe, you are in your super spaceship and about to explore our Solar system. The universe you're in is 2-dimentional - simplifying things but still reflecting much of the essentials - and in it there are only the Sun and its 9 planets (Pluto included for the moment, and plus the Moon) and you & your spaceshipt pulling one another by the Newton's law - Gravity is considered in beteen any 2 of the 12 astronomical objects including you.

First, let's warm up by trying the controls. Use arrow key Up/Down for speed up/down, left/right for steering. "A"/"Z" to zoom in/out, "Q"/"W" to accelerate/decelerate in time (Known bug: high time mulitiplier may cause significant inaccuracy in the simulation). 

[Transport me to that Spaceship!](https://eddydong.github.io/solar)

![img](img/solar/1.jpg)

Now you can try to orbit your home - the Earth. At the given speed of 7.9km/s, speed relevant to the Earth to be precise, it should be circling the Earth perfectly. "Relevant" here refers to the fact that the Earth itself is orbiting the Sun at roughly 30km/s - yes that's fast! You may also ask, is the Sun also moving? For sure! It travels around the center of the Milkyway Galaxy. That's it - in physics, nothing is absolutely "static". It's all about relativity!

![img](img/solar/2.jpg)

OK now it's time to follow the steps of the Apollo 11 - fly me to the Moon! Now you'll find out how FAR the moon actually is... And how difficult it is to orbit your spacecraft around a fast moving astronomical object. When you travel from the Earth to the Moon, the dominant gravity field will change from the Earth's to the Moon's - to be precise there is also the Sun's somewhere in the middle - and you need to change your reference system to the Moon as well to obtain relative speed/acceleration and angle etc.

![img](img/solar/21.jpg)

Since you've successfully traveled and orbited the Moon, you might start thinking of something even bigger. What about other planet? Maybe the most unique looking one... the one with the big ring? Why not! Challenge is that it's such a LOOOOOONG distance between the Saturn and the Earth, you may need to accelerate to the speed of light - no worries here! Your super space literally has no limit on speed! Just push "UP" and and it will soon bring you up to the God speed! (Be aware to decelerate speed/time when it comes closer or you will miss your shot!)

![img](img/solar/3.jpg)

All astronomical objects are presented with high-resolution textures for this simulation. The Saturn, for instance, has a beautiful signature ring that you can fly through! Enjoy dancing with the big ring planet!

![img](img/solar/4.jpg)

If flying through the ring of Saturn is still not crazy enough for you, let's switch of the collision detection for a while and do somethign really crazy! First, let's feel the gravity pull. You'll be lanuching the spaceship from the surface of the Earth and if you do not steer it to the orbital direction, it will do a free fall to the center of the Earth and be accelerated to an unbelievable high speed. This is called the Gravity Assist - an extreme one in this case - it's so strong and it fires you like a bullet to the deep space for a journey of no return...

![img](img/solar/5.jpg)

The nex crazy thing you might find interesting to do is to orbit the Sun. You will find out how fast you must be to balance the mighty pull of the Sun. Teperature is not considered in this simulation so don't be afraid of being melt down if getting too close to the surface. With a larger speed, you will enter an elliptic orbit around the Sun and, yes, you will be like a comet! Zoom out in view & time to see your trail and feel the formidable vast of space!

![img](img/solar/6.jpg)

Finally, let's do some physical experiments, like the 2-star system. If you put another Jupiter near the current Jupiter and give it an initial speed, you'll likely get them circling around each other, and at the same time both of them circling the Sun. Turning on the trail display and you'll find how amazing their moving is. One step further, if you add the 3rd Jupiter into this 2-star system, you'll witness the ["Double Pendulum"](https://en.wikipedia.org/wiki/Double_pendulum#Chaotic_motion) or the Chaotic Motion or the Butterfly Effect - Chances are, they will smash into each other, if collision detection is on, or one or more Jupiters will shoot out of the system by the Gravity Assist.

![img](img/solar/7.jpg)

To build this, I started off by researching on the physics behind the curtain, like the [Wikipedia on the Elliptic orbit](https://en.wikipedia.org/wiki/Elliptic_orbit) and many more. Finally I concluded the following key formulars for the computation of real-time orbit, speed/acceleration and the angle for each and every astronomical bodies including the little spaceship.

```markdown
//name, img, imgScale, imgOffsetX, imgOffsetY, x, y, r, m, v, dv
r=Math.pow((X-x)*(X-x)+(Y-y)*(Y-y),1/2);
A=dv-getAngle(X-x, Y-y);
a=-G*M*m/(m*v*v-2*G*M*m/r);
d=G*G*M*M*m*m-4*(m*v*v*r*r*Math.sin(A)*Math.sin(A)/2)*(G*M*m/r-m*v*v/2);
c1=1/((G*M*m+Math.pow(d, 1/2))/(m*v*v*r*r*Math.sin(A)*Math.sin(A)))-a;
c2=1/((G*M*m-Math.pow(d, 1/2))/(m*v*v*r*r*Math.sin(A)*Math.sin(A)))-a;
b=Math.pow(a*a-c2*c2,1/2);
var tt=(r*r+4*c2*c2-(2*a-r)*(2*a-r))/(4*r*c2);
if (tt>1){tt=1} else if (tt<-1){tt=-1};
t= Math.acos(tt);
rot=t+getAngle(x,y);
```

Here is one of the original sketches I made to figure the physics out, which led to above shown codes:
![img](img/solar/IMG_9948.jpeg)

All phsical data were obtained from the internet and should be roughly reflect the reality of our Solar System.

```markdown
//name, img, imgScale, imgOffsetX, imgOffsetY, x, y, r, m, v, dv
earth= new Ball('Earth', ImgEarth, 1.02, 0,0, -1.5e11, 0, 6.375e6, 5.965e24, 29.79e3, Math.PI/2);
moon=new Ball('Moon', ImgMoon, 1, 0,0,  -384000e3-1.5e11,0, 1738.14e3, 7.349e22, 1020+29.79e3, Math.PI/2);
sun = new Ball('Sun', ImgSun, 1.17,  0,0, 0,0, 696300e3, 1.989e30, 0, 0);
mercury = new Ball('Mercury', ImgMercury, 1, 0,0,  -5791e7,0, 2440e3, 5.965e24*0.0553, 47.89e3, Math.PI/2);
venus = new Ball('Venus', ImgVenus, 1, 0,0,  -108208930e3,0, 6052e3, 5.965e24*0.815, 35.03e3, -Math.PI/2);
mars = new Ball('Mars', ImgMars, 1, 0,0,  -227940000e3,0, 3398e3, 5.965e24*0.1074, 24.13e3, Math.PI/2);
jupiter = new Ball('Jupiter', ImgJupiter, 1, 0,0,  -778330000e3,0, 71492e3, 1.9e27, 13.06e3, Math.PI/2);
saturn = new Ball('Saturn', ImgSaturn, 3, -0.012,0.016,  -1429400000e3,0, 60330e3, 5.965e24*95.18, 9.65e3, Math.PI/2);
uranus = new Ball('Uranus', ImgUranus, 1, 0,0,  -19.2184*AU,0, 25559e3, 5.965e24*14.54, 6.81e3, Math.PI/2);
neptune = new Ball('Neptune', ImgNeptune, 1, 0,0,  -30.1104*AU,0, 24764e3, 5.965e24*17.15, 5.43e3, Math.PI/2);

me = new Ball('Ship', ImgShip, 1, 0,0, -(6.375e6+1e2+550e3)-1.5e11, 0, 1e2, 100e3, 9000+29.79e3, Math.PI/2+0.01);
```

Then I setup an annimation loop with [RequestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) and used the [Canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) to visualize all objects and their ever changing properties.

This project was designed & implemented for my son, Mars, who is a big fun of Astronomy back in 2020 or so - 2 years ago from today. Not all funcitonalities mentioned above were were consolidated into the same program, the html file, yet, meaning you may need to try other ones in the same folder - let me know if you have particular interest in any of them and I'll help you find it out.

Finally, here are some sketches by my son illustrating his universe fantasy - I carelessly found them in his secret journal ;-)

![img](img/solar/IMG_9950.jpeg)
![img](img/solar/IMG_9956.jpeg)
![img](img/solar/IMG_9951.jpeg)
![img](img/solar/IMG_9952.jpeg)

I was indeed inspired by his fantasy to make this project. Hope it also inspires you in some way.

Eddy K. Dong
June 11, 2022

[GO BACK TO TOP](#eddys-cyber-garage)



### GREEDY WORMS (FOR MY SON MARS)

[Play the prototype with AI](https://eddydong.github.io/greedy_worms)

First of all, let's visit and worship the original work [Slither.io](http://slither.io). The was where all these started from. The only issue with Slither.io is that the network latency sometimes too high for this kind of real time multi-player action game. In fact this is common for any global MMOBA (Massive Multiplayer Online Battle Arena). Besides that, it's still the best in this category. 

It followed the basic rules for a good game in almost every perspective: 
- Extremely easy to start with;
- But really hard to be at the top;
- Failure comes with a feeling that we're gonna make it for the next attempt - and so you're hooked firmly;
- Very strong interactions with a lot of real people in real time;
- Very simple but very attractive graphics;
- Webpage based - no hassle installing anything.

The most amazing part of the game to me is: You start with a tiny body and also a tiny field of view, in which others may appear gigantic around you. When you grow up, your field of view is widen accordingly and you may notice some tiny ones around you - just like your starting point. At different stage you'll experience differenct mentalities - this is quite facinating!

I was so into this little game and started thinking of writing one from scratch on my own - my typical reaction when I find something (not necessarily games or any computer programs) interesting in life ;-) 

Here it is. You start from tiny, admiring those gigantic creatures slithering besides you.

![img](img/greedy_worms/2.jpg)

And gradually you grew up a bit...

![img](img/greedy_worms/3.jpg)

When you were identified as the next prey by a huge dragon...

![img](img/greedy_worms/4.jpg)

And circled by it...

![img](img/greedy_worms/5.jpg)

But you managed to slaughter the dragon. You ate its engergy and became a dragon...

![img](img/greedy_worms/6.jpg)

And here is the secrets behind the AI of the NPC's:

A NPC has 3 "sentiment" modes:
1. Flee danger, when there is danger;
2. Attack other players, when 1) there is no danger and 2) there is a good chance to cut or to circle others;
3. Hunt for food, when there is no danger, nor opportunity for attack;

And 2 movement modes:
1. Walk (or slow slithering, for "sentiment" mode #1 & #3)
2. Run (or fast slithering, for "sentiment" mode #2)

The world is devided into tiles, making it more efficient for the NPC's to locate food. It will decide which tile is its desination based on evaluation of the total food value of the tiles and their current distance to itself. When it's far, it guide itself to the center of that tile. When it gets closer, it will decide its desination on individule foods, instead of tiles. I'm using a technique called LoD (Level of Details) in the 3D rendering to improve efficiency in the food search.

In the following debug-mode screenshots, you can see the above mentioned tiles clearly. Also you will notice the sector around the head of the NPC worms. That's how they feel the surrounding objects and tell if there is a threat of a chance to attack others. (Push "Enter" to enter/exit debug mode)

![img](img/greedy_worms/7.jpg)
![img](img/greedy_worms/8.jpg)

Finally in the God-view mode, your field of view will be set to the maximum value and you'll have a holistic view of the entire battle arena. (Push "Esc" to enter/exit "God-view" mode)
![img](img/greedy_worms/9.jpg)

Above prototype was done by myself in the summer of 2015 (?) and then I decided make it into a full-fledged game by accomplishing the following ambicious tasks:

1. Use Unity to make it a native app for IOS and Android phones;
2. Add MMOBA elements - heroes and skill trees (2 unique skills x 16 unique heroes introduced for the first batch);
3. Make it a real MMOBA - supporting hundreds of people play in the same arena at the same time, even supporting realtime audio chat; 
4. High concurrency low latency servers setup in Sound, North & East China.

This time I hired a team to do the job while I was the project investor and the product manager. Then there came the Worm Heroes, the full-fledged version of the Greedy Worms the prototype. 

![img](img/greedy_worms/a1.jpeg)
![img](img/greedy_worms/a2.jpeg)
![img](img/greedy_worms/a3.jpeg)
![img](img/greedy_worms/a4.jpeg)
![img](img/greedy_worms/a5.jpeg)
![img](img/greedy_worms/a6.jpeg)
![img](img/greedy_worms/a7.jpeg)
![img](img/greedy_worms/a8.jpeg)
![img](img/greedy_worms/a9.jpeg)
![img](img/greedy_worms/a10.jpeg)


[GO BACK TO TOP](#eddys-cyber-garage)

### REVSERSI

[Play Reversi with Eddy's brainchild!](https://eddydong.github.io/reversi)
(Use Backspace to regret and withdraw your last move)

![img](img/reversi/1.jpg)

This is to commemorate my first AI algorithm back in 2001. The original one was done in Turbo Pascal and then in Delphi - congrats you're "experienced enough" if you've heard any of those names ;-) - and now I re-wrote it in Javascript.

The basic idea behind the AI algorithm was like the following:

1. List all eligible positions that you can place your piece at;
2. Note down the gain (number of opponent pieces you can reverse) you can get for each of these eligible positions;
3. Imagine you are now your opponent, and repeat #1 & #2 but for #2 you add a minus sign in front of the gain;
4. Repeat step #1 to #3 for several times (depth of AI thinking);
5. The position with the highest gain value should be your next step.

Sound easy but it needs some work to really turn the idea into workable code, for instances:

First you need to setup the chess map:

```markdown
function initmap(){
	map=[];
	for (var i=0; i<mapsize; i++){
		map.push([]);
		for (var j=0; j<mapsize; j++)
			map[i].push(0);
	};
	map[parseInt(mapsize/2)-1][parseInt(mapsize/2)-1]=1;
	map[parseInt(mapsize/2)-1][parseInt(mapsize/2)]=2;
	map[parseInt(mapsize/2)][parseInt(mapsize/2)-1]=2;
	map[parseInt(mapsize/2)][parseInt(mapsize/2)]=1;
};
```

For #1, you need define the game rules - how to define an eligible move:
```markdown
function cango(player, themap){
	for (var i=0; i<mapsize; i++)
		for (var j=0; j<mapsize; j++)
			if (caneat(player,themap, j,i)[0]>0) return true;
	return false;
};
```

For #2, you need to calculate your gain (or your loss if current player is your opponent) with a [recursive function](https://en.wikipedia.org/wiki/Recursion_(computer_science)) like the following:
```markdown
function bestmove(player, ai_level){
	var tempmap=[], direct_gain;
	var maxgain, maxx, maxy, over=true;
	for (var i=0; i<mapsize; i++)
		for (var j=0; j<mapsize; j++) {
			direct_gain=caneat(player, draftmap, j,i);
			if (direct_gain[0]>0){			
				if (ai_level>1){
					tempmap=copymap(draftmap);
					eat(player, draftmap, j, i);
					var res, gain;
					res = bestmove(3-player, ai_level-1);
					if (res[2]==-99991021) {
						bestmove(player, ai_level-1);
						gain = direct_gain[1] + res[2];
					} else gain = direct_gain[1] - res[2];
					draftmap=copymap(tempmap);
				} else gain=direct_gain[1];
				if (maxgain==null || gain > maxgain || 
					(gain==maxgain && Math.random()>0.5)){
					maxgain=gain;
					maxx=j;
					maxy=i;
				};
				over=false;
			};
		};
	if (over){
		if (player==curPlayer) maxgain=-99999
			else maxgain=-99991021;
	};
	return [maxx,maxy,maxgain];
};
```

Please refer to the [My Github Project](https://github.com/eddydong/reversi) for the rest of the program, together with other versions, for instance the "AI Combat Mode", in which you put different AI algorithms, or the same algorithm but with different sets of AI parameters, and let them fight with each other and find out who is number one. You may setup 100 rounds and record the win's and loss'es for each algorithm - just like the A/B test. [AI Combat Version](https://eddydong.github.io/reversi/reversi AI arena 1.html)

Here is a screenshot for the ongoing combat between 2 of my algorithms:
![img](img/reversi/2.jpg)

Theoretically, if the computer is fast enough to do a 30 level deep thinking using this algorithm, there will be no chance for its challenger to win at all. But the reality is: this algorithm runs slow even on a modern computer when the depth of search is set above 6. In other words, although it has already been very hard to defeat, but it's obviously not 100% sure that it will win the game.

One way to improve this is to add some human experience rules on top of the search algorithm. To be specific, at the last (bottom) layer of the imaginary evaluation, instead of returning the gain/loss of that layer, arbitrage human touches on important positions on the map - like the 4 corners and some positions near them, and the 4 edges etc - will be added on top of the results of the search algorithm.

Looks like a easy game and there are only a few scores of positions to consider and thus the game is always very quick. You may start thinking of solving it completely with any better algorithm. But the reality is:

>The Othello 8x8 game tree size is estimated at 10<sup>54</sup> nodes, and the number of legal positions is estimated at less than 10<sup>28</sup>. The game remains unsolved. A solution could possibly be found using intensive computation with top programs on fast parallel hardware or through distributed computation.
[Computer Othello on Wikipedia](https://en.wikipedia.org/wiki/Computer_Othello)

Let's go visit and worship the strongest Reversi (or Othello) algorithm on this planet so far:
[LOGISTELLO](https://skatgame.net/mburo/log.html)



All Right Reserved (C) Eddy K. Dong 2022
[GO BACK TO TOP](#eddys-cyber-garage)


