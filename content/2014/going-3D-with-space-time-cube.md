Title: Going 3D With Space Time Cube
Date: 2014-09-02 17:35
Tags: python, twitter
Category: development

<p>Seeing <a href="http://anitagraser.com/2012/08/05/space-time-cubes-exploring-twitter-streams-3/">Anita&#8217;s space-time cube</a> back in 2013 was a moment of <em>woooow</em> for me. I&#8217;ve been interested in unusual ways of displaying data ever since I started studying GIS and this one was just great. <em>How the hell did she make it?!</em>, I thought back then.</p>

<p>And I asked her, we had a little e-mail conversation and that was it. I got busy and had to postpone my attemps to create that viz until I dove into my diploma thesis. So&hellip;here you go.</p>

<h3>Recipe</h3>

<p>What you need is:</p>

<ul>
<li><strong><a href="https://github.com/jdf/processing.py">processing.py</a></strong> which is a Python port of <a href="http://processing.org/">processing</a> environment.</li>
<li>A <strong>basemap</strong> that fits the extent you are about to show in the viz. I recommend QGIS for obtaining an image.</li>
<li>A <strong>JSON file</strong> with tweets you got via <a href="/2014/analyzing-twitter-languages-with-streaming-api/">Twitter REST API</a> (yes, the viz was made to display tweets).</li>
<li>A <strong>python script</strong> I wrote.</li>
</ul>

<h3>How to make it delicious</h3>

<p>First things first, you need to add a <code>timestamp</code> property to tweets you want to show (with the following Python code). <code>created_at</code> param is a datetime string like <code>Sat Jun 22 21:30:42 +0000 2013</code> of every tweet in a loop. As a result you get a number of seconds since 1.1.1970. </p>

<pre><code>def string_to_timestamp(created_at):
    """Return the timestamp from created_at object."""
    locale.setlocale(locale.LC_TIME, 'en_US.utf8')
    created_at = created_at.split(' ')
    created_at[1] = str(strptime(created_at[1], '%b').tm_mon)
    timestamp = strptime(' '.join(created_at[i] for i in [1,2,3,5]), '%m %d %H:%M:%S %Y') # returns Month Day Time Year
    return mktime(timestamp)
</code></pre>

<p>As you probably guess, the <code>timestamp</code> property is the one we&#8217;re gonna display on the vertical axis. <strong>You definitely want the tweets to be sorted chronologically in your JSON file!</strong></p>

<pre><code>#!/usr/bin/python
# -*- coding: utf-8 -*-
#avconv -i frame-%04d.png -r 25 -b 65536k  video.mp4

from peasy import PeasyCam
import json

basemap = None
tweets = []
angle = 0

def setup():
    global basemap
    global tweets

    size(1010, 605, P3D)

    data = loadJSONArray('./tweets.json')
    count = data.size()

    last = data.getJSONObject(data.size()-1).getFloat('timestamp')
    first = data.getJSONObject(0).getFloat('timestamp')

    for i in range(0, count):
        lon = data.getJSONObject(i).getJSONObject('coordinates').getJSONArray('coordinates').getFloat(0)
        lat = data.getJSONObject(i).getJSONObject('coordinates').getJSONArray('coordinates').getFloat(1)
        time = data.getJSONObject(i).getFloat('timestamp')

        x = map(lon, -19.68624620368202116, 58.92453879754536672, 0, width)
        y = map(time, first, last, 0, 500)
        z = map(lat, 16.59971950210866964, 63.68835804244784526, 0, height)

        tweets.append({'x': x, 'y': y, 'z': z})

    basemap = loadImage('basemap.png')

    cam = PeasyCam(this,53,100,-25,700)
    cam.setMinimumDistance(1)
    cam.setMaximumDistance(1500)

def draw():
    global basemap
    global tweets
    global angle

    background(0)

    # Uncomment to rotate the cube
    """if angle &lt; 360:
        rotateY(radians(angle))
        angle += 1
    else:
        angle = 360 - angle"""

    # box definition
    stroke(150,150,150)
    strokeWeight(.5)
    noFill()
    box(1010,500,605)


    # basemap definition
    translate(-505,250,-302.5)
    rotateX(HALF_PI)
    image(basemap,0,0)

    for i in range(0, len(tweets)):
        strokeWeight(.5)
        stroke(255,255,255)
        line(tweets[i].get('x'), height-tweets[i].get('z'), tweets[i].get('y'), tweets[i].get('x'), height-tweets[i].get('z'), 0)

        strokeWeight(5)
        stroke(255,0,0)
        point(tweets[i].get('x'), height-tweets[i].get('z'), tweets[i].get('y'))

        strokeWeight(2)
        stroke(255,255,255)
        point(tweets[i].get('x'), height-tweets[i].get('z'), 0)
        lrp = map(i, 0, len(tweets), 0, 1)
        frm = color(255,0,0)
        to = color(0,0,255)
        if i &lt; len(tweets)-1:
            strokeWeight(1)
            stroke(lerpColor(frm,to,lrp))
            line(tweets[i].get('x'), height-tweets[i].get('z'), tweets[i].get('y'), tweets[i+1].get('x'), height-tweets[i+1].get('z'), tweets[i+1].get('y'))

    # Uncomment to capture the screens
    """if frameCount &gt; 360:
        noLoop()
    else:
        saveFrame('screens/frame-####.png')"""
</code></pre>

<p>You should be most interested in these lines:</p>

<pre><code>x = map(lon, -19.68624620368202116, 58.92453879754536672, 0, width)
y = map(time, first, last, 0, 500)
z = map(lat, 16.59971950210866964, 63.68835804244784526, 0, height)
</code></pre>

<p><img src="http://www.processing.org/tutorials/p3d/imgs/coordinatesystem.png" title="Processing coordinate system" class="img-rounded pull-left">They define how coordinates inside the cube should be computed. As you see, <code>x</code> is the result of mapping longitudinal extent of our area to the width of cube, the same happens to <code>z</code> and latitude, and to <code>y</code> (but here we map time, not coordinates).</p>

<p>The bounding box used in those computations is the bounding box of the basemap. Interesting thing about Processing and its 3D environment is how it defines the beginning of the coordinate system. As you can see on the left, it might be slighty different from what you could expect. That&#8217;s what you need to be careful about.</p>

<h3>How does it look</h3>

<iframe width="420" height="315" src="//www.youtube.com/embed/4jl6-qOiSAE?rel=0" frameborder="0" allowfullscreen></iframe>
