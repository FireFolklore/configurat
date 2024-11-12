# GtkInteract

[![Build Status](https://travis-ci.org/jverzani/GtkInteract.jl.svg?branch=master)](https://travis-ci.org/jverzani/GtkInteract.jl)

The [`Interact`](https://github.com/JuliaLang/Interact.jl) package
brings interactive widgets to `IJulia` notebooks. In particular, the
`@manipulate` macro makes it trivial to define simple interactive
graphics.  `Interact` can animate graphics using `Gadfly`, `PyPlot`,
or `Winston`. For more fluid graphical animations, the 
[`Patchwork`](https://github.com/shashi/Patchwork.jl) package can be
used to efficiently manipulate SVG graphics, including those created
through `Gadfly`,

The `GtkInteract` package modifies `Interact`'s `@manipulate` macro to
allow interactive widgets from the command-line REPL using the `Gtk`
package for the widget toolkit. This package then allows for similarly
easy interactive graphics from the command line. It works with the
following packages: `Immerse`, or `Plots` with the `:immerse` backend.
There is some experimental, though buggy, support for using `PyPlot`
graphics. `Winston` graphics should also work.

The basic syntax is the same as for `Interact`. For example, 

```
using GtkInteract, Plots
backend(:immerse)
@manipulate for ϕ = 0:π/16:4π, f = Dict(:sin=>sin, :cos=>cos)
    plot(θ -> f(θ + ϕ), 0, 25)
end
```

This produces a layout along the lines of:

![Imgur](http://i.imgur.com/1MiynXf.png)

Using `Immerse` directly is also in the design (though currently there is an issue)

```
using GtkInteract, Immerse
@manipulate for ϕ = 0:π/16:4π, f = Dict(:sin=>sin, :cos=>cos)
    xs = linspace(0, 25)
    ys = map(θ -> f(θ + ϕ), xs)
    Immerse.plot(x=xs, y=ys)
end
```

(The issues is the canvas does not get refreshed.)

## Text output

Text output can also be displayed (though not as nicely as in `IJulia` due to a lack of HTML support):

```
using GtkInteract, SymPy
x = symbols("x")
@manipulate for n=1:5
   a = diff(sin(x^2), x, n)
   SymPy.jprint(a)
end
```

## basic widgets

The basic widgets can also be used by hand to create simple GUIs:

```
using GtkInteract

w = mainwindow(title="Simple test")
n = slider(1:10, label="n")
m = slider(11:20, label="m")
cg = cairographic()

append!(w, [n, m, cg])		# layout widgets
```


More complicated layouts are possible using a few layouts similar to those in the `Escher` package:

```
window(vbox(hbox(n, m),
                     grow(cg)),
              title="Some title")
```


```
@map push!(cg, plot(sin, n*pi, m*pi))
```

For now, there are no layout options.

## Installation

Until this package lands in `Julia`'s METADATA it should be installed via "cloning":

```
Pkg.clone("https://github.com/jverzani/GtkInteract.jl")
```

This package requires [Gtk](https://github.com/JuliaLang/Gtk.jl) (for
GTK+ 3, not GTK+ 2). See that page for installation notes.

## Using with PyPlot

[This is currently broken on a mac, segfaulting with the interactivity!]

There is experimental support for plotting with PyPlot. Using `PyPlot`
requires an extra wrapper function, called `GtkInteract.withfig`. (The `withfig`
function is defined in  `PyPlot` and modified here, hence the module qualification.)

```
using GtkInteract, PyPlot

f = figure()
@manipulate for n in 1:10, m in 1:10
    GtkInteract.withfig(f) do
      ts = linspace(0, 2*n*m*pi, 2500)
      xs = [sin(m*t) for t in ts]
      ys = [cos(n*t) for t in ts]
      PyPlot.plot(xs, ys)
    end
end
```

It can be a bit slower, as this does not draw onto a canvas, but
rather creates an image file and displays that on each update.  In the
background `pygui(false)` is called. Not doing so leads to a crash on
some machines.

(To copy-and-paste code that works with `Interact` simply requires some local definition such as `withfig=GtkInteract.withfig`.)

