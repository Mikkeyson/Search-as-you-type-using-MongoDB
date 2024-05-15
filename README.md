"# Search-as-you-type-using-MongoDB" 

This POC was created to demonstrate a working prototype of a Search-as-you-type application that can be implemented using MongoDB. 
Below you'll find a step by step guide on how to connect to MongoDB, and how to use the application. 

To ease reading through the document, here is the list of contents: 
1. [MongoDB setup](doc:linking-to-pages##MongoDB-setup)
2. [Mulesoft setup](doc:linking-to-pages##Mulesoft-setup)
3. [Query formation with 1 key](doc:linking-to-pages#Path-1-Query-formation-with-1-key)
4. [Query formation with 2 keys](doc:linking-to-pages#Path-2-Query-formation-with-2-keys)
5. [POC limitations](doc:linking-to-pages#POC-limitations)

## MongoDB setup

1. Unless you're using a project-provided infrastructure, you'll need to sign up to MongoDB here: https://www.mongodb.com/ .
2. Set up your Cluster by choosing the free option.
3. Set up the Network Access to the IP value of 0.0.0.0/0, which includes your current IP address.
4. Set up the Database Access and configure your own user & password (store the password in a separate document as you'll need it later).
5. You can navigate to Collections to see the list of predefined collections provided by Mongo. The sample_mflix connection has been used for this POC.

## Mulesoft setup

1. Add the MongoDB connector from Exchange.
   
![MongoDBConnector](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/c138aff2-955d-4caf-a3f2-bc11196707bc)

3. Go to the Global Elements, select Create and Search for MongoDB.
4. You'll need to add the Connection String, as seen in the screenshot below, then click OK.

   a) To obtain the Connection String, you need to head back to the MongoDB Cluster you have configured.
   
   b) Click Connect, then select the Compass option. Copy the String provided and paste it into the MongoDB connector in Mulesoft.
   
   c) Replace the <password> value in the String with the password you have set to your user previously.
   
   d) Add "sample_mflix" after the String so the application connects to the specific Database.
   
![MongoDBConfig](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/14b84fbc-a92a-42e5-b8f5-405f661b13ec)


The API is configured to retrieve records based on the number of Keys provided by the end user as seen in the screenshot below: 

 ![implementation](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/2e17b715-a566-4d56-b637-acb913511d1f)

The application determines in the Choice router whether the user has sent 1 or 2 keys by validating the following condition: **payload.Search[0].Key2 == null**
Based on the evaluation, it will follow a certain path, as described below. 

## Path 1 - Query formation with 1 key

**Request body** 

![1key](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/8bb56422-9537-453e-86ee-96a038f70d76)

**The script**

![script1key](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/1d0a583c-9434-4ee0-9d63-92ba8bb6cd36)

    The script evaluates the key and constructs the query for the MongoDB database. 
    A script explanation by ChatGPT below > wink wink <: 
    
                        "\$regex": "^" ++ value:
                                    "\$regex" defines a regular expression for MongoDB queries.
                                    "^" is a regex pattern that matches the beginning of a string.
                                    ++ is the concatenation operator in DataWeave.
                                    value is the variable we declared earlier, so "^" ++ value constructs a regex pattern that matches any string starting with value.
                        "\$options": "i": 
                                    "\$options" specifies options for the regex.
                                    "i" makes the regex case-insensitive.

The **Find Documents** connector is configured with the following values:
        Query: Payload
        Collection name: Movies
        Fields: , (the comma retrieves all the fields available)

The **Transform Message** component works with the data retrieved just for demonstration purposes. 
    That's how the actual data looks in the database:

![ActualData](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/dbbd02a1-a3da-4a75-bf4c-7ed048cfc3d7)

    That's what we'll retrieve: 

![PayloadTransformation1](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/a35a3668-e816-437f-b18c-ab62805839a9)

**Response**

![Response1](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/bd07151a-9885-4cb7-97f2-59ec6a23cc57)

Changing the Value in the Request Body to "Great Ba": 

**Request/Response:**

![Response2](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/d107bf15-ef43-4861-954b-7e1adbc625cc)

It now retrieves the only records that matches the condition.

## Path 2 - Query formation with 2 keys

**The script** is similar, but this time it creates the Regex pattern for 2 keys.

![script2keys](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/99c075cd-232a-4263-babc-2b9c5cf587c2)

**Request body**

![RequestBody2](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/4baf166f-e608-4a6a-8285-c28df7e8363f)

The **Find Documents** connector uses the same configuration as above.

The **Transform Message** uses a Map function to group the records, as shown below:

![payloadTransformation2](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/c4032ee1-9517-4ca1-b62d-b37f4d8f520e)

**Response**

![Response3](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/bf4e1ccd-b7d4-4dc2-a47d-ddd2aa6c641e)


## POC limitations: 

1. The search function works for String values (for "Value" field in the Request Body) within the database. As noticed below, the collection may include Object and Arrays as well.

![limitation1](https://github.com/Mikkeyson/Search-as-you-type-using-MongoDB/assets/169890397/b36da5a9-acd7-4467-ab50-5cba127a8ca1)


