## Classification 
Google Earth Engine's classification scripts utilize the ESA WorldCover classification, which encompasses the entire globe. However, we have tailored it to focus specifically on our Area of Interest (AOI). The ESA WorldCover classification comprises eleven distinct classes, but for the purpose of our work, we have honed in on three key classes: Built-up areas, and Agricultural land.
## BareArea 
The BareArea scripts are designed to leverage Landsat 8 surface reflectance imagery, which consists of eleven spectral bands, each with a spatial resolution of 30 meters. Landsat 8 is equipped with two sensors: the Operational Land Imager (OLI) and the Thermal Infrared Sensor (TIRS). Within the Google Earth Engine platform, Landsat imagery is carefully curated and filtered to focus specifically on our Area of Interest (AOI). From the Landsat dataset, bands 2, 3, 4, 5, 6, and 7 are selected to capture relevant spectral information. Subsequently, an inbuilt algorithm within Google Earth Engine is employed to process this imagery and delineate the bare areas within our AOI.

## Riparian Zone
Within the Google Earth Engine platform, the riparian zones of the project districts were delineated utilizing the Shuttle Radar Topography Mission (SRTM) Digital Elevation Data version 4. This dataset was specifically designed to furnish consistent and top-tier elevation data. Employing a Google Earth Engine algorithm, the SRTM data underwent processing to isolate the riparian areas within our project districts. This methodological approach ensures precision and reliability in identifying these critical ecological zones.


