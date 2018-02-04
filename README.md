# masymos-morre

## Morre – Model Ranked Retrieval Engine
Morre is an Information Retrieval System to search for computational models and related data. It is based on Neo4J and Lucene. Supported model formats are currently SBML and CellML. You can search for models or related persons, publications and annotations.

For this showcase we use cURL, however feel free to use any tool able to handle HTTP Get and POST requests.

## Setup
First of all, please follow the readme at https://github.com/ronhenkel/masymos-core to sertup da Neo4j Server with the MaSyMoS plugin.

## Query
Once this is done, the Plugin offers the following entry points:
- http://yourServer:7474/db/data/ext/Diagnose/graphdb/
  - is_model_manager_alive/ returns a true if the MaSyMoS model manager was loaded or false if not
  - list_index_available/ a list of all available index
  - cellml_model_query_features/ a list of all featrues that can be used with a query for CellML models
  - sbml_model_query_features/ a list of all featrues that can be used with a query for SBML models
  - annotatio_query_features/ a list of all featrues that can be used with a query for extracted annotation
  - person_query_features/ a list of all featrues that can be used with a query for extracted persons
  - publication_query_featrues/ a list of all featrues that can be used with a query for publications
- http://yourServer:7474/morre/query/ a detaild description is provided for each entry point, all following allow 'topn' (limiting the results to the top n retrieved results),
  - simple_cellml_model_query/ allows the features 'keyword' (provided keywords are queried against all features)  retrieves CellML models
  - simple_sbml_model_query/ same as above, retrieves SBML models
  - simple_sedml_query/ same as above, retrieves Sed-ML descriptions
  - cellml_model_query/ takes a list of features `ID, NAME, COMPONENT, VARIABLE, CREATOR, AUTHOR` and a corresponding list of keywords, retrieves CellML models
  - sbml_model_query/ same as above, the featrues are `ID, NAME, COMPARTMENT, SPECIES, REACTION, CREATOR, AUTHOR`, retrieves SBML models
  - model_query/ retrives models, regardless of type, using the feature 'keyword'
  - publication_query/ retrives publications corresponding to a model, features are `ABSTRACT, AFFILIATION, JOURNAL, YEAR`
  - publication_model_query/ same as above, but retrives the models connected to a publication
  - person_query/ retrives persons corresponding to a model, features are `FAMILYNAME, GIVENNAME, EMAIL, ORGANIZATION`
  - person_model_query/ same as above, but retrives the models connected to a person
  - annotation_query/  allows the features 'keyword', retrieves all matching annotations
  - annotation_model_query/ same as above, but retrives models connected to an annotation
- http://yourServer:7474/morre/model_update_service/
  - add_model/ adds a model to the database, takes the parameters `fileId, url, modelType[SBML|CELLML|SEDML]`, fileId a user defined name, url is an accessable location to load the model and modelType defines the encoding, loading OWL is not provided in server mode, this method returns a `uID` as result
  - delete_model/ removes a model from the database, takes the `uID`generated during add_model/ as input, input of `fileID` is optional and used to double-check
  - create_annotation_index/ by default no annotation index is generated in server mode when adding a model, takes the parameter `dropExistingIndex true|false` to decide if the annotation index should be deleted or updated
  
## Examples
For each query type a GET and POST request is possible. The GET request will return a JSON object holding all available features for the particular query. In case of the cellml_model_query the request is:
`curl -X GET http://morre.sems.uni-rostock.de:7474/morre/query/cellml_model_query/ -H "Content-Type: text/plain"`
The returned JSON object is: `["ID","NAME","COMPONENT","VARIABLE","CREATOR","AUTHOR"]`

Now we can setup a query using POST and a JSON object:
`curl -X POST http://yourServer:7474/morre/query/cellml_model_query/ -H "Content-Type: application/json" -d '{"features":["NAME","COMPONENT"], "keywords":["novak","sodium model math channel"]}'`
As a result a JSON object is returned containing the retrieved models and according scores.
`[
{"modelName":"Calzone_Thieffry_Tyson_Novak_2007_version01","score":0.013774771,"modelID":"Calzone_Thieffry_Tyson_Novak_2007_version01","databaseID":3011380124378,"documentURI":"http://models.cellml.org/exposure/1a3f36d015121d5596565fe7d9afb332/calzone_thieffry_tyson_novak_2007.cellml","filename":"calzone_thieffry_tyson_novak_2007.cellml"},
{"modelName":"irvine_model_1999","score":0.011314708,"modelID":"irvine_model_1999","databaseID":2775129546107,"documentURI":"http://models.cellml.org/exposure/5ba0fed413fc9648336caaea382e038a/irvine_jafri_winslow_1999.cellml","filename":"irvine_jafri_winslow_1999.cellml"},
{"modelName":"potter_model_2005","score":0.0059871213,"modelID":"potter_model_2005","databaseID":1175814884599,"documentURI":"http://models.cellml.org/exposure/80982c99e643a576d10e7f3e271a2299/potter_greller_cho_nuttall_stroup_suva_tobin_2005_a.cellml","filename":"potter_greller_cho_nuttall_stroup_suva_tobin_2005_a.cellml"},
...`

### More Examples
```
curl -X GET http://yourServer:7474/morre/query/cellml_model_query/ -H "Content-Type: text/plain"

curl -X POST http://yourServer:7474/morre/query/simple_cellml_model_query/ -H "Content-Type: application/json" -d '{"keyword":"novak"}'

curl -X POST http://yourServer:7474/morre/query/simple_cellml_model_query/ -H "Content-Type: application/json" -d '{"keyword":"novak", "topn":"3"}'

curl -X POST http://yourServer:7474/morre/query/cellml_model_query/ -H "Content-Type: application/json" -d '{"features":["NAME","ID","COMPONENT"], "keywords":["novak","novak","sodium_current_h_gate"]}'

curl -X POST http://yourServer:7474/morre/query/cellml_model_query/ -H "Content-Type: application/json" -d '{"features":["NAME","ID","COMPONENT"], "keywords":["novak","novak","sodium_current_h_gate"],"topn":"3"}'

curl -X POST http://yourServer:7474/morre/query/person_model_query/ -H "Content-Type: application/json" -d '{"features":["FAMILYNAME"], "keywords":["Lloyd"]}'

curl -X POST http://yourServer:7474/morre/query/publication_model_query/ -H "Content-Type: application/json" -d '{"features":["TITLE"], "keywords":["Mathematical modeling of mechanically modulated rhythm disturbances in homogeneous and heterogeneous myocardium with attenuated activity of na+ -k+ pump"]}'

curl -X POST http://yourServer:7474/morre/query/person_query/ -H "Content-Type: application/json" -d '{"features":["GIVENNAME"], "keywords":["Sulman"]'

curl -X POST http://yourServer:7474/morre/query/publication_query/ -H "Content-Type: application/json" -d '{"features":["TITLE"], "keywords":["Mathematical modeling of mechanically modulated rhythm disturbances in homogeneous and heterogeneous myocardium with attenuated activity of na+ -k+ pump"]}'

curl -X POST http://yourServer:7474/morre/query/annotation_model_query/ -H "Content-Type: application/json" -d '{"keyword":"This study investigates the reverse mode of the Na/glucose cotransporter SGLT1. In giant excised inside-out membrane patches from Xenopus laevis oocytes expressing rabbit SGLT1, application of alpha-methyl D"}'

curl -X POST http://yourServer:7474/morre/query/annotation_query/ -H "Content-Type: application/json" -d '{"keyword":"This study investigates the reverse mode of the Na/glucose cotransporter SGLT1. In giant excised inside-out membrane patches from Xenopus laevis oocytes expressing rabbit SGLT1, application of alpha-methyl D"}'

curl -X POST http://yourServer:7474/morre/query/model_query/ -H "Content-Type: application/json" -d '{"keyword":"novak sodium model math channel"}'
```
