---
title: "Moody and Circ"
date: 2018-01-07T16:53:33-05:00
draft: false
---

Although [SpiceyPy](https://github.com/AndrewAnnex/SpiceyPy) is my largest public python package, this blog post will be about
two smaller, but conceptually important projects that I have started that I think
present some interesting concepts for the planetary science community.

Both of these tools are Command Line Interfaces (CLIs) that are built upon the excellent python package ["python-fire"](https://github.com/google/python-fire). Fire is able to produce a rich command line utility from the existing functions in a python file or class, with essentially no additional code other than the import for fire and a single function call. Fire also grants you options like dropping into a REPL and tab completion. This from my view would be sufficient for most use cases I have and have seen. Alternatives like ["argparse"](https://docs.python.org/3/library/argparse.html) which is built-in to python requires a lot more work and another package called ["click"](http://click.pocoo.org/5/) is popular I just do not need access to the additional functionality. In short, I am able to write functions into a class and directly expose them with fire with essentially no additional code, and after uploading to pypi users can get access to this functionality in their terminals or use the same code as a python package. 

### Moody

The first tool, ["moody"](https://github.com/AndrewAnnex/moody), provides a minimal interface to the Mars ODE REST api through the command line so that simple meta-data queries and provides commands to download HiRISE and CTX EDR images by partial product id. While HiRISE EDR products consist of several .IMG files, they are relatively easy to download using wget. In moody the command would look like:

``` bash
moody hirise-edr PSP_001810_1825*
```

However CTX images are not able to be downloaded using the same trick as they are separated by data volumes, but by using the ODE I am able to query for the correct url without much work. Meta-data can be easily retrieved by running:

``` bash
moody get-ctx-meta P19_008442_1899*
```

And if I am only interested in a single field the emission angle can be retrieved by:

``` bash
moody get-ctx-meta-by-key P19_008442_1899*  Emission_angle
```

At the moment, the functionality for moody is fairly use-case focused, but my hope is that it will grow as more meta-data query use-cases are contributed. In any case, for users that want to perform programatic queries to the ODE, moody provides a template to build upon. My hope is that this tool will be useful for people who know a hirise or ctx product id, but just need to do quick lookups of meta-data without opening a web browser and having to click and scroll some page for the info.

### Circ

My newest tool, ["Circ"](https://github.com/AndrewAnnex/circ), attempts to construct reasonable mosaics using CTX images over a provided bounding box. This came out of a need to look at large areas of Mars where the existing CTX mosaic in Google Earth does not cover. It currently does this by first selecting the CTX images that intersect the user provided bounding box, and then selects images with low emission angles (which is configurable by the user). This is what the command looks like:

``` bash
circ make-vrt 136.0 -7.0 139.5 -3.5 --name gale --em_tol 1.0
```
Here I am making a mosaic over Gale crater and I am limiting the emission angles to be no greater than 1.0 degrees to keep things consistently nadir.

Then the collection is sorted by emission angle and area, and the foot prints are then intersected in a cascade operation so that footprints that almost completely overlap previous footprints are excluded. This reduces the total number of images needed to cover the area to a high degree, and then a similar reduction is then performed by stochastically sampling the collection to further reduce the number of images needed. This is a cheap way to get to good-enough results to what I think is something similar to a packing problem that I am sure is more elegantly solved elsewhere, but from what I have tested so far this works pretty well.

Once the minimal set of images have been selected, the pre-computed pyramided geotiffs provided by the ["ASU Mars Space Flight Facility"](https://viewer.mars.asu.edu/viewer/ctx) are then downloaded in parallel to a folder, and a GDAL VRT is built from the tiffs in that folder. In brief, VRTs are xml files that can point to multiple data sources readable by GDAL that can be local or remote, and then gdal treats the resulting vrt file as a single data source that can then be read directly by other GDAL powered applications like QGIS. For more on VRTs [this blog post](http://www.paolocorti.net/2012/03/08/gdal_virtual_formats/) explains it with more detail. This results in an instant mosaic that can be viewed in QGIS, and then further gdal operations can create a single flat geotiff mosaic to save disk space. Here is a zoom in to the gale mosaic I made using the circ command above displayed in QGIS overlaying the MOLA shaded-colorized topography map:

<img class="pure-img" src=/img/gale_circ_vrt.jpg alt="Gale Mosaic from VRT created by circ"></img>

As you can see there is a pretty large gap by the Dulce Vallis and Farah Vallis markers, looking at the available images it looks like an image with an emission angle of around 2 degrees was excluded, but this is because I asked for images with equal to or less than 1 degree. Otherwise the results are usable. And of course if you needed to add an image to a VRT or remove one, you would simply update the folder containing all of the downloaded geotiffs and re-run gdalbuildvrt. A thing that is missing from Circ at the moment is the ability to histogram match the images to make the overlaps less obvious, but that is something I hope can either be contributed or eventually I will get to it if there is not some other existing utility or function in QGIS or gdal that can do it.

## Conclusion

In my first long blog post, I have discussed two CLI tools I have built for the martians of the planetary science community. I hope they prove to be useful! At the top I alluded to there being interesting concepts for the planetary community, and if I had not made it clear then I should say that it would be a great thing if more members and made their work available in command line tools. Spending a little time to polish them up makes these utilities better for your own use and will benefit the community as a whole if they are made available.
