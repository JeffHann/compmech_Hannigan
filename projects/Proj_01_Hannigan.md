---
jupytext:
  formats: notebooks//ipynb,md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.4
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

```{code-cell} ipython3
import numpy as np
import matplotlib.pyplot as plt
plt.style.use('fivethirtyeight')
```

# Computational Mechanics Project #01 - Heat Transfer in Forensic Science

We can use our current skillset for a macabre application. We can predict the time of death based upon the current temperature and change in temperature of a corpse. 

Forensic scientists use Newton's law of cooling to determine the time elapsed since the loss of life, 

$\frac{dT}{dt} = -K(T-T_a)$,

where $T$ is the current temperature, $T_a$ is the ambient temperature, $t$ is the elapsed time in hours, and $K$ is an empirical constant. 

Suppose the temperature of the corpse is 85$^o$F at 11:00 am. Then, 2 hours later the temperature is 74$^{o}$F. 

Assume ambient temperature is a constant 65$^{o}$F.

1. Use Python to calculate $K$ using a finite difference approximation, $\frac{dT}{dt} \approx \frac{T(t+\Delta t)-T(t)}{\Delta t}$.

```{code-cell} ipython3
Ta = 65 #ambient temp in Fahrenheit
T1 = 85 #initial temp at 11:00 in Fahrenheit
T2 = 74 # temp at 13:00 in Fahrenheit
dt = 2 #time elapsed in hours

dTdt = (T2-T1)/dt #Temp rate of change using finite difference approximation
K = -(dTdt/(T2-Ta)) #K constant value for Newton's Law of Cooling
print('The value of K is:', K)
```

2. Change your work from problem 1 to create a function that accepts the temperature at two times, ambient temperature, and the time elapsed to return $K$.

```{code-cell} ipython3
def K_Constant(Ti, Tf, Tamb, t):
    '''This Fn returns the K constant for Newton's Law of Cooling given:
    Ti => initial temp.(F), Tf => final temp.(F), Tamb => ambient temp.(F) 
    and t => time elapsed(hours)'''
    dTdt = (Tf-Ti)/t #Temp rate of change using finite difference approximation
    K = -(dTdt/(Tf-Ta)) #K constant value for Newton's Law of Cooling
    return K
```

```{code-cell} ipython3
Constant = K_Constant(85, 74, 65, 2)
print('The value of K is:', K)
```

3. A first-order thermal system has the following analytical solution, 

    $T(t) =T_a+(T(0)-T_a)e^{-Kt}$

    where $T(0)$ is the temperature of the corpse at t=0 hours i.e. at the time of discovery and $T_a$ is a constant ambient temperature. 

    a. Show that an Euler integration converges to the analytical solution as the time step is decreased. Use the constant $K$ derived above and the initial temperature, T(0) = 85$^o$F. 

    b. What is the final temperature as t$\rightarrow\infty$?
    
    c. At what time was the corpse 98.6$^{o}$F? i.e. what was the time of death?

```{code-cell} ipython3
#Part a. Euler integration
def BodyTemp(tf, n, T_1, T_a):
    '''Returns the time values, analytical solution, and Euler integration, for the temperature of a dead body given: 
    tf => the final time in hours, n => number of time steps, T_1 => initial body temp., and T_a => ambient temp.'''
    K = Constant
    t = np.linspace(0, tf, n) #creates a numpy array with 10*tf+1 sections from 0 to tf hours
    dt = t[1]-t[0]
    
    T_ana = np.zeros(len(t)) #creates numpy zeros array for analytical soln.
    T_ana[0] = T_1 #sets initial body temp. value
    #for loop to calculate the other analytical temp. values:
    for i in range(1, len(t)):
        T_ana[i] = T_a + (T_ana[0]-T_a)*np.exp(-K*t[i])
    
    T_eul = np.zeros(len(t)) #creates numpy zeros array for euler integration soln.
    T_eul[0] = T_1 #sets initial temp. value
    #for loop to calculate the other euler temp. values:
    for i in range(1, len(t)):
        T_eul[i] = T_eul[i-1]-K*(T_eul[i-1] - T_a)*dt
    return(t, T_ana, T_eul) #returns the time, temp.,analytical and euler integration 

time, a_ana, a_eul = BodyTemp(5, 31, 85, 65)

#plotting the soln. for Part a.
plt.plot(time, a_eul, label='Euler Integration')
plt.plot(time, a_ana, label='Analytical Soln.')
plt.ylim(64,85)
plt.xlabel('Time (hours)')
plt.ylabel('Body Temp.(Fahrenheit)')
plt.title('Euler Intergration Convergence')
plt.legend()
```

```{code-cell} ipython3
# Part b. Final Temperature
time, a_ana, a_eul = BodyTemp(48, 97, 85, 65)

#plotting the soln. for Part b.
plt.plot(time, a_eul, label='Euler Integration')
plt.plot(time, a_ana, label='Analytical Soln.')
plt.ylim(60,85)
plt.xlabel('Time (hours)')
plt.ylabel('Body Temp.(Fahrenheit)')
plt.title('Temperature as t -> infinity')
plt.legend()
print('Changing the time range to extend to 2 days from body discovery \nshows both solutions do not dip below the ambient temp. of 65 deg. F')
```

```{code-cell} ipython3
# Part c
from datetime import *
from math import modf

Tb = 98.6 #Human(live) body temp becomes the new initial temp.
K = Constant #same K constant derived in parts 1 and 2
T0 = 85 #body temp. at discovery at 11:00am
Ta = 65 #ambient temp. 
t0 = datetime.strptime("09/13/22 11:00", "%m/%d/%y %H:%M") #sets time body was found to 11:00am

dt = np.log((T0-Ta)/(Tb-Ta))/(-K) #solves for the time delta to cool to 85 from 98.6
mins, hrs = modf(dt) #splits dt into hours and decimal minutes
hrs = int(hrs) #changes variable to integer form
mins *= 60 #converts to whole minutes from decimal minutes
secs, mins = modf(mins) #splits minutes to minutes and seconds
mins = int(mins)
secs = int(secs*60)
death_t = t0 - (timedelta(hours=hrs, minutes=mins, seconds=secs))
death_t = datetime.time(death_t)
print("The time of death was", death_t)
```

```{code-cell} ipython3

```
