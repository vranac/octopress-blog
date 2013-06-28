---
layout: post
title: "The curious problem of embedded document duplication"
date: 2013-06-28 09:08
comments: true
categories:
    - mongo
    - doctrine
    - php
    - symfony
tags:
    - mongo
    - doctrine
    - php
    - symfony
keywords: symfony, doctrine, mongodb, embedMany, duplicates
---

We upgraded the current project to Symfony 2.3.1 and sent it in production.
All was working well for a few days and then interesting bug report came yesterday.
<!--more-->

A bit of a background...

We are using mongodb for the project, overall happy with it.
The document that is being edited has embedded documents for photos, files etc using @MongoDB\EmbedMany(...) annotation.
Each embeddable document for the photo has an url, name and description.

In the Symfony form they are represented as embedded collection of forms, with prototype, adding and deletion

``` php
$builder->add('photos', 'collection', array(
    'type' => new Embeddable\Photo(),
    'prototype' => true,
    'allow_add' => true,
    'allow_delete' => true,
    'by_reference' => false,
    'required' => false,
    'attr' => array('form-type' => 'horizontal', 'prototype-type' => 'photo-listing')
));
```
We are using bootstrap and I overloaded the form_div_layout.twig to generate different layouts depending on the prototype-type attribute passed.
The view has a bit of js required for collection, nothing special and/or of consequence.
All standard stuff so far, meat and potatoes you might say.

Getting back to our problem at hand...

The users were uploading photos and they were being uploaded and recorded, but with **duplicates**.

I am looking through the code, there is a check for duplicates in place, it is working, confirmed
by tests and var_dumping (yeah, fugly but hey, it works).

The uploading works, the deletion works, the editing works... And then I try sorting (jQuery Sortable is used), I drag and drop and item, then try to upload and save it...

All hell breaks loose seven ways from sunday, my mind is stuttering like Porky Pig trying to say "WTF??".
The code behaves, the js behaves, all is well until I send it off to Doctrine ODM for persisting.

So I take a look at the at the mongodb queries that doctrine generates (shortened for brevity and dumped the full url :) ) and I find this little gem:

``` javascript
...
db.listings.update({ "_id": ObjectId("51ccb74f428562a30b000003") }, { "$set": { "photos.7.url": "https://s3.amazonaws.com/.../image-640-480-3.jpg" } });
db.listings.update({ "_id": ObjectId("51ccb74f428562a30b000003") }, { "$unset": { "photos.0": true, "photos.1": true, "photos.2": true, "photos.3": true } });
db.listings.update({ "_id": ObjectId("51ccb74f428562a30b000003") }, { "$pull": { "photos": null } });
db.listings.update({ "_id": ObjectId("51ccb74f428562a30b000003") }, { "$pushAll": { "photos": [ { "url": "https://s3.amazonaws.com/.../23a-6.jpg" }, { "url": "https://s3.amazonaws.com/.../image-640-480-1-copy-1.jpg" }, { "url": "https://s3.amazonaws.com/.../image-640-480-2-3.jpg" }, { "url": "https://s3.amazonaws.com/.../image-640-480-3.jpg" }, { "url": "https://s3.amazonaws.com/.../2104878/23-2.jpg" } ] } });
...

```

The thing to note is the line 5, the [$pushAll](http://docs.mongodb.org/manual/reference/operator/pushAll/) which will **append** the items to the current collection.
Awesome, so here is how my freakin' duplicates are created, and my mind is going like Porky Pig again, time to unleash my Google-Fu...

Lo and behold I get to this [major bug report](http://www.doctrine-project.org/jira/browse/MODM-92?page=com.atlassian.jira.plugin.system.issuetabpanels:all-tabpanel) that is supposed to be fixed, but seems that I am somehow hitting it... Oh goodie...

First comment opens my eyes like a 3-year-old-sticking-a-finger-in-your-eye-coz-that-is-how-daddy-is-to-be-woken-up (Luv ya kid) and points me in the right direction, I need to set the strategy for saving to "set" from the default "pushAll".
Well alright now, that is something to work with.

``` php
/**
 * @MongoDB\EmbedMany(targetDocument="MyBundleNameBundle\Embeddable\Photo", strategy="set")
 * @Assert\Valid
 */
protected $photos;
```

That was simple... So time to retest the page, and the document is now being updated as it should, the doctrine generates the query with [$set](http://docs.mongodb.org/manual/reference/operator/set/)

``` javascript
...
db.listings.update({ "_id": ObjectId("51ccb74f428562a30b000003") }, { "$set": "photos.7.url": "https://s3.amazonaws.com/.../image-640-480-3-1.jpg" } });
db.listings.update({ "_id": ObjectId("51ccb74f428562a30b000003") }, { "$unset": { "photos.0": true, "photos.1": true, "photos.2": true, "photos.3": true } });
db.listings.update({ "_id": ObjectId("51ccb74f428562a30b000003") }, { "$pull": { "photos": null } });
db.listings.update({ "_id": ObjectId("51ccb74f428562a30b000003") }, { "$set": { "photos": [ { "url": "https://s3.amazonaws.com/.../23a-6.jpg" }, { "url": "https://s3.amazonaws.com/.../image-640-480-1-copy-1.jpg" }, { "url": "https://s3.amazonaws.com/.../image-640-480-2-3.jpg" }, { "url": "https://s3.amazonaws.com/.../image-640-480-3-1.jpg" }, { "url": "https://s3.amazonaws.com/.../2104878/23-2.jpg" } ] } });
...
```

{% youtube gBzJGckMYO4 %}