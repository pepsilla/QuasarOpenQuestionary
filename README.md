# Quasar Open Questionary
> A project made with quasar-framework to design an simple questionnaire by Alejandro Casasús & José M. Pérez

## Features:
- Basic SCORM 1.2 support
- Data model in statics/open_questionary/data_model.json
- Added sass-loader and vue dependencies 

## ToDo:
- better scorm-api-wrapper
- other eLearning tracking
- other data sources

## Requires:
>Node.js and **quasar-framework v 0.6.5**

## Build setup:
``` bash
# install dependencies
$ npm install
-replace node_modules/scorm-api-wrapper/saw.js from src/statics/todoScormApiWrapper/saw.js
# serve with hot reload at localhost:8080
$ quasar dev

# build for production with minification
$ quasar build

# lint code
$ quasar lint
```
