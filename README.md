# DS 4320 Project 2: Predicting Air Quality in Washington, D.C.

This project utilizes data sourced from various air quality control sites with the intent of analyzing air quality levels and making predictions as to what 

### Name:
Teagan Britten

### NetID:
uup3cy

### DOI:

### Press Release 
[Access Here](pressrelease.md)

### Pipeline:
[Access Here](pipeline.ipynb)

### License:
[MIT License](LICENSE)

## Problem Definition

### Initial General Problem:
Predicting Air Quality

### Refined Problem:
Predicting air quality for a given day in Washington, DC

### Motivation
Air Quality has a significant act on our planet and society. A low air quality is reflective of harmful pollution and emissions that accelerate the affects of climate change and put the future of the planet at risk. Additionally, air pollution affects the health and living of humans, animals, and plants living around the causes of pollution and can have noticeable negative effects if frequently exposed to bad air quality. 

### Rationale
Washington, DC is at the heart of one of the most populous metropolitan regions in the United States, and itself is home to almost seven hundred thousand people. With that and the significant activity that takes place in the district, there is a risk for high rates of pollution from motor vehicles, industrial buildings, and other sources. Additionally, DC is an area that has struggled with disparate impacts of harmful conditions between races, and air pollution is no different. 

### [(Press Release) New System Allows DC Government to Better Understand and Predict Air Quality](pressrelease.md)

## Domain Exposition

### Terminology

| Term | Definition | 
| --- | --- |
| Ozone | An atmospheric gas, is often called smog when at ground level. It is created when pollutants emitted by cars, power plants, industrial boilers, refineries, and other sources chemically react in the presence of sunlight | 
| Traffic-Related Air Pollution (TRAP) | A mixture of gasses and particles, has most of the elements of human-made air pollution: ground-level ozone, various forms of carbon, nitrogen oxides, sulfur oxides, volatile organic compounds, polycyclic aromatic hydrocarbons, and fine particulate matter |
| Particulate matter (PM) | Composed of chemicals such as sulfates, nitrates, carbon, or mineral dusts. Vehicle and industrial emissions from fossil fuel combustion, cigarette smoke, and burning organic matter, such as wildfires, all contain PM |
Polycyclic aromatic hydrocarbons (PAH) | Organic compounds containing carbon and hydrogen. Of more than 100 PAHs known to be widespread in the environment, 15 are listed in the Report on Carcinogens. In addition to combustion, many industrial processes, such as iron, steel, and rubber product manufacturing, as well as power generation, also produce PAHs as a by-product. PAHs are also found in particulate matter | 
| Noxious gases | Include carbon dioxide, carbon monoxide, nitrogen oxides (NOx), and sulfur oxides (SOx), are components of motor vehicle emissions and byproducts of industrial processes |

The project lives in the overarching domain of environmental science, specifically focusing on environmental health and air pollution. Environmental Scientists study air pollution as it originates from both artificial (human-made) sources and natural sources. The geographic scope of this project is limited to data collected in the District of Columbia and made public by Open Data DC, and looks at levels of various chemicals in the air across the city to understand the effect that air quality has on the people and how this is different from day to day. 

### Background Readings 

[Access The Full Folder Here](https://1drv.ms/f/c/6a550bae65b9bbfe/IgCVajrRajQsSZp8YouXwCRDAQoMPtj_BaMYrsDs93-fYqA?e=guo86E)

| Title | Description | Link |
| --- | --- | --- |
| Air Pollution and Your Health | Looks at causes of air pollution and what health conditions might be affected or caused by it | [Access Here](https://1drv.ms/b/c/6a550bae65b9bbfe/IQDIQdVGsdayS4hHVLFnOUCJAVdLXA3io75y82Hu8ki61k8?e=paNk2x) |
| DC area gets an ‘F’ for air quality, according to new report | Describes challenges in DC's air quality for 2025, including a lower rating than 2024 | [Access Here](https://1drv.ms/b/c/6a550bae65b9bbfe/IQBdce8XTp5VQ6sRG71yTCRYAQgq2kicrcoaklAPyxukAJE?e=03Lujw) |
| Disparities in the Impact of Air Pollution | Discusses the more severe impacts of air pollution on minorities, particularly black neighbourhoods and the Medicaid population | [Access Here](https://1drv.ms/b/c/6a550bae65b9bbfe/IQBNKmA_jNr3TZVZ-xgH_9wQAZaVaEMiPnKOvHPV9nGWiSM?e=arz6vc) |
| 5 ways cities are cleaning the air we breathe | An observation of methods used by cities around the world to reduce city-related pollution | [Access Here](https://1drv.ms/b/c/6a550bae65b9bbfe/IQCxuUp2QIilRIeo2yeCH7FwAR11KMRp66EHJYwvzj-wTys?e=I6a4Na) |
| How Is Air Quality Measured? | An entry-level description of the methods used to communicate air quality | [Access Here](https://1drv.ms/b/c/6a550bae65b9bbfe/IQBGdaIweNx7QbKYmpZ5gN11AeNAwrEG1_TyWH98byjxrPg?e=cnbVPV) | 

## Data Creation 

This data is collected from the DC Open Data Portal 