# EDDY'S CYBER GARAGE

[APOLLO MUSIC IMPROVISING](#apollo-music-improvising)

[SHANGHAI PANDEMIC SIMULATION](#shanghai-pandemic-simulation)

[SOLAR SYSTEM SIMULATION](#solar-simulation-for-my-son-mars)

[GREEDY WORMS](#greedy-worms-for-my-son-mars)

[REVERSI](#reversi)

### APOLLO MUSIC IMPROVISING

[Listen to the endless piano & cello duet improvised by Apollo](https://eddydong.github.io/apollo)


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
![img](img/solar/IMG_9951.jpeg)
![img](img/solar/IMG_9952.jpeg)
![img](img/solar/IMG_9953.jpeg)
![img](img/solar/IMG_9954.jpeg)
![img](img/solar/IMG_9955.jpeg)
![img](img/solar/IMG_9956.jpeg)
![img](img/solar/IMG_9957.jpeg)
![img](img/solar/IMG_9958.jpeg)

I was indeed inspired by his fantasy to make this project. Hope it also inspires you in some way.

Eddy K. Dong
June 11, 2022

[GO BACK TO TOP](#eddys-cyber-garage)



### GREEDY WORMS (FOR MY SON MARS)

[Play the webpage game](https://eddydong.github.io/greedy_worms)

This was the project I did

![img](img/greedy_worms/1.jpg)
![img](img/greedy_worms/2.jpg)
![img](img/greedy_worms/3.jpg)
![img](img/greedy_worms/4.jpg)
![img](img/greedy_worms/5.jpg)
![img](img/greedy_worms/6.jpg)
![img](img/greedy_worms/7.jpg)
![img](img/greedy_worms/8.jpg)
![img](img/greedy_worms/9.jpg)
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

[Play Reversi with Eddy's bot!](https://eddydong.github.io/reversi)


All Right Reserved (C) Eddy K. Dong 2022
[GO BACK TO TOP](#eddys-cyber-garage)


