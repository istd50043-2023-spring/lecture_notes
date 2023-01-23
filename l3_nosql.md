# 50.043 - NoSQL

## Learning Outcomes

By the end of this unit, you should be able to

1. articulate the trade-off made between relational model and other data model.
2. contrast the different use-cases for SQL and NoSQL.
3. List the pros and cons for different NoSQL-driven data storage.

## Motivation

NoSQL existed long ago. It recently received more attention due to the shift of infrastruture from on-premise to cloud, from single server to distributed system.

Recall from week 1 lesson, we learnt that database system should be efficient, crash consitent, supporting concurrent access, offering data abstraction. 

As we scale out to multiple machines, we have to give up some of the above. It is technically challenging to have a distributed system that is consistent.  Another less alarming issue is that it is hard to perform efficient data join across machines (though there has been progress made). 

Due to various reasons, developers turn to the older alternative, key-value data structure. 

## Key-value store

Key-value store is simple. Many of us have seen the in-memory version, e.g. dictionary of Python, HashMap of Java, etc. The idea of key-value storage is to put data associated with a key in a look-up table. Keys must be unique, otherwise existing data will be overwritten. 

Storing data in key-value pairs offers some obvious advantages
1. easy to retrieve, like part of your code
2. fit in many applications, mobile app, web app. 
3. fast, since it is just a lookup and/or set.
4. easy to scale, since we could split the data by key-space. 

However key-value pair storage has several limitations.
1. limited API, we can only get or set. There is no range query.
2. Join query is out of the question.
3. it does not differetiate the data type you store by default everything is binary.
4. it does not support inner structure in your data.
5. there is no immediate consistency gauranteed, i.e. sometimes your change takes a while to be reflected on the web app.

Due to the above reasons, it pushes the processing load to the application-tier (which is your source code, in Python, Java or C etc.)

## Document store

### Unstructured Document

In this category, the data store saves data in an unstructured manner. Treating everything as text or sequence of bytes.

The advantage is it saves every thing, like your file system. The disadantage is that it offers no processing API.

### Semi-structured Document

It is a tree-like structure. Data are stored like JSON/XML document. Some implementation offers limited support of indices and range search, e.g. MongoDB.

The advantage is that semi-structured data are close to their in-memory counter-parts living in the software application, (binary tree, graph, JSON). It offers flexibility of going back and fro between database and app without the need of joining and object-relational mapping. 

Most of the semi-structured data are self-explanatory, respecting data types and data structure. 

One obvious disadvantage is that semi-strutured data do not eliminate data redundancy. Immediate consistency is handled loosely. Range queries could be harder to implememented.


