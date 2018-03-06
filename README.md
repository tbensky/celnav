# Teaching and Learning Celestial Navigation Using Google Maps


Here is documentation for the code presented in the paper "Teaching and
Learning Celestial Navigation Using Google Maps" that was published in
the [Journal of Navigation](https://www.cambridge.org/core/journals/journal-of-navigation) (https://doi.org/10.1017/S0373463317000777), published
online: 07 November 2017.

## Quick start

To get up and running quickly, do this:

1. Download celnav.html and the images folder.

2. Wherever you place celnav.html, maintain the structure, in that the 'images' folder (with all images in it) is directly branched off of the location
where you put celnav.html.

3. Load celnav.html into a text editor.  Put your Google Maps API key into line 5 (the
long `<script async defer src="https://maps.googleapis.....>` line.

4. Load celnav.html into your web browser (using its File->Open...) menu option.

You should see a map of a ship and the Sun, near the Hawaiian islands.

5. If nothing happens in your browser, check your Javascript error console. You might have to hit 'reload' again (and again) in your browser, or
maybe your Google Maps API key is wrong, etc.


## Live figures from the paper

Here are live Javascript pages that render the figures found in the paper.

<ul>

<li> <a href=https://tbensky.github.io/celnav/celnav_fig01.html>Fig. 1</a>
<li> <a href=https://tbensky.github.io/celnav/celnav_fig02a.html>Fig. 2a</a>
<li> <a href=https://tbensky.github.io/celnav/celnav_fig02b.html>Fig. 2b</a>
<li> <a href=https://tbensky.github.io/celnav/celnav_fig04.html>Fig. 4</a>
<li> <a href=https://tbensky.github.io/celnav/celnav_fig05.html>Fig. 5</a>
<li> <a href=https://tbensky.github.io/celnav/celnav_fig06.html>Fig. 6</a>
<li> <a href=https://tbensky.github.io/celnav/celnav_fig07.html>Fig. 7</a>
</ul>


## Start here: Making the figures found in the paper 

Probably the best way to get started with the code is to see how it generates the figures found in the paper.

To do so, scroll down to line 775 (or so) in celnav.html, the one containing the text 'START HERE.' There you'll see a whole slew
of comments on how each figure was generated. 

Try to at least glance at the function ``render_all()``.  It is the main function called
when any of the code is ready to draw something on the map (like the ship, sun, an LOP, etc.),
and looks like this
```javascript
 function render_all() {
                do_calcs();                 // Always leave uncommented
                
                //
                //pick what you want to draw by uncommenting a given draw_... line
                //comment out what you don't want to draw
                //
                //draw_ap_balloon();        // Draw a balloon at the AP
                //draw_co_Hc();               // Draw AP-to-sun connectors 
                //draw_intercept();           // Draw intercept (difference between Hc and Ho (computed vs. observed altitudes) )
                //draw_lop();               // Draw a line of position
                //draw_ap_dot();            // Draw a small dot at the AP
                //draw_cop();               // Draw a circle of position
                draw_ship();                // Draw the ship
                draw_sun();                 // Draw the Sun
                //draw_nav_triangle();      // Draw the navigation triangle (connecting the north pole, AP, and GP)
                
            }
```

To control what is drawn, you basically uncomment what you want to draw and comment what you don't want to draw. Above,
it looks like only the ship and sun will be drawn.



## Setting up your own celestial navigation scenario

Simulating a sight-reduction, requires three items: (lat,lng) of the AP, (declination,GHA) of the Sun's GP, and the altitude of the Sun that would be
measured from the ship.  These are held as global variables at the top of the code as in this block:

```javascript
        var ship_lat, ship_lng;     // position of the ship
        var ap_lat, ap_lng;         // position of the AP
        var gha, decl;              // GHA and declination of the Sun
        var alt;                    // altitude of the Sun
```


So if you want to set up your own scenario, say to do things perhaps in waters more familiar to you, comment out (or remove) all lines between the ```////START HERE////``` and ```///END///``` blocks.  It its place, 
define all of these variables, as you wish. In other words, something like:

```javascript
    ship_lat = 27.33
    ship_lng = -160.82
                 
    ap_lat = 23.63;
    ap_lng = -155;

    decl = 6.23;
    gha = -139.22;
    alt = 60.6266;
```

Naturally, the (decl,GHA) of the Sun would come from the Nautical Almanac, and the altitude, which in this case is the altitude of the Sun one would observe from the
deck of the ship, which would be found an online calculator, like the [NOAA Solar Calculator](http://www.esrl.noaa.gov/gmd/grad/solcalc/). The observation
time is naturally a constant throughout.

With these variables defined, do a call to a function called ```do_calcs()```. This will compute a variety of things from your variables, including:

* The navigation triangle for the AP.  Results are available in variables ```A```, ```B```, ```b```, ```C```, and ```c```.

* The navigation triangle for the ship (supposedly, the observed altitude could come from this--we just always used the online calculator). See Fig 3 in the
paper. Results are available in variables ```Aship```, ```bship```, ```Cship```, and ```cship```.

* The altitude intercept, ```dH```.

* (lat,lng) of the intercept point (towards or away from the GP, relative to the AP). This is available in the object ```intercept_lat_lng``` or as scalars
in ```intercept_lat_lng.lng()``` and ```intercept_lat_lng.lat()```. This is the center point of the LOP.

* End-points of the line of position (LOP) in objects ```lop1``` and ```lop2```, and as scalars as ```lop1.lng()```, ```lop1.lat()```, ```lop2.lng()``` and ```lop2.lat()```.
These points are equidistant from the (lat,lng) in ```intercept_lat_lng```.

* Shortest distance from the ship to the LOP in ```ship_lop_dist```.

All quantities computed by ```do_calc()``` are now available for use as global variables. Here they are, as declared in the code:

```javascript
        var A, B, b, C, c;                  // Navigation triangle for the AP
        var Aship, bship, Cship, cship;     // Navigation triangle for the ship
        var intercept_lat_lng;              // Intercept point, toward or away from the GP along the AP-GP connector
        var lop1, lop2;                     // Endpoints of the LOP
        var coalt;                          // 90 - alt
        var Hc;                             // 90 - C from APs computed navigation triangle (i.e. the altitude computed from the AP)
        var Ho;                             // = alt (i.e. the altitude observed from the ship).
        var dH;                             // altitude intercept
        var ship_lop_dist;                  // closest resulting ship-to-LOP distance.
```

Next, decide what you you want to plot on the map.  The totality of pre-defined drawing capabilities are in the function called ```render_all()``` (above). So
here is a code-set that would set up the key variables, run the calculations, the plot the ship, AP, intercept distance, and line of position.  We also
center the map on the ship, and set the zoom level to 6.  


```javascript
    ship_lat = 27.33
    ship_lng = -160.82
                 
    ap_lat = 23.63;
    ap_lng = -155;

    decl = 6.23;
    gha = -139.22;
    alt = 60.6266;
    
    do_calcs();
    
    draw_ship();
    draw_ap_balloon();
    draw_intercept(); 
    draw_lop();
    
    map.setCenter(new google.maps.LatLng(ship_lat,ship_lng));
    map.setZoom(6);
    
    console.log(ship_lop_dist);
    console.log(intercept_lat_lng.lat(), intercept_lat_lng.lng());
    console.log(dH);
```

The last 3 lines, containing the ```console.log()``` statements will
output text to the Javascript console (they are like ```print``` or
```write``` statements in other languages).  Here we output the eventual
ship-to-lop distance, the intercept coordinates, and the difference between
the observed (from the ship) and computed (from the AP) intercept distances.  

Opening the Javascript console in this case will reveal a 16.93, which is the Nautical Miles
between the ship and LOP. You'll see the (lat,lng) of the intercept is
(28,-159.9) and the difference between ```Hc``` and ```Ho``` (```dH```) is -6.28.

## St. Hilaire's journey (12/17/1837 @ 10:15am)

(Slowly) working to use this framework to re-create St. Hilaire's journey from
South Carolina to Greenock, Scotland. His navigation showed us (for the
first time), the utility of approximating a circle of position with
a line of position.

In the page below, I have Small's Lighthouse plotted, and St. Hilaire's likely
circle of position, with the Sun's GP way down on the tip of Africa. I think
the Sun's GP on 12/17/1837 at 10:15am was (23.33S,29E). The altitude was 10.04 deg.,
althought 8.88 deg puts the circle of position right through the lighthouse,
as per St. Hilaire's log.

<ul>

<li> <a href=https://tbensky.github.io/celnav/celnav_st_hilaire.html>St. Hilaire</a>

</ul>


  



