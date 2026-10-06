# From Zero: How Satellite Mapping of Invasive Plants Actually Works
**A plain-language primer, written for a programmer with no remote-sensing background**
For: Vishnuvarthan — 13 August 2026

---

## 0. Read this part first

You are not behind. Remote sensing looks like a science because it's taught by scientists, but the daily work of it is data engineering: fetch arrays, filter them, do arithmetic on them, threshold the result, draw a map. You have done harder things than this.

There are exactly four ideas you need. Everything else is detail you can look up when you hit it.

1. A satellite image is a stack of numbers, not a photograph.
2. Different plants reflect light differently, so those numbers can identify a species.
3. Subtract this year's numbers from last year's and you can see what changed.
4. Google Earth Engine does all of this on Google's computers, for free, so your laptop doesn't matter.

That's it. That's the field. The rest of this document unpacks those four.

---

## 1. The problem you're solving, in ordinary words

There is a tree called **Senna spectabilis**. It isn't from India — it was brought in from South America as a fast-growing shade and firewood tree. In the forests of the Nilgiris it turned out to grow far too well. It shades out native undergrowth, so the grasses and shrubs that elephants and deer actually eat stop growing beneath it. A forest full of Senna looks green and healthy from a road, and is close to useless as habitat.

**Lantana camara** is a shrub with the same story and a wider spread.

The Tamil Nadu Forest Department is under a court order to remove Senna. Crews go into the forest, cut it, and haul the biomass to paper mills. They then report a number: *this many hectares cleared*.

Here is the gap, and it's the whole reason your project exists:

**Senna coppices.** Cut it and it regrows from the stump. So "1,000 hectares cleared" in March does not mean 1,000 hectares are clear in December. Nobody is systematically checking. The reporting measures *effort* — hectares worked on — when what matters is *outcome* — hectares that stayed clear.

Checking on foot is impossible: the terrain is enormous, roadless, and full of elephants. Checking from space is cheap, repeatable, and nobody's doing it.

That's your project. Not "AI for conservation" in the abstract. One specific unanswered question: **did it grow back?**

---

## 2. A satellite image is not a photograph

This is the idea that unlocks everything else, so take it slowly.

When your phone takes a picture, it records three numbers for every pixel: how much **red**, **green** and **blue** light hit that spot. Three numbers per pixel, because human eyes have three kinds of colour receptor. The photo is built to look right to you.

A satellite is not built to please your eye. It records **more colours than you can see**, including ones outside human vision entirely. Each of those recorded colour-ranges is called a **band**.

So where your phone photo is an array of shape `(height, width, 3)`, a Sentinel-2 satellite image is an array of shape `(height, width, 13)`. Same data structure. More depth.

The bands that matter to you:

| Band | What it measures | Why you care |
|---|---|---|
| Blue, Green, Red | Visible light | Lets you make a normal-looking image to check your work |
| **Red Edge** (3 bands) | The sharp transition at the edge of red | Extremely sensitive to plant health and plant type |
| **Near-Infrared (NIR)** | Light just past red, invisible to humans | **The single most important band in vegetation science** |
| Shortwave Infrared (2 bands) | Further out still | Moisture content, burnt areas, bare soil |

### Why near-infrared is the star

Living plant leaves do something strange: they absorb red light hard (chlorophyll eats it for photosynthesis) and reflect near-infrared light hard (the internal leaf structure bounces it back).

So a healthy plant, in satellite numbers, is **low red, very high NIR**. Bare soil is roughly equal in both. Water is low in both.

This is why one of the oldest and most useful calculations in the field is just:

```
NDVI = (NIR - Red) / (NIR + Red)
```

That's it. That's NDVI — the Normalised Difference Vegetation Index — and it is one line of arithmetic on two arrays. Result runs from -1 to +1. Water is negative. Bare rock is near zero. Dense healthy forest is 0.8+.

You now understand the most-cited tool in remote sensing. It took a paragraph.

---

## 3. Why you can identify a specific plant from 786 km up

Every surface reflects a different amount of light at each wavelength. Plot reflectance against wavelength and you get a curve — this is called a **spectral signature**. Different materials have different curves. Pine forest, tea plantation, granite, water, and a Senna thicket all have distinguishable curves.

The trouble is that green things look fairly similar to each other. Telling Senna from surrounding native forest on spectral signature alone is hard.

**Senna gives you a gift, though.** It flowers in a mass of bright yellow, and the whole canopy turns yellow at once. That does two things:

1. **It shifts the spectral signature dramatically.** A yellow canopy reflects far more green and red light than a green one. In the numbers, a flowering Senna stand jumps right out.
2. **It's seasonal.** It happens at a particular time of year and not at others.

That second point is the powerful one, and it has a name: **phenology** — the timing of biological events through the year.

Native evergreen forest looks roughly the same in March and in August. A Senna stand does not. So instead of asking "does this pixel look like Senna?", you ask a much easier question: **"does this pixel turn yellow every year at the same time?"**

That's a pattern-over-time question. It's a time-series problem. It is much more your kind of problem than a botany problem.

---

## 4. Change detection: the actual technique

Take the same patch of ground at two dates. Compute your index (NDVI, or a yellowness index, or several). Subtract.

```
change = index_2026 - index_2024
```

Where the number moved a lot, something happened. Where it didn't, nothing did.

For your project it runs like this:

- **Before clearing:** a compartment shows the strong seasonal yellow-flowering signal. Senna is present.
- **After clearing:** the signal vanishes. Confirms the clearing actually happened, and shows its true boundary — which may not match the reported boundary.
- **One or two years later:** does the signal come back? If yes, that compartment regrew and the clearing didn't hold.

Output: a map with three categories — cleared and holding, cleared and regrowing, never cleared. Nobody has that map. That map is worth real money and real access to people.

The complication you'll actually hit: distinguishing "Senna regrew" from "native vegetation recovered" from "the crew came back and cut it again." That's a genuine research question, not a solved one — which is exactly what makes it publishable and grant-worthy rather than a homework exercise.

---

## 5. What Google Earth Engine is, and why it changes the game

The naive version of this project is impossible on your hardware. One Sentinel-2 scene is over a gigabyte. You want several years of them across multiple reserves. That's terabytes to download, store, and process. Not happening on a laptop in Erode.

**Google Earth Engine (GEE) solves this by inverting the model: you send your code to the data, instead of downloading the data to your code.**

Google already stores the entire Sentinel-2, Landsat and MODIS archives — petabytes, decades deep — on their infrastructure. You write a short script saying "for this polygon, over these dates, filter out cloudy scenes, compute this index, and give me the result." Google runs it across their cluster and hands back only the answer. Often a few kilobytes.

It's free for research and non-commercial use. You sign up with a Google account.

There are two ways in:
- **The Code Editor** — a browser IDE, JavaScript, with a map pane. Best for learning: you see results instantly.
- **The Python API** (`earthengine-api`) — same engine, called from Python or a Jupyter notebook. This is where you'll end up, because it plugs into pandas, GeoPandas and the rest of your existing toolkit.

Start in the Code Editor even though JavaScript isn't your target language. Seeing a map light up on your third line of code is worth a lot when you're learning a new domain.

### Sentinel-2 specifically

Your data source. Two European Space Agency satellites, free and open.

- **Resolution:** 10 metres per pixel in the key bands. One pixel is roughly a badminton court. Big enough to miss a single tree, fine for mapping stands and thickets.
- **Revisit:** every ~5 days. So you get a fresh look at Sathyamangalam most weeks.
- **Archive:** 2015 to today. Ten years of history, free.
- **The catch:** clouds. In monsoon Tamil Nadu, many images are useless. Standard practice is **cloud masking** — throwing out cloudy pixels — and **compositing** — combining many images over a month or season into one clean picture by taking, say, the median value of each pixel. GEE has this built in and it's a few lines.

---

## 6. The vocabulary, in one table

You'll meet these constantly. Now they're demystified.

| Term | Plain meaning |
|---|---|
| **Raster** | An image. A grid of pixels with numbers. Satellite imagery is raster. |
| **Vector** | Shapes — points, lines, polygons. A reserve boundary is vector. |
| **Band** | One colour-channel of a satellite image. |
| **Spectral signature** | The reflectance curve that identifies a material. |
| **NDVI** | `(NIR-Red)/(NIR+Red)`. How much living plant is there. |
| **Phenology** | Seasonal timing of biological events — flowering, leaf drop. |
| **Composite** | Many images merged into one clean one, usually to beat clouds. |
| **Cloud mask** | Flagging and discarding cloud-contaminated pixels. |
| **Classification** | Labelling each pixel as a category (Senna / native / bare). |
| **Ground truth** | Real observations from the actual site, used to train and check your model. |
| **Compartment** | The forest department's own administrative land unit. Report in these — it's the language they manage in. |
| **Change detection** | Comparing dates to find what changed. |
| **QGIS** | Free desktop GIS software. Like Photoshop for maps. |
| **GeoPandas / Rasterio** | Python libraries. GeoPandas = pandas for vector data. Rasterio = reading raster files. |
| **Shapefile / GeoTIFF / GeoJSON** | Common file formats. Vector, raster, vector respectively. |

---

## 7. What you do NOT need to know

This matters as much as the list above, because the fear comes from imagining the whole field is prerequisite. It isn't.

- **You don't need a biology or forestry degree.** You need to know one plant's flowering behaviour, and you now do.
- **You don't need to understand satellite orbits, sensor physics, or atmospheric correction.** Use the Level-2A Sentinel-2 product; the correction is already applied.
- **You don't need to buy imagery or hardware.** Sentinel-2 is free, GEE is free, QGIS is free.
- **You don't need a GPU or a powerful machine.** The compute happens at Google.
- **You don't need deep learning to start.** Simple index thresholds and random forests are standard, respected, and often outperform neural networks on this kind of task. Bring the CV skills later if they help.
- **You don't need to be right the first time.** Publishing "here's my method, here's where it fails" is genuinely more valuable to this community than a polished claim. That's not a consolation prize — an honest failure analysis is what gets technical people to engage with you.

---

## 8. The one real gap you do have

**Ground truth.** You can build the whole pipeline from your desk, but at some point you need to know what's actually on the ground in some of those pixels — to check whether your classifier is right.

Two routes, both fine:
- **Go look.** Sathyamangalam is on your doorstep. GPS points of confirmed Senna stands, taken legally from public roads and permitted areas, are real scientific data. This is also why living in Erode is an asset rather than a limitation.
- **Borrow it.** Keystone Foundation and ATREE have already done field survey work on invasives in this landscape. This is a concrete, non-awkward reason to contact them: not "please give me a job" but "I have a method, you have ground data, can we validate this together."

That second route is why the host-organisation step and the notebook step reinforce each other. The notebook gives you something to offer. Their field data gives you something you can't get alone.

---

## 9. What comes next

You don't have to absorb this in one go. The sequence from here:

1. Sit with this document once. You don't need to memorise the table.
2. Create a Google Earth Engine account.
3. Open the Code Editor and run one script — a Sentinel-2 image of Sathyamangalam on screen. Nothing more.
4. Compute NDVI over it. One line.
5. Then we start the actual project.

Step 3 is about twenty minutes and it's the moment this stops being abstract.
