---
layout: default
---

Just a rainy day and I want to try something new and interesting.
Then I came across a
[Mandelbrot script](https://github.com/koehlma/snippets/blob/master/python/fractals/mandelbrot.py)
in obfuscated Python that looks like a Mandelbrot fractal.
Some code to generate a Mandelbrot fractal just looks like this fractal.
Awesome!

{% highlight python %}
_ = (
                                        255,
                                      lambda
                               V ,B,c
                             :c and Y(V*V+B,B, c
                               -1)if(abs(V)<6)else
               ( 2+c-4*abs(V)**-0.4)/i
                 ) ;v, x=1500,1000;C=range(v*x
                  );import struct;P=struct.pack;M,\
            j ='<QIIHHHH',open('M.bmp','wb').write
for X in j('BM'+P(M,v*x*3+26,26,12,v,x,1,24))or C:
            i ,Y=_;j(P('BBB',*(lambda T:(T*80+T**9
                  *i-950*T **99,T*70-880*T**18+701*
                 T **9 ,T*i**(1-T**45*2)))(sum(
               [ Y(0,(A%3/3.+X%v+(X/v+
                               A/3/3.-x/2)/1j)*2.5
                             /x -2.7,i)**2 for \
                               A in C
                                      [:9]])
                                        /9)
                                       ) )
{% endhighlight %}

I decided to generate an image and put the source code on it:

<a href="/blog/julia-fractals/mandelbrot.bmp" rel="lightbox[mandelbrot]" title="Mandelbrot Fractal with Obfuscated Python Code used to generate It">
<img src="/blog/julia-fractals/mandelbrot.jpg">
</a>

After this I became interested and tried to do something my self.
I came up with my
[own script](https://github.com/koehlma/snippets/blob/master/python/fractals/julia.py)
but not for Mandelbrot and also not obfuscated but it generates beautiful
[Julia Fractals](http://en.wikipedia.org/wiki/Julia_set).

{% highlight python %}
#!/usr/bin/env python3
# -*- coding:utf-8 -*-
#
# Copyright (C) 2012, Maximilian Köhl <linuxmaxi@googlemail.com>

import argparse
import collections
import struct

F = lambda z, c: z ** 2 + c     # julia base function
ITERATIONS = 255                # number of iterations

RGB = collections.namedtuple('RGB', ['red', 'green', 'blue'])

def gradient(from_rgb, to_rgb, length=10):
    """
    Generates a gradient from `from_rgb` to `to_rgb` with `length` steps and
    yields color by color.
    """
    diff_rgb = RGB(to_rgb.red - from_rgb.red, to_rgb.green - from_rgb.green,
                   to_rgb.blue - from_rgb.blue)
    length = length - 1
    for i in range(length + 1):
        yield RGB(int(from_rgb.red + diff_rgb.red / length * i),
                  int(from_rgb.green + diff_rgb.green / length * i),
                  int(from_rgb.blue + diff_rgb.blue / length * i))

def julia(z, c, f=F, r=2, iterations=ITERATIONS):
    """
    Generates the Julia fractal.
    """ 
    for i in range(iterations):
        if abs(z) > r: break
        z = f(z, c)
    return i

# gradient for fancy colors 
colors = (list(gradient(RGB(0, 0, 0), RGB(0, 0, 255), int(ITERATIONS * 0.1))) + 
          list(gradient(RGB(0, 0, 255), RGB(0, 255, 255), int(ITERATIONS * 0.2))) +
          list(gradient(RGB(0, 255, 255), RGB(0, 255, 0), int(ITERATIONS * 0.3))) +
          list(gradient(RGB(0, 255, 0), RGB(255, 255, 255), int(ITERATIONS * 0.4 + 1))))

if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Render some beautiful Julia fractals.')
    parser.add_argument('output', help='output image')
    parser.add_argument('-C', default=[-0.8, 0.2], type=float,
                        help='julia\'s complex parameter (c = f1 + f2)', nargs=2, metavar='f')
    parser.add_argument('--width', default=1500,
                        help='image width', type=int)
    parser.add_argument('--scale', default=400,
                        help='scale the image', type=int)


    arguments = parser.parse_args()
    
    c, width, scale = complex(*arguments.C), arguments.width, arguments.scale
    
    print('Generating Julia Fractal:')
    print('    c         : {:f}'.format(c))
    print('    f         : z ^ 2 + c')
    print('    iterations: 255')
    print('    scale     : {}'.format(scale))
    print('    width     : {}'.format(width))
    with open(arguments.output, 'wb') as output:
        # bmp header
        output.write(b'BM' + struct.pack('<QIIHHHH', width * width * 3 + 26, 26,
                                         12, width, width, 1, 24))
        # scaling
        values = []
        for i in range(int(-(width + 9) / 2), int((width + 9) / 2)):
            values.append(i / scale)
        # total steps    
        total = arguments.width ** 2
        # rendering
        for y in range(width):
            for x in range(width):
                current = width * y + x
                print('\r    progress  : {:.2f}% ({}/{})'.format(current / total * 100,
                                                                 current, total), end='')
                color = colors[julia(complex(values[x], values[y]), c)]
                output.write(struct.pack('BBB', color.blue, color.green, color.red))
        print() 
{% endhighlight %}


# Examples
Here are just some nice examples testing various parameters.
## Julia 1
    
    ./julia.py julia1.bmp
    
    c = -0.8 + 0.2i
    
<a href="/blog/julia-fractals/julia1.bmp" rel="lightbox[julia]" title="Julia 1">
<img src="/blog/julia-fractals/julia1.jpg">
</a>

## Julia 2

    ./julia.py -C 0 0.8 julia2.bmp
    
    c = 0 + 0.8i

<a href="/blog/julia-fractals/julia2.bmp" rel="lightbox[julia]" title="Julia 2">
<img src="/blog/julia-fractals/julia2.jpg">
</a>

## Julia 2 with Scale 1000
    
    ./julia.py -C 0 0.8 --scale 1000 julia2-scale1000.bmp
    
    c = 0 + 0.8i
    scale = 1000
    

<a href="/blog/julia-fractals/julia2-scale1000.bmp" rel="lightbox[julia]" title="Julia 2 with Scale 1000">
<img src="/blog/julia-fractals/julia2-scale1000.jpg">
</a>

## Julia 3

    ./julia.py -C -1 0 julia3.bmp
    
    c = -1 + 0i

<a href="/blog/julia-fractals/julia3.bmp" rel="lightbox[julia]" title="Julia 3">
<img src="/blog/julia-fractals/julia3.jpg">
</a>
