---
layout: post
title: Extracting and Storing Report Friendly Data from Dynamic Forms in RavenDB
---

We have recently been tasked with creating a form builder that our internal staff could use to - you guessed it - create forms for potential customers. These forms are truly dynamic as our internal staff can create any number of fields of any type, containing any data. These forms are subsequently linked to certain screens of the system in order to allow different tenants to capture different data from the same process. For example, when creating an incident, some tenants just want to capture basic information while others want to capture social insurance number and a unique ID given to the customer when the incident was reported. This form builder gives us the flexibility to model exactly what each tenant, in whatever industry, wants to record.

The data captured by the dynamic form is stored with the form schema. This way we don't have to worry about  form versioning. This is sent to the server in a JSON format:


    {  
       "name":"Case",
       "guid":"077ebaca-d397-4805-80c3-4294d8bcce4f",
       "details":{  
       		"submit":"Save",
			"cancel":"Cancel",
       		"success":"Thank you. Your form has been submitted.",
       		"error":"Sorry, we could not submit your form."
		},
       "fields":
		[  
	      {  "guid":"eaa46736-ed26-4cb9-b7ab-253f1609743a", "label":"Name", "type":"text", "value":"Name of incident", "required":false, "disabled":false,
	     	 "options":[], "sizeX":2, "sizeY":1, "col":0, "row":0
	      },
		  {  "guid":"4d4fdbac-9f0c-456d-9865-a248154889d4", "label":"Priority", "type":"select", "value":"1", "required":false, "disabled":false,
	     	 "options":[{  "label":"High", "value":1}, {"label":"Low", "value":2}], "sizeX":2, "sizeY":1, "col":1, "row":2
	      },
       ]
    }

There are numerous ways to store this data, such as: 

- A list of key/value pairs
- Reflection to create an object to store
- Dynamic object

However, the data needs to be stored in a reporting friendly format. With report friendly format I mean in columns, with column name and value; hence, this rules out the key/value pair option. I try to always avoid reflection if I can as it is cumbersome. This leaves me with dynamic objects but first we need to extract the data from the above JSON object.

### Extracting the Data ###
To extract the data, we simply need to parse the JSON data and retrieve the label and value for each field. I decided to use [SimpleJson](https://www.nuget.org/packages/SimpleJson/) as it has good dynamic support and is very simple to use:

    dynamic form = SimpleJson.DeserializeObject(formData);

There are different kinds of fields and value are stored differently for some of the field types. For example, a select field will store the integer value not the text, which needs to be retrieved form the list of options specified in the JSON data. In order to support multiple field types that might be storing values differently, and to conform to the open closed principle, I decided to use the Chain of Responsibility pattern (more on this pattern in a future post).

### Creating and Storing the Dynamic Object ###
A simple way to create a dynamic object is by using ExpandoObject. ExpandoObject implements IDictionary, hence, it allows us to create and set properties using the indexer, which is perfect in our scenario as the properties to be created are specified as strings in the JSON object representing the form's data. However, in order to use indexers, you need to cast the ExpandoObject to a dictionary (IDictionary<string, object> to be precise). Therefore, to create our dynamic object, we just need to loop through all the fields and create a property per field:

    dynamic form = new ExpandoObject();
    foreach (var field in fields)
    {
	    var propertyName = _getValidPropertyName(field.label.ToString());
	    ((IDictionary<string, object>)form)[propertyName] = field.value;
    }

We can now store the form object, this will create a new collection called ExpandoObject that contains properties representing all the fields from the JSON object with their correct values as depicted below:

![RavenDB](/images/Dynamic-objects-in-RavenDB.png "RavenDB")
