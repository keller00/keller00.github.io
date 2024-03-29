---
title: "Cerberus - The Mighty Watchdog of Hades"
date: 2016-10-09T00:00:00-04:00
subtitle: "Perfect tool to syntax check your json api"
tags:
- syntax checking
- syntax validation
- cerberus
- python
- programming
---
I was given the task of writing an automated test to check the syntax of json objects returned by a product's api calls.
[__Cerberus__][1] is a python library that allows to check different fields of a dictionary based on various things.
At first I thought about writing my own testing library and after planning and designing for an hour I knew exactly what I needed and how it should have been implemented. Before I started actually writing this library I checked on Google and found [__Cerberus__][1]. It was exactly what I needed!

[__Cerberus__][1] has the ```Validator()``` object to actually check the syntax of a dictionary, but it needs a Schema to know what to expect and what is acceptable.

I hope that this example will help you get started with Cerberus. Disclaimer: I have wrote an extensive 600-700 lines long test for this one api and now I see how this could be done in a more elegant way by extending the Validator Object and defining my own object types, but for this one api test splitting up the schemas into smaller dictionaries will do.

For the sake of this example let's assume that the following is returned by our api call:  

```python
{'clip_id': '34sa3-olm34-16re3-u2h4',
'playlist_id': 'as23d-h345b-3545-jfh45',
'timeline_id': None,
'playlist_index': 0,
'clip_time_remaining': '00:00:02:29.0',
'protected_out_timestamp': 765455241,
'protected_in_timestamp': 765455125,
'playlist_time_remaining': '00:00:00.0',
'playmode': 0,
'timecode': '00:00:13:00.1',
'date': '2016-05-06',
'timestamp': '145683421',
'playspeed': 1.0,
'record': True,
'ports': {'0': [0, 0], '1': [0, 1], '2': [1, 0], '3': [1, 1]},
'id': '3524-a456-h312-aky4'}
```
In this example we have every simple and most often used python variable types (strings, integers, float, list and dictionaries).  
As I worked my way through many of these dictionaries, I have recognized the patterns and separated the schemas into smaller dictionaries. Like so:  

```python
id_required = {'type': 'string', 'regex': '^[a-z0-9]+-[a-z0-9-]+$', 'required': True}
nullable_id_required = {'type': 'string', 'nullable'= True, 'regex': '^[a-z0-9]+-[a-z0-9-]+$', 'required': True}

Time_Object_Schema = {'timecode': {'type': 'string', 'regex': '^([0-9]{2}:){2}([0-9]{2}.{0,1}){2}[0-9]{0,1}$', 'required': True},
                      'date': {'type': 'string', 'regex': '^[0-9]{4}(-[0-9]{2}){2}$', 'required': True},
                      'timestamp': {'type': 'integer', 'coerce': int, 'required': True}}

Input_Object_Schema = {'clip_id': id_required,
    'playlist_id': id_required,
    'timeline_id': nullable_id_required,
    'playlist_index': {'type': 'integer', 'required': True, 'min': 0},
    'clip_time_remaining': Time_Object_Schema['timecode'],
    'protected_out_timestamp': Time_Object_Schema['timestamp'],
    'protected_in_timestamp': Time_Object_Schema['timestamp'],
    'playlist_time_remaining': Time_Object_Schema['timecode'],
    'playmode': {'type': 'integer', 'required': True, 'min': 0, 'max': 1},
    'timecode': Time_Object_Schema['timecode'],
    'date': Time_Object_Schema['date'],
    'timestamp': Time_Object_Schema['timestamp'],
    'playspeed': {'type': 'integer', 'required': True},
    'record': {'type': 'boolean', 'required': True},
    'ports': {'type': 'dict', 'required': True, 'valueschema': {'type': 'list', 'required': True, 'valueschema': {'type': 'integer', 'coerce': int}}},
    'id': id_required}
```
Most of the schema is pretty straight forward as the [__Cerberus__][1] documentation is good and easy to understand. I only had a hard time figuring out how to handle the weird port dictionary of lists without allowing unknown fields, as we would want to know if anything changes.  

You can just tell Cerberus to tests the values of the dictionary with the ```valuschema``` key. This way it will call ```.keys()``` on the dictionary and test the keys against the schema given in ```valueschema```. In there I told it to expect a list of integers.  

Anyone using Python and apis (1st or 3rd party ones for that matter) should use Cerberus to keep the api responses in check, so any changes would be apparent.  

[1]: http://docs.python-cerberus.org/
