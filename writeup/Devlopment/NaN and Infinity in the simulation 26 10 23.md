An odd error arose during the simulation stage. After some time, the entire playfield would go blank, even after some computation. Upon further investigation, this was due to the density field filling up with `Single.NaN` . This is an unsupported value for the simulation to have, and when found in any one of the cells of the simulation, can rapidly propagate through calculation to the other cells.
### How Did This Happen?
There are only two ways in C# for the `Single.NaN` value to come about. 
1. Somewhere, the calculation $\frac{0}{0}$ is being performed. This results in Not A Number.
2. Some function is intentionally returning not a number.
It is trivial to rule out that any function is intentionally returning NaN, as the mathematical functions used are all directly implemented, rather than relying on the standard library.

This leaves us with a dodgy division happening somewhere, which is easy to pinpoint, as there is only one place in the code where a division operation takes place (this is a performance optimisation): the `lin_solve` function. 
### Using A Debugger
By using a debugger, and placing a conditional breakpoint, it is possible to stop the code precisely the first instant that NaN is detected in the density field. However, this reveals some puzzling results.
The calculation is returning NaN, however, it is doing so in a way which does not seem to conform to the two rules above.
```cs
float calc = (x0[((i) + (N + 2) * (j))]
			    + a * (x[((i - 1) + (N + 2) * (j))]
			    + x[((i + 1) + (N + 2) * (j))]
				+ x[((i) + (N + 2) * (j - 1))]
				+ x[((i) + (N + 2) * (j + 1))])) / c;
if (Single.IsNaN(calc))     //Kills all NAN occurences as soon as possible
{
    calc = 0;
}
if (Single.IsInfinity(calc))    //Preventing runaway density (also prevents some NAN occurences
{
    calc = 1;
}
x[((i) + (N + 2) * (j))] = calc;
```
The calculations used to fill the `calc` variable amount to the following: $\frac{c + \infty}{1}$  , where $c$ is a bunch of constants formed during the calculation.
Wait a minute, how did we get to infinity? Well, as it happens, one of the elements in the array had become infinity somewhere in the calculations, likely of the last frame. Despite the fact that the result of this calculation being infinity, this is somehow yielding NaN. As a precaution, any values detected to be NaN are set to zero, to prevent issues like this from spreading.
### Infinity
Now that we have identified the infinity here as the most likely culprit of the undefined behaviour, we can pursue it further using the debugger. There should be only one way to get `Single.Infinity` as a result, by dividing a non-zero number by zero. This behaviour, while counter-intuitive to mathematicians, is the required behaviour set out in the IEEE-754 Specification on single and double precision floating point numbers.
Now, how do we get a division by zero here: when `c` is zero. So, let's see the formula for `c`: `1 + 4 * a`. Now the only way for `c == 0` to be true is for `a` to be -0.25f. Thus, we look for the formula for `a`:  `a = dt * diff * N * N`. This may look a bit wonky, as it is part of the calculations used to make the *Gauss-Sieldel Relaxation* method of solving a set of simultaneous equations more stable. 

Either way, we know that `N` is a positive whole number, equal to the dimensions of the array, `dt` is the positive fraction of a second that has passed since the last frame, and `diff` is a positive number defining how easily the fluid diffuses through the aether. Thus we come to a puzzling resolution: it is impossible for `c` to be -0.25f, and so impossible to be dividing by zero, and so impossible to be gaining infinity, and from that: impossible to get NaN.  Despite all this, we plainly have been.

I don't really have a good solution here except to say that this is behaviour I could not replicate anywhere else. So, as a protection measure against this, if the value is detected to be NaN, it is forced to zero, and if it is Infinity, it is forced to one. This is a messy, hacky way of preventing this issue, but ultimately, this is also the best way of fixing this.