# Charcoal Production Site Monitoring Service
## Introduction
Firewood and charcoal provide more than 80 percent of energy used in
sub-Saharan Africa, according to the World Agroforestry Centre, with only
a small volume produced sustainably. In Ghana, charcoal is a popular
choice for cooking and heating in urban areas as it is locally available and
by far the cheapest option when compared to electricity, kerosene, and
cooking gas. However, since charcoal is produced from wood, forest
resources are under increasing pressure because charcoal production has
accelerated deforestation throughout the region.

To address these challenges, the SERVIR-West Africa consortium member,
Centre for Remote Sensing and Geographic Information Services
(CERSGIS), is collaborating with local governments and non-governmental
organizations to develop a charcoal production monitoring service.

Using
free and open-source satellite data from NASA’s Landsat suite, current
charcoal production sites are being mapped with pertinent information
shared at the district government level for more informed environmental
planning. This monitoring service will also incorporate a text messaging
service for mobile phones to disseminate the location of approved
production sites (and protected areas to avoid) to stakeholders in rural
communities.

The Charcoal Production Monitoring service, has a monitoring platform with information on all charcoal production sites in project districts in the savanna landscape. This provides the necessary spatial data input to decision-makers and partners interested, who will use the information to target areas for remediation and landscape restoration activities. In addition to the monitoring platform is a mobile application for collecting data on the field.


## Goals and Objectives
* Support the development of a sustainable charcoal value chain through earth observation technology intervention and capacity building.
* Co-design and co-develop a web based charcoal production site monitoring and decision support portal
* Engaing national, district and traditional authorities to secure their buy-in.
* Generate anciliary dataset for the charcoal production districts.
* Generate maps for charcoal districts.
* Train stakeholders on the usage of the mobile application.


  

## Methodology and Workflow
 The Charcoal Production Monitoring service started with the use of Sentinel-2 imagery and the Planet monthly mosaic. While useful, this method was hindered by weather (cloud cover), land cover variations, and limited data availability. The current phase of the project employs high resolution image from Google Earth Pro other open source softwares like QGIS, Open Foris Collect and Collect Earth. The method for the detection of the charcoal production sites involves some key steps:

* Survey Design :The design of the survey for the collection process is created using Open Foris Collect. It involves creating a structured approach to gather information from the charcoal districts. It encompasses various elements such as creating a survey from scratch, importing an already existing survey, and a database to review/preview completed and uncompleted surveys. An effective survey design ensures that the collected data is reliable, valid, and relevant to the project. Overall, a well-designed survey is essential for obtaining accurate and meaningful data to inform decision-making, research, or policy development.

* Creation of Grids : The grids creation was done using QGIS open source software. This was done to provide a clear and organized framework during the collection process. 10km x 10km grids were generated over the identified charcoal districts.  250m x250m grids was later superimposed on the 10km x 10km grids and saved in an ESRI shapefile formart. Uniques codes are assigned to the grids generated since each charcoal district has different codes. The grids were created to provide a structured way to organize and compare data, making it easier to identify patterns, trends, and relationships. 


* Google Earth Pro : The generated grids are imported into Google Earth Pro for the collection process to begin. Placemarks are used to identify and indicate whether the production site is a scar or kiln on the satellite image. A detected production site is described by it's dimension (measured in meters), date of observance and type (wheter a scar or kiln). The scar/kiln detected are then saved in KML format.

* Validation of collected points : The detected charcoal sites and survey are imported using the Open Foris Collect Earth desktop version into Google Earth Pro. The points are seen as a plot. When a plot is clicked, the survey questions to be answered pops up and each question is answered and saved. The detected charcoal scars and kilns are validated upon answering all survey questions, ensuring data accuracy and reliability for subsequent analysis and decision-making processes.


![Charcoal-workflow](<images/Workflow/Charcoal Workflow.jpg>)

## Project Partners


<table style="border: 0;">
  <tr width="100%"> 
    <td><img src="./images/Logos/cersgis-logo.png" width="800" height="50"></td>
    <td><img src="./images/Logos/SERVIR_Logo.png" width="500" height="50"></td>
    <td><img src="./images/Logos/nasa.png" width="500" ></td>
    <td><img src="./images/Logos/USAID.png" width="1000"></td>
    <td><img src="./images/Logos/ICRISAT-1.jpg" width="600"></td>
  </tr>
</table>
</br>
